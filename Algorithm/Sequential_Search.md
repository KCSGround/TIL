## 순차 탐색

처음부터 해당 수를 찾아가서 그 값에 도달하면 그 위치를 리턴한다.

만약 끝까지 찾지 못한다면 -1을 리턴하는 알고리즘이다.

### <span style="color:tomato">시간 복잡도 : O(n)</span>

```Python
# 리스트에서 특정 숫자 위치 찾기
# 입력 : 리스트 ls, 찾을 값 x
# 출력 : 찾으면 그 값의 위치, 못 찾으면 -1
def seq_search(ls, x):
    n = len(ls) # 배열의 크기
    for i in range(n): # 리스트 ls의 모든 값을 탐색
        if x == ls[i]: # 해당하는 값을 찾으면
            return i   # 위치를 리턴
    return -1 # 못 찾으면 -1을 리턴
```
