# Particle Systems 강의 노트 (SIGGRAPH 2001)

## 1. 강의 개요

### 1.1 강의 내용 구성

- **하나의 기본 입자 (One Lousy Particle)**
- **입자 시스템 (Particle Systems)**
- **힘의 종류**: 중력, 스프링 등
- **구현 및 상호작용 (Implementation and Interaction)**
- **간단한 충돌 처리 (Simple Collisions)**

## 2. 뉴턴 입자 (Newtonian Particle)

### 2.1 기본 물리 법칙

뉴턴의 제2법칙을 기반으로 한 미분방정식:

```
f = ma
```

### 2.2 힘의 의존성

힘은 다음 요소들에 의존할 수 있음:

- **위치 (Position)**: x
- **속도 (Velocity)**: ẋ
- **시간 (Time)**: t

수학적 표현:

```
ẍ = f(x, ẋ, t) / m
```

## 3. 2차 미분방정식의 처리

### 3.1 문제점

- 표준 형태가 아님 (2차 도함수 포함)
- 일반적인 ODE 솔버로 직접 해결 불가

### 3.2 해결 방법: 새로운 변수 도입

새로운 변수 v를 도입하여 연결된 1차 방정식 쌍으로 변환:

**원래 방정식:**

```
ẍ = f(x, ẋ, t) / m
```

**변환된 방정식:**

```
ẋ = v
v̇ = f / m
```

## 4. 위상 공간 (Phase Space)

### 4.1 위상 공간의 개념

- **x와 v를 6차원 벡터로 연결**: 위상 공간에서의 위치
- **위상 공간에서의 속도**: 또 다른 6차원 벡터
- **결과**: 표준 1차 미분방정식

### 4.2 수학적 표현

```
[x]     [v  ]
[v] = [f/m]
```

여기서 왼쪽은 위상 공간에서의 상태, 오른쪽은 상태의 변화율

## 5. 입자 구조 (Particle Structure)

### 5.1 입자의 기본 속성

각 입자는 다음 정보를 포함:

```cpp
struct Particle {
    Vector3 x;  // 위치 (Position)
    Vector3 v;  // 속도 (Velocity)
    Vector3 f;  // 힘 누적기 (Force Accumulator)
    float m;    // 질량 (Mass)
};
```

### 5.2 위상 공간에서의 위치

- **위치**: (x, v) - 6차원 벡터
- **속도**: (v, f/m) - 6차원 벡터

## 6. 솔버 인터페이스 (Solver Interface)

### 6.1 기본 메서드

```cpp
class ParticleSystem {
public:
    void GetState(Vector& state);           // 상태 얻기
    void SetState(const Vector& state);     // 상태 설정
    void DerivEval(Vector& deriv);          // 도함수 계산
    int Dim();                              // 상태 차원 수
};
```

### 6.2 상태 벡터 구성

단일 입자의 경우:

- **입력**: [x, v] (6차원)
- **출력**: [v, f/m] (6차원)

## 7. 입자 시스템 (Particle Systems)

### 7.1 다중 입자 확장

n개의 입자로 구성된 시스템:

- **상태 차원**: 6n
- **각 입자**: 독립적인 위치, 속도, 힘, 질량

### 7.2 시스템 구조

```cpp
class ParticleSystem {
    vector<Particle> particles;
    vector<Force*> forces;
    float time;
};
```

### 7.3 솔버 인터페이스 확장

```cpp
// 6n x 1 벡터
State = [x₁, v₁, x₂, v₂, ..., xₙ, vₙ]
Deriv = [v₁, f₁/m₁, v₂, f₂/m₂, ..., vₙ, fₙ/mₙ]
```

## 8. 도함수 계산 루프 (Deriv Eval Loop)

### 8.1 계산 단계

1. **힘 초기화 (Clear Forces)**

   - 모든 입자의 힘 누적기를 0으로 설정

2. **힘 계산 (Calculate Forces)**

   - 모든 힘을 누적기에 합산

3. **수집 (Gather)**
   - 모든 입자를 순회하며 v와 f/m을 대상 배열에 복사

### 8.2 구현 예시

