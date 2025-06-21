# Stable Fluids - Animation Theory 정리 노트

## 1. 컴퓨터 그래픽스에서의 유체 시뮬레이션

### 핵심 요구사항

- **Fast** (빠른 계산 속도)
- **Looks good** (시각적으로 좋은 품질)
- **Easy to code** (구현의 용이성)

## 2. 유체 역학 (Fluid Mechanics)

### 장점

- 유체 모델링을 위한 자연스러운 프레임워크
- 완전한 Navier-Stokes 방정식 사용
- 오랜 역사를 가진 분야
- 기존 코드/알고리즘 재사용 가능

### 단점

- 방정식을 풀기 어려움
- **비선형(non-linear)** 특성

## 3. 기존 연구 (Previous Work)

### 2차원 연구

- **Yaeger & Upson 86 + Gamito et al. 95**: vortex blobs 방법
- **Chen et al. 97**: explicit in time, finite differences

### 3차원 연구

- **Foster & Metaxas 97**: explicit in time, finite differences
- **문제점**:
  - **불안정함(Unstable)!!**
  - 부정확한 스킴도 유용할 수 있음

## 4. 주요 기여사항 (Main Contribution)

### Stable Navier-Stokes Solver

- **어떤 시간 간격(time step)도 사용 가능**
- 더 큰 시간 간격 = 더 빠른 시뮬레이션
- 정확하지는 않음(NOT accurate) - 하지만 안정적

## 5. 응용 (Application)

### 기본 원리

- 속도(velocity)를 사용하여 밀도(density)를 이동시킴
- 2D와 3D 모두에서 작동

### 시뮬레이션 루프

```
While (simulating)
    - UI에서 힘(force) 받기
    - UI에서 밀도 소스(density source) 받기
    - 속도 업데이트
    - 밀도 업데이트
    - 밀도 표시
```

## 6. 방정식 (Equations)

### 기본 방정식

속도는 질량을 보존해야 함(mass conservation)

#### 밀도 방정식

$$\frac{\partial \rho}{\partial t} = -(\mathbf{u} \cdot \nabla)\rho + \kappa\nabla^2\rho + S$$

#### 속도 방정식

$$\frac{\partial \mathbf{u}}{\partial t} = -(\mathbf{u} \cdot \nabla)\mathbf{u} + \nu\nabla^2\mathbf{u} + \mathbf{f}$$

방정식들이 매우 유사한 구조를 가짐!

### 방정식 구성 요소 분석

#### 밀도 변화율 (Evolution of density)

1. **이류항(Advection)**: $-(\mathbf{u} \cdot \nabla)\rho$

   - 밀도가 흐름 방향으로 변화

2. **확산항(Diffusion)**: $\kappa\nabla^2\rho$

   - 밀도가 시간에 따라 확산

3. **소스항(Source)**: $S$
   - UI로부터의 밀도 증가

## 7. 알고리즘 (Algorithm)

### 공간 분할

- 공간을 복셀(voxel)로 세분화
- 각 복셀의 중심에 속도와 밀도 정의

### 처리 단계

```
[초기 상태] → [소스 추가] → [확산] → [이동] → [최종 상태]
         Δt          Δt        Δt
```

## 8. 밀도 확산 (Diffusing Densities)

### 확산 과정

- 이웃 셀들 간의 밀도 교환
- 변화량 = (밀도 유입) - (밀도 유출)
- = $k \Delta t (\text{neighbor} - \text{center}) / h^2$

### 명시적 방법의 문제점

$$D_{i,j}^{n+1} = D_{i,j}^n + k\Delta t(D_{i+1,j}^n + D_{i-1,j}^n + D_{i,j+1}^n + D_{i,j-1}^n - 4D_{i,j}^n)/h^2$$

**불안정 조건**: $\frac{k\Delta t}{h^2} > \frac{1}{2}$일 때 불안정

### 안정적인 암시적 방법

- 시간을 거꾸로 확산시켰을 때 원래 밀도를 얻을 수 있는 밀도를 찾음
- 선형 시스템으로 변환: $Ax = b$
  - $A$는 거대하지만 희소(sparse) 행렬
  - 빠른 선형 솔버 필요

### 선형 솔버 비교

