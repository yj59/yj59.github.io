---
title: "분할 정복 알고리즘 정리"

categories:
  - Algorithm

tags:
  - algorithm

toc: true
toc_sticky: true
toc_label: "분할 정복"
toc_icon: "pen"
last_modified_at: 2023-02-09

use_math: true 
---

<br>

한 문제를 작은 문제로 나누어 해결하는 문제 유형을 분할 알고리즘, 각 문제들을 다시 합쳐 정답을 구하는 문제를 정복 알고리즘이라고 부른다.

*   DP와 다른 점
    *   다이나믹 프로그래밍은 분할한 작은 문제들이 서로에게 영향을 미침 => 중복
        *   이전 문제를 기록해 다음 문제 해결*(memorization)*
    *   분할 문제는 일반적으로 부분이 중복되지 않아 한 문제씩 해결해 풀이 가능


<br>

**분할 정복 알고리즘**

*   퀵 정렬
*   합병 정렬
*   큰 수 곱셈 (카라추바 알고리즘)
*   FFT



## 1. 이분 탐색

정렬되어 있는 리스트에서 특정 값을 빠르게 찾는 알고리즘이다. 이분 탐색 알고리즘은 $O(log N)$의 시간이 걸린다.

정렬이 되어 있지 않은 리스트에서 어떤 값을 탐색하려면 리스트에 있는 값들을 모두 살펴봐야 하므로 복잡도가 $O(N)$이 된다.

$N \to \frac{N}{2} \to \frac{N}{4} \to \frac{N}{8} \to 1$ => N을 절반으로 나누어 1을 만들었을 때의 횟수: $log N$

<br>

```c++
whlie(left <= right)		// list의 left와 right 사이에 원소 find 존재
{                               // left > right라면 해당 리스트에 find 원소 없음 => 반복문 종료
    mid = (left + right) / 2;	// mid 위치 갱신
    
    if(list[mid] == find)       // list에서 find원소를 찾았을 경우
    {
        position = mid;
        break;                  // 반복문 종료
    }
    else if(list[mid] > find)   // find가 mid보다 작은 경우
        right = mid - 1;	// right를 mid의 한 칸 왼쪽으로 이동
    else                        // find가 mid보다 큰 경우
        left = mid + 1;		// left를 mid의 한 칸 오른쪽으로 이동
}
```

  
