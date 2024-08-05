+++
categories = ['Database']
series = ['Database-System']
tags = ['데이터베이스']
date = 2024-07-20T22:50:44+09:00
draft = true
+++
## 관계형 데이터베이스 설계
핵심은 무손실 분해, dependency preservation.
R1, R2의 공통된 키가 수퍼키이면 non-loss.


### Dependency preservation


### Boyce-Codd Normal Form (BCNF)
Boyce-Codd Normal Form (BCNF)는 데이터베이스 정규화의 한 형태로, 데이터베이스 설계에서 이상 현상을 최소화하고 데이터 무결성을 유지하기 위해 사용됩니다. BCNF는 제3정규형(3NF)보다 더 강력한 조건을 요구합니다. BCNF는 다음과 같은 조건을 만족해야 합니다:

1. **모든 함수 종속성은 자명해야 한다**: 자명한 함수 종속성은 종속성의 오른쪽 부분이 왼쪽 부분의 부분 집합인 경우입니다. 예를 들어, \( \alpha \rightarrow \beta \)에서 \( \beta \subseteq \alpha \)인 경우입니다.
2. **왼쪽 부분이 슈퍼키여야 한다**: 슈퍼키는 릴레이션의 모든 튜플을 고유하게 식별할 수 있는 속성의 집합입니다. 즉, \( \alpha \)가 릴레이션 \( R \)의 슈퍼키여야 합니다.

위의 이미지는 BCNF의 정의를 설명하고 있습니다. 이를 요약하면 다음과 같습니다:

- 릴레이션 스키마 \( R \)가 함수 종속성 집합 \( F \)에 대해 BCNF에 속하려면, 모든 함수 종속성 \( \alpha \rightarrow \beta \)에 대해 다음 조건 중 하나를 만족해야 합니다:
  1. \( \alpha \rightarrow \beta \)가 자명한 함수 종속성일 때 (\( \beta \subseteq \alpha \)).
  2. \( \alpha \)가 \( R \)의 슈퍼키일 때.

이 두 조건 중 하나라도 만족하지 않는 함수 종속성이 존재하면, 해당 릴레이션 스키마는 BCNF에 속하지 않습니다. BCNF는 데이터베이스 설계에서 데이터 중복을 줄이고, 삽입, 삭제, 갱신 이상을 방지하는 데 중요한 역할을 합니다.

--
함수 종속 기반으로 발견할 수 있는 모든 중복을 제거.

릴레이션 스키마 R에서 
a => b인 모든 F+(함수종속)에 대해 
둘 중 하나라도 만족하면 R은 BCNF에 속한다.
1. a => b가 자명(b가 a의 부분집합)
2. a가 R의 슈퍼키


자명하지 않은 함수종속 a => b가 최소 하나 존재하고,
a가 R의 슈퍼키가 아니면 
R의 설계를 두 개의 스키마로 대체할 수 있다.

💎 분해하기
α → β 가 BCNF를 위반하는 FD(functional dependency)라고 해보자.
이때 R은 다음 두 개로 분해될 수 있다.
1. (α U β )
2. ( R - ( β - α ) )

ex.
```
in_dep (ID, name, salary, dept_name, building, budget )
• α = dept_name
• β = building, budget

in_dep is replaced by
• (α U β ) = ( dept_name, building, budget )
• ( R - ( β - α ) ) = ( ID, name, dept_name, salary )
```
-> 결과적으로 α 가 다른쪽에 남아있게 되는건데, 관계를 이어줄 속성이 필요하기 때문.

-- (참고)
#### Trivial vs Non-trivial
💎 자명하다(trivial)은 주로 PK에 의해 결정되는 종속성임.

자명한 함수종속: 
X => Y가 자명한 경우: Y가 X의 부분집합인 경우.
X = a, b
Y = a
a,b => a는 자명. 

자명하지 않은 함수종속:
X => Y가 자명하지 않은 경우: Y가 X의 일부 속성만 포함하는 경우.
X = a, b
Y = c
함수종속 a,b => c는 자명하지 않음. 
Y(c)는 X(a,b)의 부분집합이 아니기 때문. 

자명한 함수종속은 항상 참이므로 특별한 고려가 필요하지 않으나 
자명하지 않은 함수종속은 데이터 무결성을 유지하고 데이터 중복을 최소화하는 데 중요한 기준이 됨.

#### 자명하지 않은 함수 종속이 있을 때 테이블을 BCNF로 분해하는 예제
R(a, b, c)
a => b, b => c

BCNF 조건을 만족하려면 모든 자명하지 않은 함수종속(X => Y)에 대해 X가 슈퍼키여야 함. 
1. a => b : a는 슈퍼키가 아니므로 BCNF 조건 위반
2. b => c : b는 슈퍼키가 아니므로 위반

따라서 BCNF를 만족하려면 테이블을 분해해야 함.
R1(a, b)
R2(a, c) -> b => c 기준으로 분해해 R3(b => c) 생성

최종적으로 
R1(a, b)
R3(b, c) 

---
### BCNF와 dependency preservation을 모두 충족시키는 것이 항상 가능한 것은 아니다

dept_advisor(s_ID, i_ID, department_name)

FD:
i_ID → dept_name
s_ID, dept_name → i_ID

여기서 i_ID는 슈퍼키가 아니므로(학생:교수는 N:1)
BCNF를 충족하지 않는다.

그러나 a = i_ID, b = dept_name 
R1 (i_ID, dept_name),
R2 (s_ID, i_ID)
로 분해한다 해도
s_ID, dept_name => i_ID 속성은 조인 없이 알 수 없다. 

따라서 BCNF를 충족하기 위한 분해는 dependency preserving하지 않다.

### BCNF vs 3NF
3NF는 무손실이고, dependency preservation을 희생하지 않아도 된다.
그러나 
- 속성값이 중복해서 나타나야하거나
- 다른 두 개의 관계때문에 특정 속성이 null로 나타나야할 수 있다는 단점이 있다.

정규화의 목적을 생각해보자.
릴레이션 R에 F 종속성이 있을 때,
R이 not good form일 때, R1, R2, .. Rn 릴레이션으로 분해할 수 있다.
각각은 무손실의 good form이어야하고 dependency를 보존할 수 있어야한다.

