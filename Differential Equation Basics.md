# Differential Equation Basics 강의 노트 (SIGGRAPH 2001)

## 1. 정준 미분방정식 (Canonical Differential Equation)

### 1.1 기본 정의

```
ẋ = f(x, t)
```

여기서:

- **x(t)**: 시간에 따라 움직이는 점 (moving point)
- **f(x,t)**: x의 속도 (velocity of x)
- **ẋ**: x의 시간에 대한 도함수 (dx/dt)

### 1.2 벡터 형태

n차원 시스템의 경우:

```
ẋᵢ = f(x, t)ᵢ    (i = 1, 2, ..., n)
```

이는 다음과 같이 벡터로 표현:

```
[x₁]     [f₁(x,t)]
[x₂]  =  [f₂(x,t)]
[⋮ ]     [  ⋮   ]
[xₙ]     [fₙ(x,t)]
```

## 2. 벡터장 (Vector Field)

### 2.1 벡터장의 개념

미분방정식 `ẋ = f(x,t)`는 **x 공간에서 벡터장을 정의**

### 2.2 기하학적 해석

- 각 점 x에서 벡터 f(x,t)가 정의됨
- 이 벡터는 그 점에서의 **순간 이동 방향과 속도**를 나타냄
- 전체 공간에 걸쳐 벡터들이 분포하여 **흐름(flow)**을 형성

### 2.3 시각화

```
x₂ ↑
   |  →  →  ↗
   |  →  →  ↗
   |  ↘  ↘  →
   └────────→ x₁
```

각 화살표는 해당 위치에서의 f(x,t) 벡터

## 3. 적분 곡선 (Integral Curves)

### 3.1 적분 곡선의 정의

- **임의의 시작점을 선택**하고 **벡터를 따라 이동**
- 벡터장을 따라 그려지는 곡선
- 미분방정식의 해(solution)를 나타냄

### 3.2 기하학적 구성

1. 시작점 x₀ 선택
2. 그 점에서의 벡터 f(x₀,t) 방향으로 이동
3. 새로운 점에서 다시 벡터 방향으로 이동
4. 이 과정을 연속적으로 반복

### 3.3 수학적 표현

적분 곡선은 다음을 만족:

```
dx/dt = f(x,t)
```

## 4. 초기값 문제 (Initial Value Problems)

### 4.1 문제 정의

**주어진 조건:**

- 시작점: x(t₀) = x₀
- 미분방정식: ẋ = f(x,t)

**목표:**

- 적분 곡선을 따라가며 해 x(t) 구하기

### 4.2 수학적 표현

```
ẋ = f(x,t)
x(t₀) = x₀
```

이를 만족하는 x(t)를 찾는 것이 목표

## 5. 오일러 방법 (Euler's Method)

### 5.1 기본 아이디어

- **가장 간단한 수치해법**
- **이산 시간 단계** 사용
- **더 큰 단계 → 더 큰 오차**

### 5.2 수학적 공식

```
x(t + Δt) = x(t) + Δt · f(x(t), t)
```

### 5.3 알고리즘 단계

1. 현재 상태 x(t)에서 시작
2. 현재 도함수 f(x(t), t) 계산
3. 선형 근사를 사용하여 다음 상태 예측
4. 시간을 Δt만큼 진행

### 5.4 구현 예시

```cpp
void eulerStep(System& sys, float h) {
    float t = sys.getTime();
    vector<float> x0 = sys.getState();
    vector<float> deltaX = sys.derivEval(x0, t);

    sys.setState(x0 + h * deltaX, t + h);
}
```

## 6. 오일러 방법의 문제점

### 6.1 문제 I: 부정확성 (Inaccuracy)

- **오차가 누적**되어 실제 해에서 벗어남
- 예시: 원형 궤도가 나선형으로 변형
- **에너지 보존이 안됨**

#### 원형 운동 예시

실제 해는 원이지만 오일러 방법으로는:

