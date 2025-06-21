# Keyframing & Curves - 강의 노트 정리

## 1. Motion과 Keyframing의 기본 개념

### 1.1 Motion의 정의

- **Motion**: Body local coordinate system에서 world coordinate system으로의 시간에 따라 변하는 변환(time-varying transformation)
- 수학적 표현: T(t)
- 시간에 따라 객체의 위치와 방향이 변화하는 것을 나타냄

### 1.2 Keyframing의 정의

- **Keyframing**: 특정 시점에서의 변환을 정의하는 기법
- T₁ = T(t₁), T₂ = T(t₂), T₃ = T(t₃) 형태로 핵심 프레임에서의 변환을 지정
- 중간 프레임들은 보간(interpolation)을 통해 생성

### 1.3 Transformation의 종류

#### 1) Rigid Transformation (강체 변환)

- **구성**: Rotation + Translation
- **표현**: 3×3 orthogonal matrix + 3-vector
- **수식**: T: x → Rx + b

#### 2) Affine Transformation (아핀 변환)

- **구성**: Scale + Shear + Rigid Transformation
- **표현**: 3×3 matrix + 3-vector
- **수식**: T: x → Ax + b

#### 3) Homogeneous Transformation (동차 변환)

- **구성**: Projective + Affine Transformation
- **표현**: 4×4 homogeneous matrix
- **수식**: T: x → Hx

#### 4) General Transformation (일반 변환)

- **특징**: Free-form deformation 포함

## 2. Spline Curves 개요

### 2.1 Classical Approach

- 전통적으로 flexible metal을 사용한 곡선 생성
- Energy minimization 원리에 기반

### 2.2 Smoothness의 개념

- **Geometric modeling에서의 smoothness**:
  - Parametric curve: x(t), y(t), ...
  - Geometric continuity 중심
- **Animation에서의 smoothness**:
  - Parametric continuity 중심
  - 곡선의 미분값들의 연속성이 중요

## 3. Polynomial Interpolation

### 3.1 Linear Interpolation

- 1차 다항식을 사용한 보간
- **수식**: x(t) = a₁t + a₀
- **행렬 형태**:
  ```
  [1  t₀] [a₀]   [x₀]
  [1  t₁] [a₁] = [x₁]
  ```

### 3.2 Quadratic Interpolation

- 2차 다항식을 사용한 보간
- **수식**: x(t) = a₂t² + a₁t + a₀
- 3개의 점을 통과하는 포물선 생성

### 3.3 General Polynomial Interpolation

- n차 다항식을 사용한 보간
- **수식**: x(t) = aₙtⁿ + aₙ₋₁tⁿ⁻¹ + ... + a₁t + a₀
- n+1개의 점을 정확히 통과

### 3.4 Polynomial Curve의 장점

- **Compact**: 적은 데이터로 곡선 표현
- **Resolution independence**: 해상도에 독립적
- **Simple**: 계산이 단순
- **Efficient**: 효율적인 처리
- **Easy to manipulate**: 조작이 용이
- **Historical reasons**: 역사적 이유

### 3.5 Lagrange Polynomial

- 가중합 형태로 표현
- **수식**: x(t) = L₀(t)x₀ + L₁(t)x₁ + ... + Lₙ(t)xₙ
- **Cardinal function**: Lₖ(tᵢ) = 1 if k=i, 0 if k≠i

### 3.6 Polynomial Interpolation의 한계

- **Oscillations at the ends**: 끝부분에서 진동 발생
- 고차 다항식 보간은 현재 거의 사용되지 않음

## 4. Piecewise Curves

### 4.1 Piecewise Curve의 개념

- 여러 개의 곡선 조각을 연결한 형태
- 각 구간에서 다른 함수를 사용
- f(t) = fᵢ(t) for t ∈ [tᵢ, tᵢ₊₁]

### 4.2 Spline Curves의 정의

- **Piecewise polynomial**: 구간별 다항식
- 각 구간 [tₖ, tₖ₊₁]에서 cubic spline의 경우:
  - x(t) = aₓt³ + bₓt² + cₓt + dₓ
  - y(t) = aᵧt³ + bᵧt² + cᵧt + dᵧ
  - z(t) = aᵣt³ + bᵣt² + cᵣt + dᵣ
- t는 각 구간에 대해 normalized
- 각 구간마다 다른 계수를 가짐

### 4.3 Control of Spline Curves

- **Control points**: 곡선의 형태를 결정하는 점들
- **Control polygon**: control points를 연결한 다각형
- **분류**:
  1. **Interpolating**: control points를 통과
  2. **Approximating**: control points에 의해 안내됨
  3. **Hybrid**: 일부 control points만 통과

