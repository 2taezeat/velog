### # 문제 출처
https://programmers.co.kr/learn/courses/30/lessons/67258

### # 풀이1
```python
def solution(gems):
    set_gem = set(gems)
    length = len(set_gem)
    gnqh = []
    dict1 = dict()
    for i in range(len(gems)):
        try:
            del dict1[gems[i]]
        except:
            pass
    
        dict1[gems[i]] = i

        if len(dict1) == length:
            vl = list(dict1.values())
            if vl[-1] == vl[0]:
                return vl[0]+1, vl[-1]+1
            else:
                gnqh.append([vl[-1]-vl[0], vl[0]+1, vl[-1]+1])
    gnqh.sort()
    return gnqh[0][1], gnqh[0][2]


#print( solution(["DIA", "RUBY", "RUBY", "RUBY", "RUBY", "EMERALD", "SAPPHIRE", "DIA"]) )
#print( solution(["DIA", "RUBY", "RUBY", "DIA", "DIA", "EMERALD", "SAPPHIRE", "DIA"]) )
# print( solution(["AA", "AB", "AC", "AA", "AC"]) )
# print( solution(["XYZ", "XYZ", "XYZ"]) )
# print( solution(["ZZZ", "YYY", "NNNN", "YYY", "BBB"]) )코드를 입력하세요
```
정확성 모두 통과
효율성 15번 testcase 만 통과하지 못함. (총 95.6점)

### # 풀이2
```python
from collections import defaultdict
def solution(gems):
    set_gem = set(gems)
    length = len(set_gem)
    dict1 = defaultdict(int)
    a1, a2 = 0, 999999999
    for i in range(len(gems)):

        if dict1[gems[i]] != 0:
            del dict1[gems[i]]
            dict1[gems[i]] = i + 1
        else:
            dict1[gems[i]] = i + 1

        if len(dict1) == length:
            vl = list(dict1.values())
            if vl[-1] == vl[0]:
                return vl[0], vl[-1]
            else:
                if a2-a1 > vl[-1]-vl[0]:
                    a1 = vl[0]
                    a2 = vl[-1]

    return a1,a2
    
#print( solution(["DIA", "RUBY", "RUBY", "RUBY", "RUBY", "EMERALD", "SAPPHIRE", "DIA"]) )
#print( solution(["DIA", "RUBY", "RUBY", "DIA", "DIA", "EMERALD", "SAPPHIRE", "DIA"]) )
# print( solution(["AA", "AB", "AC", "AA", "AC"]) )
# print( solution(["XYZ", "XYZ", "XYZ"]) )
# print( solution(["ZZZ", "YYY", "NNNN", "YYY", "BBB"]) )
```
정확성, 효율성 모두 통과 (100점)

### # review
효율성을 통과하기 위해 시간을 줄이는 방법을 계속 고민해 보았습니다. 결과적으로 다음 3가지를 수정하였습니다. 결과적으로 100점이 나왔습니다.

1. `try - except` -> `if else문`
2. 그냥 `dict` -> `defaultdict(int)` // `try문`을 사용하지 않기 위해
3. `gnph(후보리스트)`를 `for문` 을 다 수행하고, 난 후 리스트 정렬을 하기 보다는 
	`for`문 안에서 `a1,a2`를 갱신하는 방향을 택함.