```cpp
void ParticleSystem::DerivEval(Vector& deriv) {
    // 1. Clear forces
    for (auto& p : particles) {
        p.f = Vector3(0, 0, 0);
    }

    // 2. Calculate forces
    for (auto& force : forces) {
        force->apply_force(*this);
    }

    // 3. Gather
    int index = 0;
    for (auto& p : particles) {
        deriv[index++] = p.v;
        deriv[index++] = p.f / p.m;
    }
}
```

## 9. 힘의 종류 (Forces)

### 9.1 힘의 분류

- **상수 중력 (Constant Gravity)**
- **위치/시간 의존 힘장 (Position/Time Dependent Force Fields)**
- **속도 의존 저항 (Velocity-Dependent Damping)**
- **n진 스프링 (n-ary Springs)**

### 9.2 힘 구조의 특징

- **이질성**: 입자와 달리 힘은 이질적
- **힘 객체**: 블랙박스 형태
- **영향 범위**: 영향을 주는 입자들을 가리킴
- **타입별 계산**: 각자의 방식으로 힘 추가

## 10. 구체적인 힘의 구현

### 10.1 중력 (Gravity)

#### 물리 법칙

```
f_gravity = m * g
```

#### 구현

```cpp
class GravityForce : public Force {
    Vector3 g;  // 중력 가속도

public:
    void apply_force(ParticleSystem& sys) override {
        for (auto& p : sys.particles) {
            p.f += p.m * g;
        }
    }
};
```

### 10.2 점성 저항 (Viscous Drag)

#### 물리 법칙

```
f_drag = -k_drag * v
```

#### 구현

```cpp
class DragForce : public Force {
    float k_drag;  // 저항 계수

public:
    void apply_force(ParticleSystem& sys) override {
        for (auto& p : sys.particles) {
            p.f -= k_drag * p.v;
        }
    }
};
```

### 10.3 감쇠 스프링 (Damped Spring)

#### 물리 법칙

```
Δx = x₁ - x₂
f = -k_s * (|Δx| - rest_length) * (Δx/|Δx|) - k_d * (Δv · Δx/|Δx|) * (Δx/|Δx|)
```

#### 구현

```cpp
class SpringForce : public Force {
    Particle* p1, *p2;
    float k_spring, k_damping;
    float rest_length;

public:
    void apply_force(ParticleSystem& sys) override {
        Vector3 delta_x = p1->x - p2->x;
        Vector3 delta_v = p1->v - p2->v;
        float length = delta_x.magnitude();

        if (length > 0) {
            Vector3 direction = delta_x / length;
            Vector3 spring_force = -k_spring * (length - rest_length) * direction;
            Vector3 damping_force = -k_damping * dot(delta_v, direction) * direction;
            Vector3 total_force = spring_force + damping_force;

            p1->f += total_force;
            p2->f -= total_force;
        }
    }
};
```

## 11. 힘 계산 과정

### 11.1 전체 과정

1. **힘 누적기 초기화**
2. **모든 힘 객체의 apply_force 함수 호출**
3. **결과를 [v, f/m, ...] 형태로 솔버에 반환**

### 11.2 힘 객체 관리

```cpp
class ParticleSystem {
    vector<Force*> forces;

public:
    void add_force(Force* f) { forces.push_back(f); }
    void clear_forces() { forces.clear(); }
};
```

## 12. 충돌 처리 (Collision Handling)

### 12.1 기본 개념

- **나중에**: 강체 충돌 및 접촉 처리
- **현재**: 간단한 점-평면 충돌
- **입자 시뮬레이터의 부가 기능**

### 12.2 벽과의 충돌

#### 법선 및 접선 성분 분해

```
V_N = (N · V) * N    // 법선 성분
V_T = V - V_N        // 접선 성분
```

### 12.3 충돌 감지 (Collision Detection)

#### 조건

1. **벽 근처에 있음**: `(X - P) · N < ε`
2. **벽을 향해 이동**: `N · V < 0`

```cpp
bool is_colliding(const Vector3& pos, const Vector3& vel,
                 const Vector3& wall_point, const Vector3& wall_normal) {
    float distance = dot(pos - wall_point, wall_normal);
    float velocity_toward_wall = dot(vel, wall_normal);

    return (distance < epsilon) && (velocity_toward_wall < 0);
}
```

