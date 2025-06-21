# Affine Geometry - 강의 노트 정리

## 1. Geometric Programming

### 1.1 개요

- 벡터, 점, 변환과 같은 기하학적 개체를 다루는 방법
- 전통적으로 컴퓨터 그래픽스 패키지는 동차 좌표계(homogeneous coordinates)를 사용하여 구현
- 이 강의에서는 아핀 기하학과 좌표 독립적(coordinate-free) 기하학적 프로그래밍을 다룸

### 1.2 좌표 의존성의 문제점

#### 문제 상황: 두 점의 "합"

- Point p와 Point q가 있을 때, 이들의 합은 무엇인가?
- 좌표를 가정하면: p = (x₁, y₁), q = (x₂, y₂)
- 합 = (x₁ + x₂, y₁ + y₂)

#### 문제점

1. 이것이 기하학적으로 의미가 있는가?
2. 원점을 기준으로 한 벡터 합으로 해석 가능
3. **하지만 다른 원점을 선택하면 다른 결과가 나옴!**

## 2. Vector Space vs Affine Space

### 2.1 Vector Space (벡터 공간)

- 벡터와 관련 연산들을 포함
- **점(points)은 포함하지 않음**

### 2.2 Affine Space (아핀 공간)

- 벡터 공간의 상위 집합(superset)
- 벡터, 점, 그리고 관련 연산들을 포함

## 3. Points and Vectors의 구분

### 3.1 정의

- **Point**: 좌표값으로 지정된 위치
- **Vector**: 두 점 사이의 차이로 지정됨

### 3.2 관계

- 원점이 지정되면, 점은 원점으로부터의 벡터로 표현 가능
- 하지만 좌표 독립적 개념에서 **점은 여전히 벡터가 아님**

## 4. 좌표 독립적 기하학 연산

### 4.1 덧셈 (Addition)

| 연산                    | 결과   |
| ----------------------- | ------ |
| u + v (vector + vector) | vector |
| p + w (point + vector)  | point  |

### 4.2 뺄셈 (Subtraction)

| 연산                    | 결과   |
| ----------------------- | ------ |
| u - v (vector - vector) | vector |
| p - q (point - point)   | vector |
| p - w (point - vector)  | point  |

### 4.3 스칼라 곱 (Scalar Multiplication)

| 연산                | 결과      |
| ------------------- | --------- |
| scalar · vector     | vector    |
| 1 · point           | point     |
| 0 · point           | vector    |
| c · point (c ≠ 0,1) | undefined |

### 4.4 선형 결합 (Linear Combination)

```
∑ᵢ cᵢvᵢ = c₀v₀ + c₁v₁ + ... + cₙvₙ = v
```

- 선형 공간은 기저 벡터들의 집합으로 span됨
- 공간의 모든 점은 기저의 선형 결합으로 표현 가능

### 4.5 아핀 결합 (Affine Combination)

```
∑ᵢ cᵢpᵢ = p₀ + ∑ᵢ₌₁ⁿ cᵢ(pᵢ - p₀)  when ∑cᵢ = 1
```

결과:

- ∑cᵢ = 1일 때: **point**
- ∑cᵢ = 0일 때: **vector**
- 그 외: **undefined**

## 5. 예시

- **(p + q)/2**: 선분 pq의 중점
- **(p + q)/3**: 기하학적 의미를 찾을 수 있나? (undefined - 계수의 합이 2/3)
- **(p + q + r)/3**: 삼각형 pqr의 무게중심
- **(p/2 + q/2 - r)**: r에서 p와 q의 중점으로의 벡터

## 6. Affine Frame (아핀 좌표계)

### 6.1 정의

- 벡터 집합 {vᵢ | i = 1, ..., N}과 점 o로 정의
- {vᵢ}는 연관된 벡터 공간의 기저
- o는 프레임의 원점
- N은 아핀 공간의 차원

### 6.2 표현

- 임의의 점 p: **p = o + c₁v₁ + c₂v₂ + ... + cₙvₙ**
- 임의의 벡터 v: **v = c₁v₁ + c₂v₂ + ... + cₙvₙ**

## 7. Barycentric Coordinates (무게중심 좌표)

점들 qᵢ (i = 0,1,...,N)를 정의:

- q₀ = o
- qᵢ = o + vᵢ (i = 1,...,N)

그러면:

```
p = c₀q₀ + c₁q₁ + ... + cₙqₙ
```

여기서 c₀ = 1 - c₁ - ... - cₙ

## 8. Homogeneous Coordinates (동차 좌표)

### 8.1 표현

3차원 공간에서:

- **Point**: (x, y, z, 1)
- **Vector**: (x, y, z, 0)

### 8.2 연산 예시

- (x₁, y₁, z₁, 1) + (x₂, y₂, z₂, 1) = (x₁+x₂, y₁+y₂, z₁+z₂, 2) → undefined
- (x₁, y₁, z₁, 1) - (x₂, y₂, z₂, 1) = (x₁-x₂, y₁-y₂, z₁-z₂, 0) → vector
- (x₁, y₁, z₁, 1) + (x₂, y₂, z₂, 0) = (x₁+x₂, y₁+y₂, z₁+z₂, 1) → point

## 9. Projective Space (투영 공간)

### 9.1 동차 좌표의 확장

- (x, y, z, w) = (x/w, y/w, z/w, 1)
- 투시 투영(perspective projection) 처리에 유용

### 9.2 문제점

**대수적으로 일관성이 없음!**

예시:

```
(1,0,0,1) + (1,1,0,1) = (2,1,0,2) = (1, 1/2, 0, 1)
(1,0,0,1) + (2,2,0,2) = (3,2,0,3) = (1, 2/3, 0, 1)
```

동일한 점들의 합이 다른 결과를 낳음

## 10. 핵심 연산 규칙 정리

1. point + point = undefined
2. point - point = vector
3. point ± vector = point
4. vector ± vector = vector
5. scalar · vector = vector
6. ∑ scalar · vector = vector
7. scalar · point =
   - point (if scalar = 1)
   - vector (if scalar = 0)
   - undefined (otherwise)
8. ∑ scalar · point =
   - point (if ∑ scalar = 1)
   - vector (if ∑ scalar = 0)
   - undefined (otherwise)

## 요약

**Vector Space**

- 벡터와 관련 연산만 포함
- 점은 없음

**Affine Space**

- 벡터 공간의 상위집합
- 벡터, 점, 그리고 관련 연산 포함
- **연산이 제한적임!** (기하학적 의미를 보존하기 위해)