```
실제: x² + y² = constant (원)
오일러: 반지름이 계속 증가 (나선)
```

### 6.2 문제 II: 불안정성 (Instability)

- **시간 단계가 너무 클 때** 발생
- 해가 발산하거나 진동
- **수치적 폭발** 가능성

#### 안정성 조건

```
Δt < 2/λ_max
```

여기서 λ_max는 시스템의 최대 고유값

## 7. 중점 방법 (Midpoint Method)

### 7.1 기본 아이디어

오일러 방법의 정확도를 개선하기 위해 **중점에서의 도함수 사용**

### 7.2 알고리즘 단계

```
a. 오일러 단계를 중점까지 계산
   x_mid = x(t) + (Δt/2) · f(x(t), t)

b. 중점에서 f 평가
   f_mid = f(x_mid, t + Δt/2)

c. 중점 값을 사용하여 전체 단계 수행
   x(t + Δt) = x(t) + Δt · f_mid
```

### 7.3 수학적 표현

```
k₁ = f(x(t), t)
k₂ = f(x(t) + (Δt/2)k₁, t + Δt/2)
x(t + Δt) = x(t) + Δt · k₂
```

### 7.4 구현 예시

```cpp
void midpointStep(System& sys, float h) {
    float t = sys.getTime();
    vector<float> x0 = sys.getState();

    // a. 중점까지의 오일러 단계
    vector<float> f0 = sys.derivEval(x0, t);
    vector<float> x_mid = x0 + (h/2.0f) * f0;

    // b. 중점에서 도함수 평가
    vector<float> f_mid = sys.derivEval(x_mid, t + h/2.0f);

    // c. 중점 값으로 전체 단계
    sys.setState(x0 + h * f_mid, t + h);
}
```

## 8. 방법들의 차수 (Order of Methods)

### 8.1 차수의 의미

- **n차 방법**: 국소 오차가 O(h^(n+1))
- **전체 오차**: O(h^n)

### 8.2 방법별 차수

- **오일러 방법**: 1차 방법

  - 국소 오차: O(h²)
  - 전체 오차: O(h)

- **중점 방법**: 2차 방법
  - 국소 오차: O(h³)
  - 전체 오차: O(h²)

### 8.3 정확도 비교

같은 시간 간격에서:

- 오일러: 오차 ∝ h
- 중점: 오차 ∝ h²

따라서 h를 반으로 줄이면:

- 오일러: 오차 1/2배
- 중점: 오차 1/4배

## 9. 고급 방법들

### 9.1 더 많은 방법들

- **Runge-Kutta 4차**: 가장 널리 사용
- **Adams-Bashforth**: 다단계 방법
- **Backward Euler**: 암시적 방법
- **Verlet**: 에너지 보존 방법

### 9.2 유용한 힌트

- **오일러 방법을 사용하지 마라** (하지만 결국 사용하게 됨)
- **적응적 시간 단계 사용**: 오차에 따라 h 조정
- **시스템에 맞는 방법 선택**: 강성, 보존량 등 고려

### 9.3 적응적 시간 단계

```cpp
float adaptiveStep(System& sys, float h_initial, float tolerance) {
    // 큰 단계로 계산
    vector<float> x1 = oneStep(sys, h_initial);

    // 작은 단계 두 번으로 계산
    vector<float> x2 = oneStep(sys, h_initial/2);
    x2 = oneStep(sys, h_initial/2);

    // 오차 추정
    float error = norm(x1 - x2);

    if (error < tolerance) {
        return h_initial * 1.2f;  // 시간 단계 증가
    } else {
        return h_initial * 0.8f;  // 시간 단계 감소
    }
}
```

## 10. 모듈화 구현 (Modular Implementation)

### 10.1 설계 원칙

- **재사용 가능한 솔버 코드**
- **모델 구현 단순화**
- **명확한 인터페이스 정의**