분명 모든 종속성(F)이 자명한데도 (PK에 의한 속성값)
삽입시 두 개의 튜플을 넣어야하는 상황.
-> Higher Normal Forms


💎 지금까지
- BCNF를 위한 분해 알고리즘: a => b non trivial 종속성 중 a가 not super key인 경우. 
- 3NF 분해 알고리즘: a => b, a가 슈퍼키가 아니어도 (b-a)에 속한 각 속성이 R의 후보키에 포함되면 분해하지 않아도 됨.


---
### Closure 
Closure of attribute sets는 데이터베이스 이론에서 중요한 개념입니다. **주어진 속성 집합의 closure는 해당 속성 집합이 함수 종속성 집합 \( F \) 아래에서 결정할 수 있는 모든 속성의 집합을 의미합니다.** 이를 통해 후보 키(candidate key)를 찾거나 데이터베이스 정규화를 수행할 수 있습니다.

🍋 Closure를 구하는 목적
- a가 슈퍼키인지 검증하기 위해 a+ closure(a가 결정할 수 있는 모든 속성의 집합)을 계산하고, a+가 R 릴레이션의 모든 속성을 포함하는지 본다.

#### Closure 계산 알고리즘
주어진 속성 집합 \( \alpha \)의 closure \( \alpha^+ \)를 계산하는 알고리즘은 다음과 같습니다:

1. **초기화**: 결과 집합 \( result \)를 \( \alpha \)로 초기화합니다.
2. **반복**: \( result \)가 변경될 때까지 반복합니다.
   - 각 함수 종속성 \( \beta \rightarrow \gamma \)에 대해:
     - 만약 \( \beta \subseteq result \)이면, \( result \)에 \( \gamma \)를 추가합니다.

### 예제
다음은 속성 집합 \( R \)과 함수 종속성 집합 \( F \)가 주어졌을 때, 속성 집합 \( AG \)의 closure를 계산하는 예제입니다.

- **속성 집합 \( R \)**: \( (A, B, C, G, H, I) \)
- **함수 종속성 집합 \( F \)**:
  - \( A \rightarrow B \)
  - \( A \rightarrow C \)
  - \( CG \rightarrow H \)
  - \( CG \rightarrow I \)
  - \( B \rightarrow H \)

### \( (AG)^+ \) 계산
1. **초기화**: \( result = AG \)
2. **반복**:
   - \( A \rightarrow C \)와 \( A \rightarrow B \)로 인해 \( result = ABCG \)
   - \( CG \rightarrow H \)로 인해 \( result = ABCGH \)
   - \( CG \rightarrow I \)로 인해 \( result = ABCGHI \)

따라서 \( (AG)^+ = ABCGHI \)입니다.

### AG가 후보 키인지 확인
1. **AG가 슈퍼 키인지 확인**:
   - \( (AG)^+ \)가 \( R \)을 포함하는지 확인합니다. \( (AG)^+ = ABCGHI \)이므로 \( R \)을 포함합니다. 따라서 AG는 슈퍼 키입니다. 
   -> {AG}가 R의 모든 것을 결정할 수 있다.
2. **AG의 부분 집합이 슈퍼 키인지 확인**:
   - \( A \)가 \( R \)을 포함하는지 확인합니다. \( (A)^+ \)를 계산하여 확인합니다.
      - A+ = {A,B,C,H} -> A+가 R을 포함하지 않으므로 A는 슈퍼키가 아니다.
   - \( G \)가 \( R \)을 포함하는지 확인합니다. \( (G)^+ \)를 계산하여 확인합니다.
      - G+ = {G}
   - 일반적으로 크기가 \( n-1 \)인 각 부분 집합에 대해 확인합니다.

AG 집합은 슈퍼키이나 A, G 각각은 슈퍼키가 아니다. 

---
---
## Canonical cover
이 슬라이드는 함수적 종속성에서 불필요한 속성을 제거하는 방법을 설명하고 있습니다. 이를 이해하기 위해 예를 들어 설명해보겠습니다.

### 예제 1: 왼쪽에서 속성 제거
함수적 종속성 \( \alpha \rightarrow \beta \)에서 왼쪽(즉, \(\alpha\))에서 속성 \(A\)가 불필요한지 확인하는 방법을 살펴보겠습니다.

#### 예제:
- 함수적 종속성 집합 \(F\): \{AB → C, A → B\}
- \( \alpha \rightarrow \beta \): AB → C

이제 \(A\)가 \( \alpha \)에서 불필요한지 확인해보겠습니다.
1. \(A \in \alpha\) (즉, \(A\)는 \(AB\)의 일부입니다).
2. \(F\)가 \((F - \{AB \rightarrow C\}) \cup \{(AB - A) \rightarrow C\}\)를 논리적으로 포함하는지 확인합니다.
   - \(F - \{AB \rightarrow C\} = \{A \rightarrow B\}\)
   - \((AB - A) \rightarrow C = B \rightarrow C\)
   - 따라서, \((F - \{AB \rightarrow C\}) \cup \{B \rightarrow C\} = \{A \rightarrow B, B \rightarrow C\}\)

이제 \{A → B, B → C\}가 원래의 \(F\)를 논리적으로 포함하는지 확인합니다. 여기서 A → B와 B → C를 통해 A → C를 유도할 수 있습니다. 따라서 \(A\)는 불필요한 속성입니다.

### 예제 2: 오른쪽에서 속성 제거
함수적 종속성 \( \alpha \rightarrow \beta \)에서 오른쪽(즉, \(\beta\))에서 속성 \(A\)가 불필요한지 확인하는 방법을 살펴보겠습니다.

#### 예제:
- 함수적 종속성 집합 \(F\): \{A → BC, B → C\}
- \( \alpha \rightarrow \beta \): A → BC