| 이름                   | 비용       | 설명                                            |
| ---------------------- | ---------- | ----------------------------------------------- |
| Gaussian elimination   | $N^3$      | 매우 작은 N에만 사용 (테스트용)                 |
| Jacobi/SOR             | $N^2$      | 구현은 쉽지만 느림                              |
| FFT/cyclical reduction | $N \log N$ | 내부 경계가 없을 때 사용                        |
| Conjugate gradient     | $N^{1.5}$  | 내부 경계가 있을 때 사용                        |
| Multi-grid             | $N$        | 실제로는 FFT보다 느림, 내부 경계 시 구현 어려움 |

## 9. 밀도 이동 (Moving Densities)

### 유한 차분법의 문제점

- 이웃 간에만 전달 가능
- 불안정 조건: $\frac{\Delta t ||\mathbf{u}||}{h} > 1$

### 해결책: 입자와 격자의 결합

1. **입자를 시간상 역추적**
2. **4개의 이웃 찾기**
3. **새 위치에서 밀도 보간**
4. **격자 위치에 보간된 밀도 설정**

### 안정성 증명

$$\rho_{int} = (1-s)\rho_0 + s\rho_1$$
$$\rho_0, \rho_1 \leq \rho_{max}$$
$$\rho_{int} \leq (1-s+s)\rho_{max} \leq \rho_{max}$$

**밀도는 항상 경계 내에 있음** → 무조건적으로 안정!

## 10. 속도 계산 (Computing Velocities)

### 속도 이동

- 속도는 자기 자신에 의해 이동됨
- 밀도와 동일한 방법 사용:
  1. 입자를 역추적
  2. 새 위치에서 속도 보간
  3. 격자 위치에 설정

## 11. 질량 보존 (Conservation of Mass)

### 문제점

셀로 들어오는 유량 ≠ 셀에서 나가는 유량

$$u_{i+1,j} - u_{i-1,j} + v_{i,j+1} - v_{i,j-1} = 0$$

### Hodge 분해 (Hodge Decomposition)

$$\text{우리의 필드} = \text{질량 보존 필드} + \text{그래디언트}$$

질량 보존 필드를 얻으려면:
$$\text{질량 보존} = \text{우리의 필드} - \text{그래디언트}$$

## 12. 발산 없는 속도장 계산 (Computing Divergence Free Velocity Field)

### Helmholtz-Hodge 분해 정리

속도장 $\mathbf{v}$는 다음과 같이 유일하게 분해 가능:
$$\mathbf{v} = \mathbf{u} + \nabla p$$

여기서:

- $\mathbf{u}$: 발산 없는 속도장
- $p$: 압력
- $\mathbf{u}$는 경계에 평행 ($\mathbf{u} \cdot \mathbf{n} = 0$)

### 압력 계산

1. 양변에 발산 연산자 적용:
   $$\nabla \cdot \mathbf{w} = \nabla \cdot \mathbf{u} + \nabla \cdot \nabla p$$

2. $\mathbf{u}$는 발산 없으므로:
   $$\nabla \cdot \mathbf{w} = \nabla^2 p$$

3. 이산화 후 선형 시스템:
   $$\Delta x^2 \nabla \cdot \mathbf{w} = p_{x-1,y,z} + p_{x,y-1,z} + p_{x,y,z-1} - 6p_{x,y,z} + p_{x,y,z+1} + p_{x,y+1,z} + p_{x+1,y,z}$$

### 최종 속도 계산

압력을 구한 후:
$$\mathbf{u} = \mathbf{w} - \nabla p$$

### 2D 시스템 (간단한 형태)

$$P_{i+1,j} + P_{i-1,j} + P_{i,j-1} + P_{i,j+1} - 4P_{i,j} = (u_{i+1,j} - u_{i-1,j} + v_{i,j+1} - v_{i,j-1})h$$

## 13. 구현 요약 (Summary)

### 주요 함수들

```
UpdateVelocity(U1, U0, F, visc, dt)
    AddForce(U1, U0, F, dt)      // 힘 추가
    Diffuse(U0, U1, visc, dt)     // 확산
    Move(U1, U0, U0, dt)          // 이동
    ConserveMass(U1, dt)          // 질량 보존
```

### 구현에 필요한 것들

- **입자 추적기(Particle tracer)**
- **격자 보간기(Grid interpolator)**
- **선형 솔버(Linear solver)**: FISHPAK 또는 Conjugate Gradient

### 핵심 장점

- 매우 쉬운 코딩
- 무조건적으로 안정적
- 큰 시간 간격 사용 가능
- 시각적으로 그럴듯한 결과