### 10.2 일반적 연산들

- **차원 얻기**: dim(x)
- **상태 얻기/설정**: get/set x and t
- **도함수 평가**: Deriv Eval at current (x,t)

### 10.3 솔버 인터페이스

```cpp
class DifferentialEquationSystem {
public:
    virtual int getDimension() = 0;
    virtual vector<float> getState() = 0;
    virtual void setState(const vector<float>& state, float time) = 0;
    virtual vector<float> derivEval(const vector<float>& state, float time) = 0;
    virtual float getTime() = 0;
};
```

## 11. 솔버 인터페이스 다이어그램

### 11.1 구조

```
┌─────────┐    Get/Set State     ┌────────┐
│ System  │←───────────────────→│ Solver │
│         │    Deriv Eval       │        │
│         │←───────────────────→│        │
│         │    Dim(state)       │        │
└─────────┘                     └────────┘
```

### 11.2 인터페이스 메서드

- **Get/Set State**: 현재 시스템 상태 교환
- **Deriv Eval**: 주어진 상태에서 도함수 계산
- **Dim(state)**: 상태 벡터의 차원 수

## 12. 코드 예시 분석

### 12.1 오일러 단계 구현

```cpp
void eulerStep(Sys sys, float h) {
    float t = getTime(sys);
    vector<float> x0, deltaX;

    x0 = getState(sys);                    // 현재 상태 얻기
    deltaX = derivEval(sys, x0, t);        // 도함수 계산
    setState(sys, x0 + h*deltaX, t+h);     // 새로운 상태 설정
}
```

### 12.2 일반화된 솔버 구조

```cpp
template<typename System>
class ODESolver {
    System* system;

public:
    void step(float h) {
        // 솔버별 구현 (오일러, 중점, RK4 등)
    }

    void solve(float t_start, float t_end, float h) {
        float t = t_start;
        while (t < t_end) {
            step(h);
            t += h;
        }
    }
};
```

## 13. 실제 구현 고려사항

### 13.1 성능 최적화

- **벡터 연산 최적화**: SIMD 활용
- **메모리 지역성**: 캐시 친화적 데이터 구조
- **병렬화**: OpenMP, CUDA 등 활용

### 13.2 수치적 안정성

- **조건수 확인**: 시스템의 강성 정도 평가
- **스케일링**: 변수들의 크기 차이 고려
- **반올림 오차**: double precision 고려

### 13.3 디버깅 및 검증

- **에너지 보존 확인**: 보존 시스템에서
- **대칭성 확인**: 가역적 시스템에서
- **알려진 해와 비교**: 단순한 경우

## 14. 응용 분야

### 14.1 물리 시뮬레이션

- **입자 시스템**: 중력, 전자기력
- **강체 동역학**: 회전, 충돌
- **유체 시뮬레이션**: Navier-Stokes 방정식

### 14.2 컴퓨터 그래픽스

- **애니메이션**: 키프레임 보간
- **절차적 생성**: 성장 패턴
- **시각 효과**: 연기, 불, 물

### 14.3 기타 분야

- **제어 시스템**: 로봇 공학
- **경제 모델**: 시장 동역학
- **생물학**: 개체군 동역학

## 15. 요약

### 15.1 핵심 개념

- **미분방정식**: 연속적 변화를 기술하는 수학적 도구
- **벡터장**: 공간의 각 점에서 정의된 방향과 크기
- **적분 곡선**: 벡터장을 따라 그어진 해의 궤적

### 15.2 수치해법의 특징

- **근사해**: 정확한 해석해를 구할 수 없을 때 사용
- **오차와 안정성**: 트레이드오프 관계
- **적응적 방법**: 자동으로 정확도 조절

### 15.3 실용적 조언

- **단순함부터 시작**: 오일러 → 중점 → RK4
- **검증 필수**: 알려진 해와 비교
- **적응적 방법 사용**: 효율성과 정확성 균형