이제 \(C\)가 \( \beta \)에서 불필요한지 확인해보겠습니다.
1. \(C \in \beta\) (즉, \(C\)는 \(BC\)의 일부입니다).
2. \(F\)가 \((F - \{A \rightarrow BC\}) \cup \{A \rightarrow (BC - C)\}\)를 논리적으로 포함하는지 확인합니다.
   - \(F - \{A \rightarrow BC\} = \{B \rightarrow C\}\)
   - \(A \rightarrow (BC - C) = A \rightarrow B\)
   - 따라서, \((F - \{A \rightarrow BC\}) \cup \{A \rightarrow B\} = \{B \rightarrow C, A \rightarrow B\}\)

이제 \{B → C, A → B\}가 원래의 \(F\)를 논리적으로 포함하는지 확인합니다. 여기서 A → B와 B → C를 통해 A → BC를 유도할 수 있습니다. 따라서 \(C\)는 불필요한 속성입니다.

이와 같이, 함수적 종속성에서 불필요한 속성을 제거함으로써 더 간결하고 효율적인 종속성 집합을 만들 수 있습니다.

---

**Canonical Cover는 함수적 종속성의 집합을 단순화하여 중복을 제거하고 최소화된 형태로 표현한 것입니다.** 이를 통해 데이터베이스의 무결성을 유지하면서도 효율적인 설계를 할 수 있습니다.

### 첫 번째 슬라이드: Canonical Cover의 정의
- **Canonical Cover**: 함수적 종속성 집합 \( F \)에 대해, Canonical Cover \( F_c \)는 다음 조건을 만족하는 종속성 집합입니다:
  - \( F \)는 \( F_c \)의 모든 종속성을 논리적으로 포함합니다.
  - \( F_c \)는 \( F \)의 모든 종속성을 논리적으로 포함합니다.
  - \( F_c \)의 어떤 함수적 종속성도 불필요한 속성을 포함하지 않습니다.
  - \( F_c \)의 각 함수적 종속성의 왼쪽 부분은 고유합니다. 즉, \( F_c \)에는 다음과 같은 두 종속성이 존재하지 않습니다:
    - \( \alpha_1 \rightarrow \beta_1 \)와 \( \alpha_2 \rightarrow \beta_2 \)가 있고, \( \alpha_1 = \alpha_2 \)인 경우.

### 두 번째 슬라이드: Canonical Cover를 계산하는 방법
- **Canonical Cover를 계산하는 절차**:
  1. **반복**:
     - \( \alpha_1 \rightarrow \beta_1 \)와 \( \alpha_1 \rightarrow \beta_2 \) 형태의 종속성을 \( \alpha_1 \rightarrow \beta_1, \beta_2 \)로 대체하기 위해 유니온 규칙을 사용합니다.
     - \( F_c \)에서 불필요한 속성을 포함하는 함수적 종속성 \( \alpha \rightarrow \beta \)를 찾습니다.
       - 이 때, 불필요한 속성 검사는 \( F \)가 아닌 \( F_c \)를 사용하여 수행합니다.
     - 불필요한 속성이 발견되면, 이를 \( \alpha \rightarrow \beta \)에서 삭제합니다.
  2. **반복 종료 조건**: \( F_c \)가 더 이상 변경되지 않을 때까지 반복합니다.

- **참고**:
  - 유니온 규칙은 불필요한 속성이 삭제된 후 다시 적용될 수 있으므로, 반복적으로 적용해야 합니다.

이 과정을 통해 데이터베이스 설계에서 함수적 종속성을 최소화하고, 중복을 제거하여 효율적인 데이터베이스 구조를 만들 수 있습니다.


---

## Dependency Preservation
슬라이드 2에서 언급된 "제한 \( F_i \)"에 대한 설명을 더 명확하게 이해하도록 돕겠습니다.

### 정의
- **제한 \( F_i \)**: \( F \)를 \( R_i \)로 제한한 것은 \( R_i \)의 속성만 포함하는 모든 함수적 종속성의 집합 \( F_i \)입니다.

### 의미
이 말은, 전체 함수적 종속성 집합 \( F \)에서 특정 릴레이션 \( R_i \)에 속하는 속성들만 포함하는 함수적 종속성들을 추출한 것을 의미합니다. 즉, 각 분해된 릴레이션 \( R_i \)에 대해 해당 릴레이션에 적용될 수 있는 함수적 종속성만을 모은 집합을 \( F_i \)라고 합니다.

### 예제
예제를 통해 좀 더 구체적으로 설명하겠습니다.

#### 예제 스키마
- 전체 릴레이션 \( R = (A, B, C, D) \)
- 함수적 종속성 집합 \( F = \{ A \rightarrow B, B \rightarrow C, C \rightarrow D, A \rightarrow D \} \)

#### 분해
릴레이션 \( R \)을 두 개의 릴레이션으로 분해한다고 가정합시다:
- \( R_1 = (A, B) \)
- \( R_2 = (B, C, D) \)

#### 제한 \( F_i \)
- **\( F_1 \)**: \( R_1 = (A, B) \)의 속성만 포함하는 함수적 종속성
  - \( F_1 = \{ A \rightarrow B \} \)
  - 여기서 \( A \rightarrow B \)만 \( R_1 \)의 속성 \( A \)와 \( B \)를 포함하고 있으므로 \( F_1 \)에 포함됩니다.

- **\( F_2 \)**: \( R_2 = (B, C, D) \)의 속성만 포함하는 함수적 종속성
  - \( F_2 = \{ B \rightarrow C, C \rightarrow D, A \rightarrow D \} \)
  - 여기서 \( B \rightarrow C \)와 \( C \rightarrow D \)는 \( R_2 \)의 속성 \( B, C, D \)를 포함하고 있으므로 \( F_2 \)에 포함됩니다.
  - \( A \rightarrow D \)는 \( R_2 \)의 속성 \( A \)를 포함하지 않으므로 \( F_2 \)에 포함되지 않습니다.

### 요약
따라서, 제한 \( F_i \)는 전체 함수적 종속성 집합 \( F \)에서 특정 릴레이션 \( R_i \)의 속성만 포함하는 함수적 종속성들을 추출한 집합입니다. 이는 각 분해된 릴레이션 \( R_i \)에 대해 해당 릴레이션에서 유효한 함수적 종속성만을 고려하여 데이터 무결성을 유지하는 데 사용됩니다.

