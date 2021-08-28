# Python_Honey_Tips

## List를 출력하는 꿀팁

```Python
ls = [[1,1],[2,2],[3,3]]
print(' '.join(map(str, s)))
-----------------------------
출력 예제
1 1
2 2
3 3
# 단 한 줄로 간단히 출력 가능!
```

---

## 런타임 에러 (RecursionError)

최근 백준 실버2 문제를 풀다보니 DFS나 BFS같은 트리 탐색 문제가 나온다.
<br>
DFS같은 문제는 재귀를 이용해서 푸는데 파이참에서는 문제 없이 실행이 되지만 백준에서는 아래와 같은 오류가 뜬다.
<br>
![RecursionError](../assets/RecursionError.png)
<br>
**RecursionError**는 재귀와 관련된 에러이다. 가장 많이 발생하는 이유는 Python이 정한 최대 재귀 깊이보다 재귀의 깊이가 더 깊어질 때이다.
BOJ의 채점 서버에서 최대 깊이는 1000으로 제한되어있다.
<br>

```Python
import sys
sys.setrecursionlimit(10000)
```

위와 같이 **sys.setrecirionlimit()** 를 이용해 재귀깊이의 제한을 풀어주면 해결이 된다.
