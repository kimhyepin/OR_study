# Solving the Muliple knapsack problem with google OR tools

<div>* A step-by-step walkthrough of using linear programming in python.</div>

#### Question:

사람은 5명이고 각각 포장팩을 들고있다. 각 포장팩은 50 pounds 의 무게 제약과, 50 입방인치의 부피 제약이 있다. 각 가방은 안전성을 보장하기 위하여 최대 5 rads의 방사능에만 노출 될 수 있다. 각 item은 특정 레벨의 utility를 가지고 있고, 총 item의 갯수는 19개이다.

#### 최종목표: 5개의 가방에 안전성을 보장하면서 util (효용성) 을 극대화하는 항목 조합 선택

#### Solution:

OR tools의 linear-solver 부터 가져와주기 (솔버 생성 단계)

```python
import ortools
from ortools.linear_solver import pywraplp

solver = solver = pywraplp.Solver.CreateSolver('SCIP')
```

\*\* sets, parameter 정의 단계
<br> 1. sets
<br> number_of_item = N = {1, 2, 3 ..., 19}
<br> number_of_bags = M = {1, 2, 3, 4, 5}
<br>
<br> 2. 매개변수 (data, parameter)
<br> wi = the amount of weight in pounds (lb) for item i (item 무게 data)
<br> vi = the amount of volumne in cubic inches (in^3) for item i, (item 부피 data)
<br> ri = the amount of radiation (rads) for item i (item 방사능 data)
<br> ui = the amount of value (utils) for item i (item 효용성 data)
<br> cj = the weight capacity (lb) for bag j (bag 무게제한 data)
<br> dj = the volume capacity (in^3) for bag j (bag 부피제한 data)
<br> ej = the radiation capacity (rads) for bag j (bag 방사능제한 data)

```python
data = {}  # 빈 딕셔너리 생성
values = [48, 30, 42, 36, 22, 43, 18, 24, 36,
          29, 30, 25, 19, 41, 34, 32, 27, 24, 18]
weights = [10, 30, 12, 22, 12, 20, 9, 9, 18,
           20, 25, 18, 7, 16, 24, 21, 21, 32, 9]
volume = [15, 20, 18, 20, 5, 12, 7, 7, 24,
          30, 25, 20, 5, 25, 19, 24, 19, 14, 30]
rad = [3, 1, 2, 3, 1, 2, 0, 2, 2, 1, 2, 3, 4, 3, 2, 3, 1, 1, 3]
# 문제에서 주어진 각 "item들의 조건"을 배열로 만들어주기

assert len(values) == len(weights) == len(volume) == len(rad)
# python 에서 가정 설정문으로 이용되는 assert
# if문으로도 물론 제어될 수 있지만, 이와 같이 원하는 바를 보증하는 방식으로 코딩하는 방어적 프로그래밍의 방법도 있음을 참고

data['values'] = values
data['weights'] = weights
data['volume'] = volume
data['rad'] = rad
# data라는 빈 dictionary에 key주고, value 값으로써 위에 list로 생성한 값을 줌
# {'values': [48, 30, 42, 36, 22, 43, 18, 24, 36, 29, 30, 25, 19, 41, 34, 32, 27, 24, 18]} 이러한 구조등으로 이뤄지게 됨

data['items'] = list(range(len(weights)))
data['num_items'] = len(values)
# item의 총 갯수(19개)를 저장하며 item 관련 변수정의 종료

# 포장팩 관련 변수정의 시작부
# 모든 포장팩에 대한 동일 무게 제약 (50 pounds), 동일 부피 제약 (50 입방인치), 동일 방사능 제약 (5 rads)

number_bags = 5
data['bag_capacities'] = [50, 50, 50, 50, 50]  # pounds
data['bag_volume'] = [50, 50, 50, 50, 50]
data['rad_capacities'] = [5, 5, 5, 5, 5]
data['bags'] = list(range(number_bags))

# 이 문장은 위의 가정 설정문과 동일하게 입력값을 하나도 놓치지 않았는지 확인하기 위해 존재 (만약에 assert 뒤의 내용이 false면 AssertionError 예외를 발생시키고 코드 실행을 바로 종료한다.)

assert len(data['bag_capacities']) == number_bags
assert len(data['bag_capacities']) == len(
    data['bag_volume']) == len(data['rad_capacities'])

# print 문에서 사용된 * 연산자는 python의 unpacking operator로 사용됨
# packing 연산자는 **
# python packing, unpacking 개념 짚고넘어가기 (밑에 링크 참고)
# packing: 한 변수에 여러개의 데이터를 넣는 것, unpacking: 한 변수의 데이터를 각각의 변수로 반환

print("Values: ", *data['values'])
print('Weights:', *data['weights'])
print('Volume:', *data['volume'])
print('Radiation Levels:', *data['rad'])
print("Number of Items:", data['num_items'])
print("Number of Knapsacks:", number_bags)
print('Knapsack Capacities: 50 Pounds, 50 cubic inches, 5 Levels of Radiation')
```