이해를 돕기 위해 요약하자면:
- **제한 \( F_i \)**: 전체 함수적 종속성 \( F \)에서 특정 릴레이션 \( R_i \)의 속성만 포함하는 함수적 종속성의 집합입니다.
- 이는 분해된 각 릴레이션 \( R_i \)에 대해 해당 릴레이션에서 유효한 함수적 종속성을 나타냅니다.

---

### 슬라이드 3: Testing for Dependency Preservation
1. **의존성 보존 테스트**:
   - \( R \)를 \( R_1, R_2, ..., R_n \)으로 분해할 때, 의존성 \( \alpha \rightarrow \beta \)가 보존되는지 확인하기 위해 다음 테스트를 적용합니다.
   
2. **테스트 절차**:
   - \( result = \alpha \)
   - 반복:
     - 분해된 각 \( R_i \)에 대해:
       - \( t = (result \cap R_i)^+ \cap R_i \)
       - \( result = result \cup t \)
   - \( result \)가 변경되지 않을 때까지 반복
   
3. **결과 확인**:
   - \( result \)가 \( \beta \)의 모든 속성을 포함하면, 함수적 종속성 \( \alpha \rightarrow \beta \)가 보존됩니다.
   
4. **모든 의존성에 대해 테스트**:
   - 분해가 의존성을 보존하는지 확인하기 위해 \( F \)의 모든 의존성에 대해 테스트를 적용합니다.
   
5. **시간 복잡도**:
   - 이 절차는 \( F^+ \)와 \( (F_1 \cup F_2 \cup ... \cup F_n)^+ \)를 계산하는 데 필요한 지수 시간 대신 다항 시간(polynomial time)이 소요됩니다.

### 슬라이드 4: Example
1. **예제 스키마 \( R \)**:
   - \( R = (A, B, C) \)
   - \( F = \{ A \rightarrow B, B \rightarrow C \} \)
   - 키(Key) = \{A\}
   
2. **BCNF**:
   - \( R \)은 BCNF가 아닙니다.
   
3. **분해**:
   - \( R_1 = (A, B) \), \( R_2 = (B, C) \)
   - \( R_1 \)과 \( R_2 \)는 BCNF에 속합니다.
   - 손실 없는 조인 분해(Lossless-join decomposition)
   - 의존성 보존(Dependency preserving)

이 예제는 \( R \)을 BCNF로 분해하면서 의존성을 보존하는 방법을 보여줍니다.

---

위의 절차는 특정 함수적 종속성 \( \alpha \rightarrow \beta \)가 분해된 릴레이션에서 보존되는지 확인하는 방법입니다. 이 절차를 예제로 설명하겠습니다.

### 예제 스키마 및 함수적 종속성
- 전체 릴레이션 \( R = (A, B, C) \)
- 함수적 종속성 집합 \( F = \{ A \rightarrow B, B \rightarrow C \} \)

### 분해
릴레이션 \( R \)을 두 개의 릴레이션으로 분해한다고 가정합시다:
- \( R_1 = (A, B) \)
- \( R_2 = (B, C) \)

### 테스트할 함수적 종속성
- \( \alpha \rightarrow \beta \) = \( A \rightarrow C \)

### 절차
1. **초기화**:
   - \( result = \alpha \)
   - 즉, \( result = A \)

2. **반복**:
   - 각 \( R_i \)에 대해 \( t \)를 계산하고 \( result \)를 갱신합니다.
   
3. **반복 과정**:
   - \( result = A \)
   
   - **첫 번째 반복**:
     - \( R_1 = (A, B) \)
       - \( result \cap R_1 = A \)
       - \( (result \cap R_1)^+ = A^+ = \{ A, B \} \) (A가 B를 결정하므로)
       - \( t = (result \cap R_1)^+ \cap R_1 = \{ A, B \} \cap \{ A, B \} = \{ A, B \} \)
       - \( result = result \cup t = \{ A \} \cup \{ A, B \} = \{ A, B \} \)
     
     - \( R_2 = (B, C) \)
       - \( result \cap R_2 = B \)
       - \( (result \cap R_2)^+ = B^+ = \{ B, C \} \) (B가 C를 결정하므로)
       - \( t = (result \cap R_2)^+ \cap R_2 = \{ B, C \} \cap \{ B, C \} = \{ B, C \} \)
       - \( result = result \cup t = \{ A, B \} \cup \{ B, C \} = \{ A, B, C \} \)
   
   - **두 번째 반복**:
     - \( R_1 = (A, B) \)
       - \( result \cap R_1 = \{ A, B \} \)
       - \( (result \cap R_1)^+ = \{ A, B \}^+ = \{ A, B \} \)
       - \( t = (result \cap R_1)^+ \cap R_1 = \{ A, B \} \cap \{ A, B \} = \{ A, B \} \)
       - \( result = result \cup t = \{ A, B, C \} \cup \{ A, B \} = \{ A, B, C \} \)
     
     - \( R_2 = (B, C) \)
       - \( result \cap R_2 = \{ B, C \} \)
       - \( (result \cap R_2)^+ = \{ B, C \}^+ = \{ B, C \} \)
       - \( t = (result \cap R_2)^+ \cap R_2 = \{ B, C \} \cap \{ B, C \} = \{ B, C \} \)
       - \( result = result \cup t = \{ A, B, C \} \cup \{ B, C \} = \{ A, B, C \} \)

4. **종료 조건**:
   - \( result \)가 더 이상 변경되지 않을 때까지 반복합니다.
   - 현재 \( result = \{ A, B, C \} \)이며, 더 이상 변경되지 않습니다.

### 결과 확인
- 최종 \( result \)가 \( \beta \)의 모든 속성(\{C\})을 포함하는지 확인합니다.
- \( result = \{ A, B, C \} \)는 \( \beta = \{ C \} \)를 포함하므로, \( A \rightarrow C \)는 분해된 릴레이션에서 보존됩니다.

### 요약
- \( result = \alpha \)에서 시작하여, 각 릴레이션 \( R_i \)에 대해 \( result \)에 새로운 속성을 추가합니다.
- 반복 과정에서 \( result \)가 더 이상 변경되지 않을 때까지 계속합니다.
- 최종 \( result \)가 \( \beta \)의 모든 속성을 포함하면, 함수적 종속성 \( \alpha \rightarrow \beta \)가 보존됩니다.

