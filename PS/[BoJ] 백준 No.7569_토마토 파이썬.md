### # 문제 출처
https://www.acmicpc.net/problem/7569

### # 풀이
```python
import sys
from collections import deque
input = sys.stdin.readline
# input 받고, 3차원 space 배열 초기화
m,n,h = map(int,input().split())
space = [[] for i in range(h)]
result = 0
for j in range(h):
    for i in range(n):
        space[j].append(list(map(int,input().split())))
        
# 3차원 배열임으로, dz z축도 만들어야한다. 왼, 오, 아, 위, 앞, 뒤 6가지
dx = [1, -1, 0, 0, 0, 0]
dy = [0, 0, -1, 1, 0, 0]
dz = [0, 0, 0, 0, -1, 1]
visited = [[[False for i in range(m)] for i in range(n)] for i in range(h)]

def bfs():
    while(q):
        z,x,y = q.popleft()
        visited[z][x][y] = True

        for i in range(6):
            nx = x + dx[i]
            ny = y + dy[i]
            nz = z + dz[i]

            if nx > -1 and ny > -1 and nz > -1 and nx < n and ny < m and nz < h and (space[nz][nx][ny] == 0):
                if visited[nz][nx][ny] == False:
                    space[nz][nx][ny] = space[z][x][y] + 1
                    visited[nz][nx][ny] = True
                    q.append( [nz,nx,ny] )

# 1번 케이스 체킹 함수
def checking():
    for z in range(h):
        for x in range(n):
            for y in range(m):
                if space[z][x][y] == 0:
                    return 9999

# 처음에 토마토가 있는 좌표를 q에 추가
q = deque()
c = 0
t = 0
for z in range(h):
    for x in range(n):
        for y in range(m):
            if space[z][x][y] == 1:
                q.append([z,x,y])
                # 정답이 0 인 경우 체크하기 위해서
                c = c + 1

            if space[z][x][y] == -1:
                t = t + 1

if c == (m*n*h):
    print(0)
elif t == (m*n*h):
    print(-1)
else:
    
    bfs()
    if checking() == 9999:
        print(-1)
    else:
        for z in range(h):
            for x in range(n):
                for y in range(m):
                    result = max(result, space[z][x][y])
        
        print(result-1)
```

### # review
`완전 탐색` 의 방법으로`BFS`을 사용하였습니다. `3차원` 배열을 활용해야 하는 것이 특이했습니다.