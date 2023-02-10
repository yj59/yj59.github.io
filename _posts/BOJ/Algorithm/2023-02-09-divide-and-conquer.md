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

  

## 2. 합병 정렬

원소를 절반씩 나누어 각각 정렬한 후, 정렬 결과를 하나로 합치는 알고리즘이다.

리스트의 원소 개수가 1이 될 때까지 나누어야 이후 순서대로 정렬할 수 있으므로, 원소를 모두 분할하는데 $O(logN)$ 만큼의 시간이 걸린다.



<br>

<img src="https://user-images.githubusercontent.com/93882395/218068738-9e36c4f4-4cae-4bff-8284-bdba5311b7de.png" alt="image" style="zoom:33%;" /> 

리스트를 나누고 나면, 각 배열은 정렬되어 있으므로 각 배열의 원소 크기들을 비교해가며 새 배열으로 병합한다.

원소를 순회해 리스트를 한 번 합치는데 필요한 시간복잡도는 $O(N)$이므로, 배열을 모두 정렬하는데 걸리는 시간은 $O(N log N)$ 이 된다.

<br>

```c++
void sort(int start, int end)		// start번째부터 end번째까지 정렬하는 함수
{
    if (start == end) return;		// 수의 개수가 1개면 더이상 정렬할 수 없으므로 리턴해준다.

    int mid = (strart + end) / 2;
    
    sort(start, mid);			// 왼쪽 부분 정렬
    sort(mid + 1, end);			// 오른쪽 부분 정렬
    merge(start, end);			// 왼쪽과 오른쪽이 모두 정렬되어 있으므로 합병 수행
}

void merge(int start, int end)
{
    // mid를 기준으로 나눈 배열들의 각 첫 번째 원소 left, right
    // 임시 배열 인덱스 index (원소 덮어씌우기 방지용 배열)
    int mid = (start + end) / 2;
    int left = start, right = mid + 1; 
    int index = 0;			
    
    while(left <= mid && right <= end) //left는 mid까지, right는 end까지 순회
    {
        // left와 right 비교 후 더 작은 수 tmp에 적재
	if(list[left] <= list[right]) tmp[index++] = list[left++];
        else tmp[index++] = list[right++];
    }
    
    // 한 쪽에 남아있는 원소가 있을 경우 tmp에 그대로 적재
    while(left <= mid) tmp[index++] = list[left++];
    while(right <= end) tmp[index++] = list[right++];
    
    // 임시 배열 원소를 다시 list로 복사
    for(int i = start; i <= end; i++)
        list[i] = tmp[i - start];
}
```

