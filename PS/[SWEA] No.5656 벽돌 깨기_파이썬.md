### # 문제 출처
https://swexpertacademy.com/main/code/problem/problemDetail.do?contestProbId=AWXRQm6qfL0DFAUo&categoryId=AWXRQm6qfL0DFAUo&categoryType=CODE

### # 풀이1
```python
import copy
from collections import deque
from itertools import product
# 상 하 좌 우
dx = [0, 0, -1, 1]
dy = [-1, 1, 0, 0]

T = int(input())
# 여러개의 테스트 케이스가 주어지므로, 각각을 처리합니다.
for test_case in range(1, T + 1):
    n,w,h = map(int,input().split())
    space, per, answer  = [], [], []
    for i in range(h):
        space.append(list(map(int,input().split())))
    l1 = [i for i in range(w)] # per을 만들기 위한 리스트 l1

    for i in product(l1, repeat=n): # 모든 경우를 구하는 product (0,0,0) ~ (9,9,9)
        per.append(i)

    for p in per:
        c_space = copy.deepcopy(space) # 새로운 c_space를 만듬.
        count = 0 # 터지는 폭탄 count
        for t in p:
            Q = deque() # Q는 [폭탄의 범위, y값, x값]
            for j in range(h):
                if c_space[j][t] != 0: # 처음 폭탄을 탐색함.
                    count = count + 1
                    power = c_space[j][t] # power는 폭탄의 범위
                    Q.append([power,j,t])
                    c_space[j][t] = 0

                    while (Q):
                        bomb_power,y,x = Q.popleft()

                        for b_p in range(1,bomb_power): # 폭탄의 범위만큼 퍼지는 것을 고려해서 ny, nx를 계산

                            for i in range(4):
                                ny = y + (dy[i] * b_p)
                                nx = x + (dx[i] * b_p)

                                if 0 <= ny < h and 0 <= nx < w and c_space[ny][nx] != 0 :
                                    Q.append( [ c_space[ny][nx], ny, nx ] )
                                    count = count + 1
                                    c_space[ny][nx] = 0
                    break # 처음 폭탄을 만나면 다음 t값으로 넘어가야함으로 break문을 써야함.

        # 남은 폭탄이 위에서 아래로 fall
            for i in range(h-2,-1,-1):
                for j in range(w):
                    if c_space[i][j] != 0 and c_space[i+1][j] == 0:

                        for k in range(i+1,h):
                            
                            if k == h-1 and c_space[k][j] == 0:
                                c_space[k][j] = c_space[i][j]

                            elif c_space[k][j] != 0:
                                c_space[k-1][j] = c_space[i][j]
                                break

                        c_space[i][j] = 0

        # 남은 폭탄의 갯수를 구함.
        remain = 0 # 남은 폭탄을 나타내는 변수.
        for i in range(h):
            for j in range(w):
                if c_space[i][j] != 0:
                    remain = remain + 1

        answer.append((count,remain)) # 모든 경우를 일단 answer라는 list에 append 함. max 함수로 나중에 최대값을 구함.

    print('#',end='')
    print( '{0} {1}'.format( test_case, max(answer)[1] ) ) # max(answer)[1]는 가장 많이 폭발한 경우에서 => max(answer), 남은 폭탄의 갯수 => [1]를 출력함.
```

### # review

`SWEA` 문제 특성상, 문제를 충실히 읽고, 명세에 따라 `완전 탐색` 방법으로 `구현`하면 풀렸습니다.
`완전탐색` 방법에는 저는 `BFS`와 `from itertools import product`를 사용하였습니다. 
