### # 문제출처
https://programmers.co.kr/learn/courses/30/lessons/49189

### # 풀이
```python
import heapq
import sys
input = sys.stdin.readline
INF = int(1e9)

def solution(n, edge):
    start = 0
    graph = [[] for i in range(n)]
    # visited = [False] * (n+1)
    distance = [INF] * (n)
    answer = 0
    for (a,b) in edge:
        graph[a-1].append((b-1,1)) # a번 노드에서 b번 노드로 가는 비용이 1이라는 의미
        graph[b-1].append((a-1,1)) # b번 노드에서 a번 노드로 가는 비용이 1이라는 의미

    def dijkstra(start):
        q = []
        # 시작 노드로 가기 위한 최단 경로는 0으로 설정하여, 큐에 삽입
        heapq.heappush(q, (0,start))
        distance[start] = 0

        while q : # 큐가 비어 있지 않다면
            # 가장 최단 거리가 짧은 노드에 대한 정보 꺼내기
            dist, now = heapq.heappop(q)

            # 현재 노드가 이미 처리된 적이 있는 노드라면 무시
            if distance[now] < dist:
                continue

            # 현재 노드와 연결된 다른 인접한 노드들을 확인
            for i in graph[now]:
                cost = dist + i[1]
                # 현재 노들를 거쳐서, 다른 노드로 이동하는 거리가 더 짧은 경우

                if cost < distance[i[0]]:
                    distance[i[0]] = cost
                    heapq.heappush(q, (cost, i[0]))

    dijkstra(start)
    m = max(distance)
    answer = distance.count(m)

    return answer
```



### # review
최단경로 알고리즘 중에 `다익스트라` 알고리즘을 선택하였습니다. 
`무방향`그래프 임으로, a->b , b->a 의 비용을 모두 1로 `graph`에 추가하였습니다.