### 12.4 충돌 응답 (Collision Response)

#### 속도 수정

```
V' = V_T - k_r * V_N
```

여기서:

- `k_r`: 반발 계수 (0 ≤ k_r ≤ 1)
- `k_r = 0`: 완전 비탄성 충돌
- `k_r = 1`: 완전 탄성 충돌

```cpp
Vector3 collision_response(const Vector3& velocity, const Vector3& normal, float restitution) {
    Vector3 v_normal = dot(velocity, normal) * normal;
    Vector3 v_tangential = velocity - v_normal;

    return v_tangential - restitution * v_normal;
}
```

## 13. 접촉 처리 (Contact Handling)

### 13.1 접촉 조건

1. **벽 위에 있음**: `(X - P) · N < ε`
2. **벽을 따라 이동**: `N · V < ε`
3. **벽을 밀고 있음**: `N · F < ε`

### 13.2 접촉력 (Contact Force)

벽이 밀어내어 F의 법선 성분을 상쇄:

```
F' = F_T - F_N
```

이는 **제약력(constraint force)**의 예시

```cpp
Vector3 contact_force(const Vector3& applied_force, const Vector3& normal) {
    Vector3 f_normal = dot(applied_force, normal) * normal;
    Vector3 f_tangential = applied_force - f_normal;

    // 법선 성분 제거 (벽이 지지)
    return f_tangential;
}
```

## 14. 기본 2D 상호작용 (Basic 2-D Interaction)

### 14.1 마우스 연산

- **생성 (Create)**: 새로운 입자 생성
- **부착 (Attach)**: 마우스 스프링 연결
- **드래그 (Drag)**: 입자 이동
- **못 박기 (Nail)**: 입자 고정

### 14.2 상호작용 요소

#### 마우스 스프링 (Mouse Spring)

```cpp
class MouseSpring : public SpringForce {
    Vector3 mouse_position;

public:
    void update_mouse_position(const Vector3& pos) {
        mouse_position = pos;
    }

    void apply_force(ParticleSystem& sys) override {
        Vector3 delta = particle->x - mouse_position;
        Vector3 spring_force = -k_spring * delta;
        particle->f += spring_force;
    }
};
```

#### 못 (Nail)

```cpp
class Nail : public Force {
    Particle* particle;
    Vector3 fixed_position;

public:
    void apply_force(ParticleSystem& sys) override {
        // 입자를 고정 위치에 강제로 유지
        particle->x = fixed_position;
        particle->v = Vector3(0, 0, 0);
    }
};
```

## 15. 구현 고려사항

### 15.1 시스템 아키텍처

```cpp
class ParticleSimulator {
    ParticleSystem system;
    ODESolver solver;
    CollisionHandler collision_handler;
    InteractionManager interaction_manager;

public:
    void update(float dt);
    void render();
    void handle_input(const Input& input);
};
```

### 15.2 성능 최적화

- **공간 분할**: 근처 입자들만 상호작용 계산
- **적응적 시간 단계**: 시스템 안정성에 따라 조정
- **힘 캐싱**: 비싼 계산 결과 재사용

### 15.3 수치적 안정성

- **적절한 시간 단계 선택**
- **강성 시스템 처리**: implicit 적분법 고려
- **에너지 보존**: symplectic 적분법 사용

## 16. 요약

### 16.1 핵심 개념

- **입자 시스템**: 간단하지만 강력한 물리 시뮬레이션 기법
- **모듈화**: 힘, 입자, 솔버의 독립적 구현
- **확장성**: 새로운 힘과 제약 조건 쉽게 추가 가능

### 16.2 실제 응용

- **시각 효과**: 연기, 불, 폭발, 천 시뮬레이션
- **게임**: 물리 엔진의 기초
- **과학적 시뮬레이션**: 분자 동역학, 천체 물리학

### 16.3 학습 가치

- **물리 기반 애니메이션의 기초**
- **수치 해석 방법의 실제 적용**
- **객체 지향 설계의 좋은 예시**