이 과정을 통해 함수적 종속성이 분해된 릴레이션에서 보존되는지 확인할 수 있습니다.


---
# 문제풀이

슬라이드와 7.1 번 문제의 해석을 비교하여 설명해드리겠습니다.

### 슬라이드 내용 요약:
1. **Lossless Decomposition 정의**:
   - 관계 스키마 \( R \)를 \( R_1 \)과 \( R_2 \)로 분해할 때, 정보 손실이 없는 분해를 lossless decomposition이라고 합니다.
   - 공식적으로, \( \Pi_{R_1}(r) \bowtie \Pi_{R_2}(r) = r \)일 때 lossless decomposition입니다. (완전히 복구 가능)
   - 반대로, \( r \subseteq \Pi_{R_1}(r) \bowtie \Pi_{R_2}(r) \neq r \)일 때는 lossy decomposition입니다.

### 7.1 번 문제 요약:
1. **문제**:
   - 스키마 \( R = (A, B, C, D, E) \)를 \( (A, B, C) \)와 \( (A, D, E) \)로 분해합니다.
   - 주어진 함수 종속성 집합 \( F \)가 있을 때, 이 분해가 lossless decomposition임을 증명합니다.
     - \( A \rightarrow BC \)
     - \( CD \rightarrow E \)
     - \( B \rightarrow D \)
     - \( E \rightarrow A \)

2. **해답**:
   - 분해 \( \{R_1, R_2\} \)가 lossless decomposition이 되려면 \( R_1 \cap R_2 \rightarrow R_1 \) 또는 \( R_1 \cap R_2 \rightarrow R_2 \)가 성립해야 합니다.
   - 여기서 \( R_1 = (A, B, C) \), \( R_2 = (A, D, E) \)이고, \( R_1 \cap R_2 = A \)입니다.
   - \( A \)가 후보 키이므로 \( R_1 \cap R_2 \rightarrow R_1 \)가 성립합니다.

### 비교 및 설명:
- **슬라이드**에서는 lossless decomposition의 정의와 공식적인 조건을 설명합니다.
  - \( \Pi_{R_1}(r) \bowtie \Pi_{R_2}(r) = r \)일 때 lossless decomposition입니다.
  - \( r \subseteq \Pi_{R_1}(r) \bowtie \Pi_{R_2}(r) \neq r \)일 때는 lossy decomposition입니다.

- **7.1 번 문제**에서는 주어진 스키마와 함수 종속성을 이용하여 lossless decomposition임을 증명합니다.
  - \( R_1 \cap R_2 \rightarrow R_1 \) 또는 \( R_1 \cap R_2 \rightarrow R_2 \)가 성립하면 lossless decomposition입니다.
  - 문제에서 \( R_1 \cap R_2 = A \)이고, \( A \)가 후보 키이므로 \( R_1 \cap R_2 \rightarrow R_1 \)가 성립하여 lossless decomposition임을 증명합니다.

따라서, 슬라이드의 정의와 공식적인 조건을 7.1 번 문제의 해답에서 사용하여 lossless decomposition임을 증명한 것입니다.

---

좋은 질문입니다. Lossless decomposition의 조건에 대해 더 명확히 이해하기 위해 예시를 들어 설명해드리겠습니다.

### Lossless Decomposition 조건 설명:
관계 스키마 \( R \)를 두 개의 하위 스키마 \( R_1 \)과 \( R_2 \)로 분해할 때, 정보 손실이 없는 분해를 lossless decomposition이라고 합니다. 이 분해가 lossless decomposition이 되려면 다음 조건 중 하나를 만족해야 합니다:
- \( R_1 \cap R_2 \rightarrow R_1 \)
- \( R_1 \cap R_2 \rightarrow R_2 \)

이 조건은 \( R_1 \cap R_2 \)가 \( R_1 \) 또는 \( R_2 \)를 결정할 수 있어야 한다는 의미입니다. 이는 분해된 두 관계를 다시 조인했을 때 원래의 관계를 복구할 수 있음을 보장합니다.

### 예시:
#### 예시 1: Lossless Decomposition
- **관계 스키마 \( R \)**: \( (A, B, C) \)
- **함수 종속성 \( F \)**: \( A \rightarrow B \)
- **분해**: \( R_1 = (A, B) \)와 \( R_2 = (A, C) \)

여기서 \( R_1 \cap R_2 = \{A\} \)입니다. 함수 종속성 \( A \rightarrow B \)가 있으므로 \( R_1 \cap R_2 \rightarrow R_1 \)가 성립합니다. 따라서 이 분해는 lossless decomposition입니다.

#### 예시 2: Lossy Decomposition
- **관계 스키마 \( R \)**: \( (A, B, C) \)
- **함수 종속성 \( F \)**: \( A \rightarrow B \)
- **분해**: \( R_1 = (A, B) \)와 \( R_2 = (B, C) \)

여기서 \( R_1 \cap R_2 = \{B\} \)입니다. 그러나 \( B \)는 \( R_1 \)이나 \( R_2 \)를 결정할 수 없습니다. 따라서 \( R_1 \cap R_2 \rightarrow R_1 \) 또는 \( R_1 \cap R_2 \rightarrow R_2 \)가 성립하지 않으므로 이 분해는 lossy decomposition입니다.

---

좋은 질문입니다. 주어진 함수 종속성 집합에서 \( A \)가 후보 키임을 증명하는 과정을 보여드리겠습니다.

### 주어진 함수 종속성:
1. \( A \rightarrow BC \)
2. \( CD \rightarrow E \)
3. \( B \rightarrow D \)
4. \( E \rightarrow A \)

### 후보 키의 정의:
후보 키는 관계의 모든 속성을 유일하게 식별할 수 있는 최소 속성 집합입니다.

### 증명 과정:
1. **함수 종속성 분석**:
   - \( A \rightarrow BC \): \( A \)가 \( B \)와 \( C \)를 결정합니다.
   - \( CD \rightarrow E \): \( C \)와 \( D \)가 \( E \)를 결정합니다.
   - \( B \rightarrow D \): \( B \)가 \( D \)를 결정합니다.
   - \( E \rightarrow A \): \( E \)가 \( A \)를 결정합니다.