### 4.4 Piecewise Linear Curves

- 각 곡선 조각이 직선인 경우
- **Vector form**: p(t) = (p₁ - p₀)t + p₀
- **Matrix form**: p(t) = [t 1][-1 1; 1 0][p₀; p₁]
- **Basis function form**: p(t) = (1-t)p₀ + tp₁
- Affine combination으로 표현 가능

## 5. Bezier Curves

### 5.1 de Casteljau Algorithm

- **Progressive subdivision** 방식
- 재귀적으로 중간점을 계산하여 곡선 생성
- Triangle form으로 표현 가능

### 5.2 Blossom Notation

- Generalized De Casteljau step
- **Control polygon**: bᵢ = b[0^(n-i), 1^i]
- **Bezier curve**: b[t^n] = (1-t)b[0^(n-r-1), t^(r-1), 1^i] + tb[0^(n-r), t^(r-1), 1^(i+1)]

### 5.3 Bernstein Polynomial

- **Basis form**: p(t) = Σ Bᵢⁿ(t)pᵢ
- **Bernstein polynomial**: Bᵢⁿ(t) = (n choose i)(1-t)^(n-i)t^i
- Cubic Bezier의 경우: p(t) = (1-t)³p₀ + 3t(1-t)²p₁ + 3t²(1-t)p₂ + t³p₃

### 5.4 Bezier Curve의 속성

- **Convex hull property**: 곡선이 control polygon의 convex hull 내부에 존재
- **Affine invariance**: affine 변환에 불변
- **Partition of unity**: ΣBᵢⁿ(t) = 1
- **Variation diminishing**: 곡선이 control polygon보다 덜 진동
- **End point interpolation**: p(0) = b₀, p(1) = b₃

## 6. Hermite Interpolation

### 6.1 Cubic Hermite Interpolation

- 두 점과 그 점에서의 접선이 주어진 경우
- **End-point constraints**: p(0) = p₀, p(1) = p₁
- **Tangent constraints**: p'(0) = v₀, p'(1) = v₁

### 6.2 Hermite 수식 유도

```
p(t) = t³a + t²b + tc + d
p'(t) = 3t²a + 2tb + c
```

경계 조건 적용:

- p(0) = d = p₀
- p'(0) = c = v₀
- p'(1) = 3a + 2b + v₀ = v₁
- p(1) = a + b + v₀ + p₀ = p₁

### 6.3 Matrix Form

```
p(t) = [t³ t² t 1] [ 2  -2   1   1] [p₀]
                   [-3   3  -2  -1] [p₁]
                   [ 0   0   1   0] [v₀]
                   [ 1   0   0   0] [v₁]
```

### 6.4 Hermite Basis Functions

- p(t) = α₀p₀ + α₁p₁ + α₂t₀ + α₃t₁
- 각 basis function은 position 또는 tangent에 대응

### 6.5 Bezier Form으로의 변환

- **Tangent properties**:
  - ṗ(0) = 3(p₁ - p₀) = v₀
  - ṗ(1) = 3(p₃ - p₂) = v₁
- **Internal Bezier points**:
  - p₁ = p₀ + v₀/3
  - p₂ = p₃ - v₁/3

## 7. Spline의 종류

### 7.1 Uniform Hermite Spline

- Piecewise cubic Hermite interpolation
- 각 구간에서 Hermite interpolation 적용
- C¹ continuity 보장

### 7.2 Non-uniform Hermite Spline

- Knot spacing이 균일하지 않은 경우
- 각 구간의 길이에 따라 tangent 조정:
  - First segment: Hermite(p₀, p₁, v₀/t₀, v₁/t₀)
  - Second segment: Hermite(p₁, p₂, v₁/t₁, v₂/t₂)

### 7.3 Catmull-Rom Spline

- Automatic tangent determination
- **Tangent 계산**: p'(tᵢ) = (pᵢ₊₁ - pᵢ₋₁)/2
- C¹ continuity
- Convex hull property 없음 (basis function이 음수 값을 가질 수 있음)

### 7.4 Bessel Tangents

- Parabola interpolation을 사용한 tangent 계산
- **수식**: vᵢ = d/dt qᵢ(tᵢ)
- Overhauser spline = Hermite spline with Bessel tangent

## 8. Continuity

### 8.1 Continuity의 종류

- **C⁰ continuity**: Position이 양쪽에서 일치
- **C¹ continuity**: Tangent가 양쪽에서 일치
- **C² continuity**: Second derivative가 양쪽에서 일치

