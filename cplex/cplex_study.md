# Tutorial: Linear Programming, (Cplex Part 1)

## Introduction to Linear Programming

LP의 정의: 선형으로된 Objective function의 maximize나 minimize를 목표로 formulation, 이때 제약조건도 linear 해야됨 <br><br>

LP에서 허용되지 않는 것

- multiplication of two or more variables (such as x times y),
- quadratic and higher order terms (such as x squared or x cubed),
- exponents,
- logarithms,
- absolute values.

## Example: a production problem

#### Question

전화기 회사에서 1. 탁상전화기 2. 휴대폰, 총 두 종류의 전화기를 생산하고 판매한다. 각 유형의 전화기는 해당 전화기 회사에서 조립하고 도색한다. 목표는 이익을 극대화하는 것이다.
<br>회사의 생산 능력에는 한계가 있으며, 회사는 공장의 생산 능력을 초과하지 않는 범위 내에서 생산할 휴대폰 종류별 최적 수량을 계산해야 한다.<br><br>
문제에서 주어진 제약사항
<br> 1. The DeskProduction should be greater than or equal to 100. (탁상전화기 생산량 100 이상이여야만 함)
<br> 2. The CellProduction should be greater than or equal to 100. (휴대폰 생산량 100 이상이여야만 함)
<br> 3. The assembly time for DeskProduction plus the assembly time for CellProduction should not exceed 400 hours. (탁상전화기 조립시간 + 휴대폰 조립시간은 400시간 초과하면 안됨)
<br> 4. The painting time for DeskProduction plus the painting time for CellProduction should not exceed 490 hours. (탁상전화기 페인팅시간 + 휴대폰 페인팅시간은 490시간 초과하면 안됨) <br><br>
**\* 탁상전화기 판매시 단위당 이익 12, 휴대폰 판매시 단위당 이익 20
<br> ** 0.2: 탁상 전화기 1개 조립하는데 필요한 시간
<br> ** 0.4: 휴대폰 1개 조립하는데 필요한 시간
<br> ** 0.5: 탁상전화기 1개 도색하는데 필요한 시간
<br> \*\* 0.4: 휴대폰 1개 도색하는데 필요한 시간

#### Solution

여기에서 결정변수를 먼저 정의해보면, <br>
탁상전화기 생산량 (이하 DeskProduction) 과 휴대폰 생산량 (이하 cellProduction) 이고, 문제에서 주어진 사항에 의거하여 LP model을 수리적으로 formulation 해보면

<br>수리적 LP formulation 예시

```
maximize
        z = 12 desk_production + 20 cell_production

subject to:
desk_production>=100
cell_production>=100
0.2 desk_production + 0.4 cell_production <= 400
0.5 desk_production + 0.4 cell_production <= 490
```

#### Soltuion_with_Cplex

\*\* Let's use the DOcplex Python library to write the mathematical model in Python.

<step 1: Download the library.>

```python
import os

try:
    import docplex.mp
except ImportError:
    if hasattr(sys, 'real_prefix'):
        os.system('pip install docplex')
    else:
        os.system('pip install --user docplex')
```

해당 코드는 docplex의 설치여부를 확인하고, 설치가 되어있지 않는 경우에 한하여 설치를 진행하는 코드이다.

<step 2: Set up the prescripive model>
<br><br>
Create the model

```python
# 먼저 Model 클래스를 docplex.mp에서 가져옵니다.

# 이 줄은 docplex.mp.model 모듈에서 Model 클래스를 import
# Model 클래스는 최적화 모델을 생성하고 관리하기 위한 주요 인터페이스를 제공한다. 모델에는 변수, 제약 조건, 목적 함수 등을 정의할 수 있으며, 이를 통해 최적화 문제를 구성한다.

from docplex.mp.model import Model

# 이름이 있는 하나의 모델 인스턴스를 만듭니다.

# Model 클래스의 인스턴스를 생성하고 이를 변수 m에 할당한다. 생성자에 전달된 name 매개변수는 모델의 이름을 지정해준다. 여기서 모델의 이름은 'telephone_production'으로 설정되었다.

m = Model(name='telephone_production')
```

\*\*\* python의 클래스 개념 잠깐 짚고 넘어가기 \*\*\*

<br>
Define the decision variable

```python
# 기본적으로 Docplex의 모든 변수는 하한이 0이고 상한이 무한합니다.
# 이 문제는 기본적으로 LP로 formulation 됐기 때문에 의사결정 변수가 연속형 변수로 선언돼야한다.

desk = m.continuous_var(name='desk') # 탁상형 전화기 생산량 의사결정변수 생성
cell = m.continuous_var(name='cell') # 휴대폰 생산량 의사결정변수 생성
```

<br>
Set up the constraints

```python
# write constraints
# constraint #1: desk production is greater than 100
m.add_constraint(desk >= 100)

# constraint #2: cell production is greater than 100
m.add_constraint(cell >= 100)

# constraint #3: assembly time limit
ct_assembly = m.add_constraint( 0.2 * desk + 0.4 * cell <= 400)

# constraint #4: paiting time limit
ct_painting = m.add_constraint( 0.5 * desk + 0.4 * cell <= 490)
```

<br>
Express the objective

```python
m.maximize(12 * desk + 20 * cell)
```

<br>Print information about the model (최종 결과 print)

```python
m.print_information()
```

실행 결과 <br><br>
Model: telephone_production

- number of variables: 2
  - binary=0, integer=0, continuous=2
- number of constraints: 4
  - linear=4
- parameters: defaults
- objective: maximize
- problem type is: LP

### 전체 코드

<br>

```python
import os

try:
    import docplex.mp
except ImportError:
    if hasattr(sys, 'real_prefix'):
        os.system('pip install docplex')
    else:
        os.system('pip install --user docplex')

# first import the Model class from docplex.mp
from docplex.mp.model import Model

# create one model instance, with a name
m = Model(name='telephone_production')

# by default, all variables in Docplex have a lower bound of 0 and infinite upper bound
desk = m.continuous_var(name='desk')
cell = m.continuous_var(name='cell')

# write constraints
# constraint #1: desk production is greater than 100
m.add_constraint(desk >= 100)

# constraint #2: cell production is greater than 100
m.add_constraint(cell >= 100)

# constraint #3: assembly time limit
ct_assembly = m.add_constraint(0.2 * desk + 0.4 * cell <= 400)

# constraint #4: paiting time limit
ct_painting = m.add_constraint(0.5 * desk + 0.4 * cell <= 490)

m.maximize(12 * desk + 20 * cell)
m.print_information()
```