2. **속성 폐포(Closure) 계산**:
   후보 키를 찾기 위해 속성 폐포를 계산합니다. 속성 폐포는 주어진 속성 집합이 결정할 수 있는 모든 속성의 집합입니다.

   - \( A^+ \) (A의 폐포):
     - \( A \rightarrow BC \)로부터 \( A \)가 \( B \)와 \( C \)를 결정합니다.
     - \( B \rightarrow D \)로부터 \( A \)가 \( D \)를 결정합니다 (왜냐하면 \( A \)가 \( B \)를 결정하므로).
     - \( CD \rightarrow E \)로부터 \( A \)가 \( E \)를 결정합니다 (왜냐하면 \( A \)가 \( C \)와 \( D \)를 결정하므로).
     - \( E \rightarrow A \)는 이미 \( A \)가 포함되어 있으므로 추가 정보는 없습니다.
     
     따라서, \( A^+ = \{A, B, C, D, E\} \)입니다.

3. **후보 키 확인**:
   - \( A \)의 폐포 \( A^+ \)가 관계 \( R \)의 모든 속성을 포함하고 있습니다. 따라서 \( A \)는 관계 \( R \)의 후보 키입니다.

### 요약:
- 주어진 함수 종속성에서 \( A \)의 폐포를 계산해 보니 \( A \)가 관계의 모든 속성을 결정할 수 있음을 확인했습니다.
- 따라서 \( A \)는 관계 \( R \)의 후보 키입니다.

이 과정을 통해 \( A \)가 후보 키임을 알 수 있습니다.

---