### 8.2 Geometric vs Parametric Continuity

- **Gⁿ continuity**: Arc-length parameterization 기준
- **Cⁿ continuity**: Parameter 기준
- Animation에서는 주로 Cⁿ continuity가 중요

## 9. B-Splines

### 9.1 Quadratic B-Spline

- C¹ continuity
- Local control 제공

### 9.2 Cubic B-Splines

- C² continuity + local control
- **Basis form**: p(t) = b₀B₀³(t) + b₁B₁³(t) + b₂B₂³(t) + b₃B₃³(t)

### 9.3 B-Spline Basis Functions

재귀적 정의:

- B₀ⁱ(t) = 1 if tᵢ₋₁ ≤ t < tᵢ, 0 otherwise
- Bᵢⁿ(t) = ((t-tᵢ₋₁)/(tᵢ₊ₙ₋₁-tᵢ₋₁))Bᵢⁿ⁻¹(t) + ((tᵢ₊ₙ-t)/(tᵢ₊ₙ-tᵢ))Bᵢ₊₁ⁿ⁻¹(t)

### 9.4 Uniform Cubic B-Spline Basis

```
B₀³(t) = (1/6)(1-t)³
B₁³(t) = (1/6)(3t³ - 6t² + 4)
B₂³(t) = (1/6)(-3t³ + 3t² + 3t + 1)
B₃³(t) = (1/6)t³
```

### 9.5 B-Spline Properties

- **Convex hull property**
- **Affine invariance**
- **Variation diminishing**
- **C² continuity**
- **Local controllability**: 한 control point 변경이 제한된 구간에만 영향

## 10. Interpolating Cubic Spline

### 10.1 C² Interpolating Spline

- Piecewise cubic with C² continuity
- n+1 control points → n segments → 4n unknowns
- **Conditions**:
  - C⁰ continuity: 2n equations
  - C¹ continuity: n-1 equations
  - C² continuity: n-1 equations
  - Total: 4n-2 equations (2 추가 조건 필요)

### 10.2 End Conditions

1. **Natural end condition**: p̈(0) = p̈(n) = 0
2. **Quadratic end condition**: p̈(0) = p̈(t₁)
3. **Not-a-knot condition**: p⃛(t₁) is continuous
4. **Closed condition**: p̈(0) = p̈(tₙ)
5. **Bessel end condition**: Bessel tangent 사용

### 10.3 Solving System

중간 변수 사용:

- aₖ = pₖ
- bₖ = Dₖ (first derivative)
- cₖ = 3(pₖ₊₁ - pₖ) - 2Dₖ - Dₖ₊₁
- dₖ = 2(pₖ - pₖ₊₁) + Dₖ + Dₖ₊₁

Tridiagonal system으로 해결 가능

## 11. NURBS (Non-Uniform Rational B-Splines)

### 11.1 특징

- **Non-uniform knot spacing**: 불균일한 knot 간격
- **Rational polynomial**: 다항식/다항식 형태
- **Conic sections 표현 가능**: 원, 타원, 쌍곡선
- **Projective transformation에 불변**

### 11.2 관계

- Uniform B-spline ⊂ Non-uniform B-spline
- Non-rational B-spline ⊂ Rational B-spline
- B-spline ⊂ NURBS

## 12. Summary

### 12.1 Polynomial Interpolation

- Lagrange polynomial: 모든 점을 통과하는 단일 다항식
- 고차 다항식의 oscillation 문제

### 12.2 Spline Interpolation

- **Piecewise polynomial**: 구간별 다항식
- **Knot sequence**: 구간을 나누는 점들
- **Continuity across knots**: 연결점에서의 연속성

### 12.3 주요 Spline 종류

1. **Natural spline**: C² continuity, 끝점에서 가속도 = 0
2. **Catmull-Rom spline**: C¹ continuity, 자동 tangent 계산
3. **Hermite spline**: C¹ continuity, tangent 직접 지정
4. **B-spline**: C² continuity, local control, approximating

### 12.4 Basis Functions

- **Bezier basis**: Bernstein polynomials
- **B-spline basis**: Bell-shaped, overlapping functions
- **Hermite basis**: Position과 tangent 기반

### 12.5 선택 기준

- **Interpolation 필요**: Catmull-Rom, Hermite, Natural spline
- **Local control 필요**: B-spline
- **Simple segment**: Bezier
- **High continuity**: B-spline (C²), Natural spline (C²)
- **Conic sections**: NURBS