###### \*\* 참고부분 (packing, unpacking 개념)

[Python 언패킹(Unpacking) 연산자에 대해 알아보기](https://velog.io/@yunhlim/Python-%EC%96%B8%ED%8C%A8%ED%82%B9Unpacking-%EC%97%B0%EC%82%B0%EC%9E%90 'Python 언패킹에 대한 설명')

```python
t = [1, 2, 3] # 1,2,3을 변수 t에 패킹
a, b, c = t # t에 있는 값을 변수 a,b,c에 언패킹
```

결정 변수의 정의
<br><br> 결정 변수: 해당항목이 선택되었으면 1, 아니면 0
<br> xij = 1 || 0
<br>(for item i in knapsack j, i = {1, 2, ..., 19}, j ={1, 2, 3, 4, 5})

<div>이것을 python 코드로써 표현하면</div><br>

```python
# 빈 dictionary 생성
x = {}

# items와 bags의 요소를 하나씩 뺀다.

# 현재 아이템 i와 현재 bags j의 조합으로 생성된, 튜플 (i, j)를 key로 이용하여 solver.IntVar (솔버 인스턴스) 함수를 호출한다. Intvar 함수의 첫번째 인자 0은 변수의 최소값, 두번째 인자 1은 최대값을 나타낸다. 따라서 이 변수는 0과 1의 값을 가질 수 있으며 이 조건을 통해 해당 결정변수가 이진변수임을 나타내준다.

# 또한 %i는 python의 정수 포맷 지정자이다. 따라서 i와 j의 값이 각각 %i의 위치에 대입된다.

for i in data['items']:
    for j in data['bags']:
        x[(i,j)] = solver.IntVar(0,1,'x_%i_%i' % (i, j))
```

<div>이제 본격적으로 제약사항에 대한 코딩이 필요하다. 다시 한번 정리해보면, 각 bag에 대하여 1> 무게, 2> 부피, 3> 방사선량에 제약사항이 존재하며 4> 각 item들은 5개의 가방 중 하나의 가방에만 들어있도록 제약을 줘야한다. 이러한 제약사항을 코드로 표현해보겠다.</div>
<br>

```python
# item이 1개의 bag에 배치되도록 하는 constraint
for i in data['items']:
    solver.Add(sum(x[i,j] for j in data['bags'])<=1)

# bag 수용 weight 관련 constraint
for j in data['bags']:
    solver.Add(sum(x[(i,j)]*data['weights'][i]
                  for i in data['items']) <= data['bag_capacities'][j])
# bag 수용 volume 관련 constraint
for j in data['bags']:
    solver.Add(sum(x[(i,j)]*data['volume'][i]
                  for i in data['items']) <= data['bag_volume'][j])

# bag 수용 rad 관련 constraint
for j in data['bags']:
    solver.Add(sum(x[(i,j)]*data['rad'][i]
                  for i in data['items']) <= data['rad_capacities'][j])
```

<div>다음으로는 목적함수에 대한 코드구현이 필요하다. 우리의 목적은 효용 (utility) 을 maximize 하는 것이다. </div>
<br>

```python
# objective function

# 최적화할 목적 함수를 생성
objective = solver.Objective()

# 모든 items와 모든 bags에 대하여 (내부반복 수행 통해 하나씩 꺼내옴)
for i in data['items']:
    for j in data['bags']:

# 이 코드 줄은 목적 함수에 대한 계수를 설정한다. 여기서 x[(i,j)]는 결정 변수를 나타내며, 각 변수는 특정 아이템 i가 특정 가방 j에 할당되는지 여부를 나타낸다. data['values'][i]는 아이템 i의 값(또는 중요도)을 나타내며, 이 값은 목적 함수의 해당 변수의 "계수"로 설정된다.
        objective.SetCoefficient(x[(i,j)], data['values'][i])

# 최종목적이 해당 목적함수식을 max 하는 것임을 나타내준다.
objective.SetMaximization()
```

<div> 해당 문제에 대한 최종 solve 코드이다.</div>
<br>

```python
solv = solver.Solve()  # solver의 Solve 메소드를 호출하여 최적화 문제를 해결하고 결과를 solv 변수에 저장
if solv == pywraplp.Solver.OPTIMAL:  # 만약 해결 결과가 '최적' 상태라면, 즉 최적해를 찾았다면, 다음 코드 블록을 실행

    print('Total Packed Value:', objective.Value())  # 최적화된 목적 함수의 값(총 포장된 가치)을 출력
    total_weight = 0  # 총 무게를 저장할 변수를 0으로 초기화
    for j in data['bags']:  # data 사전의 'bags' 키에 해당하는 각 가방에 대해 반복
        bag_value = 0  # 해당 가방의 총 가치를 저장할 변수를 0으로 초기화
        bag_weight = 0  # 해당 가방의 총 무게를 저장할 변수를 0으로 초기화
        bag_volume= 0  # 해당 가방의 총 부피를 저장할 변수를 0으로 초기화
        bag_rad = 0  # 해당 가방의 총 방사능을 저장할 변수를 0으로 초기화
        print('\n','Bag', j+1 , '\n')  # 현재 가방의 번호를 출력, 가방 번호는 1부터 시작
        for i in data['items']:  # data 사전의 'items' 키에 해당하는 각 아이템에 대해 반복
            if x[i,j].solution_value()>0:  # 만약 아이템 i가 가방 j에 포함되었다면 (해당 변수의 해결 값이 0보다 크다면),
                # 해당 아이템의 정보(값, 무게, 부피, 방사능)를 출력
                print('Item:', i ,
                      'value',data['values'][i],
                      'Weight', data['weights'][i],
                      'Volume',data['volume'][i],
                      'Radiation', data['rad'][i]
                     )
                bag_value += data['values'][i]  # 가방의 총 가치에 아이템의 가치를 더해줌
                bag_weight += data['weights'][i]  # 가방의 총 무게에 아이템의 무게를 더해줌
                bag_volume += data['volume'][i]  # 가방의 총 부피에 아이템의 부피를 더해줌
                bag_rad += data['rad'][i]  # 가방의 총 방사능에 아이템의 방사능을 더해줌

        # 각 가방에 포함된 아이템들의 총 가치, 무게, 부피, 방사능을 출력
        print('Packed Knapsack Value: ', bag_value)
        print('Packed Knapsack Weight: ', bag_weight)
        print('Packed Knapsack Volume: ', bag_volume)
        print('Pack Knapsack Radiation: ', bag_rad)

else:  # 만약 최적의 해결책을 찾지 못했다면,
    print("There is no optimal solution")  # 최적해가 없음을 notice

```