7.5.
네, 저 공식은 데이터베이스 이론에서 사용되는 암스트롱의 공리(Armstrong's axioms) 중 하나인 의사전이 규칙(pseudotransitivity rule)을 증명하는 과정입니다. 암스트롱의 공리는 함수 종속성(functional dependency)을 다루는 규칙들로, 데이터베이스 설계에서 중요한 역할을 합니다.

의사전이 규칙은 다음과 같이 정의됩니다:
- 만약 \( \alpha \rightarrow \beta \)이고 \( \gamma \beta \rightarrow \delta \)이면, \( \alpha \gamma \rightarrow \delta \)이다.

이 규칙을 증명하기 위해 암스트롱의 공리를 사용합니다. 암스트롱의 공리는 다음과 같습니다:
1. **반사성 규칙(Reflexivity Rule)**: \( \alpha \subseteq \beta \)이면, \( \alpha \rightarrow \beta \)
2. **증강 규칙(Augmentation Rule)**: \( \alpha \rightarrow \beta \)이면, \( \alpha \gamma \rightarrow \beta \gamma \)
3. **이행 규칙(Transitivity Rule)**: \( \alpha \rightarrow \beta \)이고 \( \beta \rightarrow \delta \)이면, \( \alpha \rightarrow \delta \)

주어진 증명은 다음과 같습니다:
1. \( \alpha \rightarrow \beta \) (주어진 조건)
2. \( \alpha \gamma \rightarrow \gamma \beta \) (증강 규칙과 집합의 교환 법칙 사용)
3. \( \gamma \beta \rightarrow \delta \) (주어진 조건)
4. \( \alpha \gamma \rightarrow \delta \) (이행 규칙 사용)

이 과정을 통해 의사전이 규칙이 암스트롱의 공리로부터 유도될 수 있음을 증명합니다.

---
7.8
알겠습니다. 주어진 문제를 더 명확하게 이해하기 위해, \( \alpha^+ \)를 계산하는 알고리즘을 예시를 통해 설명해드리겠습니다.

### \( \alpha^+ \)란?
- \( \alpha^+ \)는 주어진 속성 집합 \( \alpha \)로부터 도출할 수 있는 모든 속성의 집합을 의미합니다.
- 예를 들어, \( \alpha = \{A\} \)이고 함수적 종속성(FD)이 \( A \rightarrow B \), \( B \rightarrow C \)라면, \( \alpha^+ = \{A, B, C\} \)가 됩니다.

### 알고리즘 설명
알고리즘은 \( \alpha \)의 속성들로부터 도출할 수 있는 모든 속성들을 찾아 \( \alpha^+ \)를 계산합니다.

#### 1. 초기화
1. **result**를 빈 집합으로 초기화합니다.
2. 각 함수적 종속성(FD)에 대해 **fdcount** 배열을 초기화하여 각 FD의 왼쪽에 있는 속성 수를 저장합니다.
3. 각 속성에 대해 **appears** 배열을 초기화하여 해당 속성이 왼쪽에 있는 FD의 인덱스를 저장합니다.

#### 2. addin 절차
1. \( \alpha \)의 각 속성 \( A \)에 대해 \( A \)가 result에 없으면 result에 추가합니다.
2. appears[\( A \)]의 각 요소 \( i \)에 대해 fdcount[\( i \)]를 감소시키고, fdcount[\( i \)]가 0이 되면 해당 FD의 오른쪽 부분을 addin 절차로 호출합니다.

### 예시
다음의 함수적 종속성 집합을 고려해봅시다:
- \( A \rightarrow B \)
- \( B \rightarrow C \)
- \( C \rightarrow D \)

그리고 \( \alpha = \{A\} \)라 가정합시다.

#### Step 1: 초기화
- 초기 result: \( \emptyset \)
- fdcount: \([1, 1, 1]\) (각 FD의 왼쪽에 하나의 속성)
- appears:
  - appears[\( A \)] = [0] (첫 번째 FD의 인덱스)
  - appears[\( B \)] = [1] (두 번째 FD의 인덱스)
  - appears[\( C \)] = [2] (세 번째 FD의 인덱스)

#### Step 2: addin 절차
1. \( \alpha = \{A\} \) 이므로 \( A \)를 result에 추가합니다.
   - result = \{A\}
2. appears[\( A \)]에서 첫 번째 FD를 확인합니다: \( A \rightarrow B \)
   - fdcount[0] = 0 ( \( A \)가 result에 추가되었으므로 감소)
   - fdcount[0]이 0이므로, \( B \)를 addin 절차로 호출합니다.
     - result = \{A, B\}
3. appears[\( B \)]에서 두 번째 FD를 확인합니다: \( B \rightarrow C \)
   - fdcount[1] = 0 ( \( B \)가 result에 추가되었으므로 감소)
   - fdcount[1]이 0이므로, \( C \)를 addin 절차로 호출합니다.
     - result = \{A, B, C\}
4. appears[\( C \)]에서 세 번째 FD를 확인합니다: \( C \rightarrow D \)
   - fdcount[2] = 0 ( \( C \)가 result에 추가되었으므로 감소)
   - fdcount[2]이 0이므로, \( D \)를 addin 절차로 호출합니다.
     - result = \{A, B, C, D\}

#### 최종 결과
- result = \{A, B, C, D\}
- 따라서 \( \alpha^+ = \{A, B, C, D\} \)입니다.

이 예시를 통해 알고리즘이 \( \alpha \)의 닫힘 \( \alpha^+ \)를 올바르게 계산하는 과정을 설명했습니다.

---

7.9
종속관계를 잘못 이해하고 있었다. 
user -> product를 결정하는 게 아님.
N:N 관계일뿐만 아니라 (내 소유 아니어도 가능)
설사 1:N이라 해도
여전히 아래 쿼리를 하면 여러개가 나와서
user -> product 종속관계가 성립하지 않음.
```sql 
select user_id
from user_products
group by user_id
having count(distinct product_id) > 1;
```

그게 성립했다면
user 1이면 product 2,3,4,5...가 아니라
단 하나의 product로 결정되었어야하는데.

그래서 functional dependency는
대부분 프라이머리 키나 유니크키(아마)에서 동작한다.

이를테면 products에서 id -> mallPid라던가.


---

a번 질문은 BCNF 분해 알고리즘을 사용하여 주어진 관계 스키마 \( r(\alpha, \beta, \gamma) \)를 \( r_1(\alpha, \beta) \)와 \( r_2(\alpha, \gamma) \)로 분해할 때, 어떤 기본 키와 외래 키 제약 조건이 분해된 관계에 적용될지를 묻고 있습니다.

답변에서 말하는 바는 다음과 같습니다:

- \( \alpha \)는 \( r_1 \)의 기본 키가 되어야 합니다. 이는 \( \alpha \rightarrow \beta \)라는 함수 종속성에 의해 \( \alpha \)가 \( r_1 \)에서 유일하게 식별할 수 있는 속성임을 의미합니다.
- \( \alpha \)는 \( r_2 \)에서 외래 키가 되어야 합니다. 이는 \( r_2 \)가 \( r_1 \)을 참조해야 함을 의미합니다. 즉, \( r_2 \)의 \( \alpha \) 값은 \( r_1 \)의 \( \alpha \) 값과 일치해야 합니다.

외래 키 제약 조건이 필요한 이유는 데이터의 무결성을 유지하기 위해서입니다. 예를 들어, \( r_1 \)에서 \( \alpha \) 값이 삭제되면, \( r_2 \)에서도 해당 \( \alpha \) 값을 참조하는 튜플이 존재하지 않도록 해야 합니다. 이를 통해 데이터의 일관성을 유지할 수 있습니다.



---

7.12
이 문제는 관계형 데이터베이스에서 스키마의 분해와 자연 조인에 관한 것입니다. 주어진 관계 \( u(U) \)와 스키마 \( U \)가 있을 때, 스키마 \( U \)를 \( R_1, R_2, \ldots, R_n \)으로 분해하고, \( r_i = \Pi_{R_i}(u) \)라고 정의합니다. 여기서 \( \Pi_{R_i} \)는 \( u \)의 \( R_i \) 속성들에 대한 투영(projection)을 의미합니다. 문제는 \( u \subseteq r_1 \bowtie r_2 \bowtie \cdots \bowtie r_n \)임을 증명하는 것입니다.

### 증명 과정:

1. **임의의 튜플 \( t \) 선택**:
   - \( t \)가 \( u \)의 임의의 튜플이라고 가정합니다.

2. **투영의 의미**:
   - \( r_i = \Pi_{R_i}(u) \)는 \( t[R_i] \in r_i \)를 의미합니다. 여기서 \( t[R_i] \)는 \( t \)의 \( R_i \) 속성들에 대한 부분 튜플입니다.
   - 따라서, \( t[R_1] \in r_1, t[R_2] \in r_2, \ldots, t[R_n] \in r_n \)입니다.

3. **자연 조인의 정의**:
   - 자연 조인의 정의에 따르면, \( t[R_1] \bowtie t[R_2] \bowtie \cdots \bowtie t[R_n] \in r_1 \bowtie r_2 \bowtie \cdots \bowtie r_n \)입니다.

4. **자연 조인의 결과**:
   - 자연 조인의 결과는 다음과 같이 표현됩니다:
     \[
     t[R_1] \bowtie t[R_2] \bowtie \cdots \bowtie t[R_n] = \Pi_{\alpha} (\sigma_{\beta} (t[R_1] \times t[R_2] \times \cdots \times t[R_n]))
     \]
     여기서 조건 \( \beta \)는 동일한 이름을 가진 속성들의 값이 동일해야 함을 의미하고, \( \alpha = U \)는 모든 속성들을 포함하는 투영을 의미합니다.

5. **카테시안 곱과 선택**:
   - 단일 튜플의 카테시안 곱은 하나의 튜플을 생성합니다.
   - 선택 과정은 동일한 이름을 가진 속성들이 동일한 값을 가져야 한다는 조건을 만족시킵니다. 이는 모든 속성들이 동일한 튜플에서 투영된 것이기 때문에 자동으로 만족됩니다.

6. **투영 절차**:
   - 마지막으로 투영 절차는 중복된 속성 이름을 제거합니다.

7. **분해의 정의**:
   - 분해의 정의에 따르면, \( U = R_1 \cup R_2 \cup \cdots \cup R_n \)입니다. 이는 \( t \)의 모든 속성들이 \( t[R_1] \bowtie t[R_2] \bowtie \cdots \bowtie t[R_n] \)에 포함된다는 것을 의미합니다. 즉, \( t \)는 이 조인의 결과와 동일합니다.

8. **임의의 튜플 \( t \)**:
   - \( t \)가 \( u \)의 임의의 튜플이므로, \( u \subseteq r_1 \bowtie r_2 \bowtie \cdots \bowtie r_n \)입니다.

따라서, 주어진 관계 \( u \)는 \( r_1 \bowtie r_2 \bowtie \cdots \bowtie r_n \)의 부분집합임을 증명할 수 있습니다.


---

점화식과 특정 방정식은 수열의 항들 사이의 관계를 나타내는 데 사용되는 중요한 개념입니다. 

**점화식 (Recurrence relation)**

* 수열의 한 항을 이전 항들과의 관계식으로 나타낸 것을 말합니다. 
* 예를 들어, 수열 $\{a_n\}$에서 $a_n = a_{n-1} + 2$ (단, $a_1 = 1$)은 현재 항 $a_n$이 이전 항 $a_{n-1}$보다 2 큰 수열을 정의하는 점화식입니다.

**특정 방정식 (Characteristic equation)**

* 특정 형태의 점화식, 즉 **선형 동차 점화식**을 푸는 데 사용되는 도구입니다. 
* **선형 동차 점화식**은 각 항이 이전 항들의 선형 결합으로 표현되는 점화식을 의미합니다. 
    * 예: $a_n = 3a_{n-1} - 2a_{n-2}$
* 특정 방정식은 점화식의 각 항을 $a_n = r^n$ 형태로 가정하여 얻어집니다. 

**특정 방정식을 이용한 점화식 풀이 예시**

점화식 $a_n = 3a_{n-1} - 2a_{n-2}$ (단, $a_1 = 1$, $a_2 = 3$)을 풀어봅시다.

1. **특정 방정식 만들기**: $a_n = r^n$을 대입하면 $r^n = 3r^{n-1} - 2r^{n-2}$을 얻습니다. 양변을 $r^{n-2}$로 나누면 특정 방정식 $r^2 - 3r + 2 = 0$을 얻습니다.

2. **특정 방정식 풀기**: 특정 방정식을 인수분해하면 $(r-1)(r-2) = 0$이고, 따라서 $r=1$ 또는 $r=2$입니다.

3. **일반항 찾기**: 특정 방정식의 해가 서로 다른 두 실근 $r_1$, $r_2$일 경우, 점화식의 일반항은 $a_n = c_1 r_1^n + c_2 r_2^n$ 형태로 나타납니다.  따라서 주어진 점화식의 일반항은 $a_n = c_1  + c_2 2^n$ 형태입니다.

4. **초기 조건 대입**: 초기 조건 $a_1 = 1$, $a_2 = 3$을 이용하여 $c_1$, $c_2$ 값을 구합니다. $a_1 = 1 = c_1 + 2c_2$이고 $a_2 = 3 = c_1 + 4c_2$이므로, 연립 방정식을 풀면 $c_1 = -1$, $c_2 = 1$을 얻습니다.

5. **최종 해**: 따라서 주어진 점화식의 해는 $a_n = -1 + 2^n$입니다.

**결론**

점화식은 수열의 항들 사이의 관계를 나타내는 유용한 도구이며, 특정 방정식은 특정 형태의 점화식을 푸는 데 사용되는 강력한 도구입니다. 

더 궁금한 점이 있다면 알려주세요.

---

# Chap 13.
하나의 Page에서 일어나는 일.
사실상 대부분 Slotted Page Structure.
고정길이가 잘 없음. varchar 등

### Records in Files
heap, log file structure, index file organization 순으로 중요.

heap 을 주로 쓰는게 Postgres,
B+ index file은 MySQL.

*
트랜잭션을 할 때 
연속적 파일은 가장 최신의 row가 바쁘고 그걸 memory에 올려서 빠르게 처리하면 되는데

힙파일 구조의 경우 여러 페이지들이 바빠질거고 그걸 전부 메모리로 들고올 수는 없을 수 있으므로 느려질 수 있어보임.
-> 분리하는게 유리할 수 있음. (결론)

힙파일 구조는 아무것도 할필요가 없어서? 훨씬 빠를것 같지만(삽입/삭제시 헤더만 고쳐주면 끝) insert 성능은 mysql이랑 비슷함.
한번에 메모리에서 다 할수있는게 아니라서..

*
Storage Access
- transfer 비용이 가장 크므로 이것을 최소화하는게 가장 우선.
- 메인 메모리에 최대한 많은 block들을 keeping해서 계산해야, disk access 최소화.
-> 캐시 히트와도 관련. 

OS가 아니라 버퍼 매니저가 Buffer를 관리
데이터베이스는 스스로 OS로서 메모리를 관리하고 싶어하는데.

버퍼는 파일 캐시같은 존재.
메인메모리.

버퍼 관리자는 OS의 virtual memory와 비슷한 역할. 


DBMS 입장에서는 한 프로세스가 읽을 동안 다른 프고세스가 접근하는걸 원치 않음.
내가 읽고있는데 변형이 일어나면 안돼서.

OS에서는 서로다른 virtual memory를 가지고 같은 파일을 접근할 수 있을텐데
버퍼매니저는 don't want.
-> 그걸 매니지하는게 pined lock 등.

(목표는)
여러 프로세스들이 그 메모리를 공유자원으로 사용하게 하기 위해 shared - 를 사용.
DiskIO는 버퍼 매니저를 사용할 때 단 한 번.



--
분산 환경이면 한 테이블도 
여러 파일로 구성될 수 있고,
파일은 여러 페이지로 구성되고,
그 페이지 안에서 슬롯들로 구성. 


// 여기까지 database storage 1 내용.




힙파일 단점 - 
Update an existing tuple using its record id

그걸 개선하고자 하는게
Log-structure
대표적으로 rocksDB,levelsDB.
- 키밸류 스토리지.
어쨌든 오래된 걸 없애주는.
levelDB는 같은 레벨끼리 통폐합.

--

MySQL에서도 slotted storage 씀.

둘다 증가해야하는 구조면 반대방향으로 늘어나게 하는 구조.















































