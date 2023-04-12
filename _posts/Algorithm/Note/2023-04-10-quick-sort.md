---
title: "퀵 정렬 정리"

categories:
  - Note

tags:
  - algorithm
  - sort
  - note

toc: true
toc_sticky: true
toc_label: "퀵 정렬"
toc_icon: "pen"
last_modified_at: 2023-04-10

use_math: true 
---

<br>

# **1. 개요**

* 정복 후 분할하는 알고리즘

  * 문제를 2개의 부분 문제로 분할

  * 단, 부분 문제의 크기가 일정하지 않음

<br>

# **2. 이론**

리스트의 기준 원소를 피봇(pivot)으로 설정하고 피봇보다 작은 숫자들은 왼쪽, 피봇보다 큰 숫자는 오른쪽에 위치하도록 리스트를 분할한 후 피봇을 그 사이에 놓는다.

분할된 부분 리스트에 대해서도 위 과정을 순환 반복하며 피봇으로 선정된 원소들을 차례대로 정리한다.

<br>

# **3. 피봇 설정**

<img src="https://user-images.githubusercontent.com/93882395/230892968-f7f95500-0ea3-4926-8037-23f40b07e474.png" alt="image" style="zoom:67%;" /> 

 

* 피봇을 기준으로 리스트 분할 => 부분 리스트를 나누는 기준선이라고 생각하자
  * 분할된 리스트 불균등
    * 최악의 경우와 최악의 경우가 아닐 때 시간 복잡도가 다름
    * 합병 정렬과 같이 모든 경우에 대해 동일한 시간이 쓰이지 않음
  * 피봇 선택이 시간 복잡도에 큰 영향을 미침 
* 한 번 피봇으로 선정된 원소는 다음 정렬시 고려하지 않음
  * 이미 정렬되어 있기 때문
  * 지난 피봇들을 제외한 원소들이 정렬 범위가 된다!
* 피봇으로 가장 작은 숫자 또는 가장 큰 숫자가 선택되면 부분 리스트가 한 쪽으로 치우침
  * 최악의 경우 시간 복잡도: O(n^2) => **피봇 선택이 중요!**

<br>

# **4. 의사코드**

## **퀵 정렬** 

정렬 함수를 재귀 호출해 피봇을 기준으로 리스트를 반복 분할한다.

<img src="https://user-images.githubusercontent.com/93882395/230892994-aa035dc2-6a43-4c6b-bf2d-10c2dac311f4.png" alt="image" style="zoom:67%;" />  

```c
Algorithm QuickSort(A, left, right):
	// left 인덱스보다 right 인덱스가 왼쪽에 있으면
    if(left < right) 
        // partition을 호출해 피봇 p 결정
        p = partition(A, left, right);
		// 피봇을 기준으로 리스트 반복 분할 후 정렬(재귀 함수 호출)
        QuickSort(A, left, p - 1);
        QuickSort(A, p + 1, right);
```

<br>

## **파티션 알고리즘**

부분 리스트를 정렬하고 피봇 인덱스를 반환한다.

<img src="https://user-images.githubusercontent.com/93882395/230893014-75c0f1c4-b083-44ad-ae0b-6e722e15d7b4.png" alt="image" style="zoom:67%;" />  


```c
Algorithm partition(A, left, right) :
    // 두 원소의 위치를 서로 바꿈
    swap(A[left], A[p]);  // p = floor((left+right)/2) or left
    // 리스트의 가장 왼쪽에 있는 입력을 피봇으로 정함
    pivot = A[left];
    
    // i: 왼쪽에서 오른쪽으로 이동하는 인덱스
    i = left + 1; 
    // j: 오른쪽에서 왼쪽으로 이동하는 인덱스
    j = right; 
    
    // 두 인덱스가 교차되기까지 반복 수행 => 매 반복마다 i,j 한 칸씩 이동
    while(i < j)
    	// 피봇보다 i값이 클 경우 순환 마침
        while(i < right and A[i] < pivot) i++; 
        // 피봇보다 j값이 작을 경우 순환 마침
        while(j > left+1 and A[j] > pivot) j--; 
        // 두 인덱스가 서로 교차한다면, 배열의 left 위치와 right 위치 서로 교환
        if(i < j) swap(A[i], A[j]);
    // 피봇과 A[j]값 서로 바꾸어줌
    swap(A[left], A[j]); 
    
    return j;
```

**시간 복잡도**

<br>

* 최선의 경우: $O(N log N)$
* 평균의 경우: $O(N log N)$
* 최악의 경우: $O(N^{2})$

<br>

# **5. 피봇 선정 방법**

<br>

1. 랜덤하게 선정하는 방법
2. Median-of-Three
3. Median-of-Medians

<br>

## **Median-of-Three**



<img src="https://user-images.githubusercontent.com/93882395/230893045-3e410422-7665-4340-8746-47382f2c706b.png" alt="image" style="zoom:67%;" />  



리스트의 가장 왼쪽 숫자, 중간 숫자, 가장 오른쪽 숫자 중에서 중앙값으로 피봇을 결정한다.

* *eg) [31, 1, 26] 중 중앙값인 26을 피봇으로 사용*

<br>

## **Median-of-Medians**



<img src="https://user-images.githubusercontent.com/93882395/230893068-fa6447c5-3942-468b-bc33-49b3a7c70ba6.png" alt="image" style="zoom:67%;" />  



3등분한 후 각 부분에서 가장 왼쪽 숫자, 중간 숫자, 가장 오른쪽 숫자 중에서 중앙값 
을 찾은 후, 세 중앙값들 중에서 중앙값을 피봇으로 결정한다.

<br>

# **6. 성능 향상 방법**

* 입력의 크기가 매우 클 때, 퀵 정렬의 성능을 더 향상시키기 위해 **삽입  정렬**을 동시에 사용
  * 부분 문제의 크기가 크면 삽입 정렬 사용
  * 부분 문제의 크기가 일정 수준 이하로 작아지면 분할(순환 호출)을 중단하고 삽입 정렬 사용
    * *eg) 부분 문제의 크기가 25~50일 경우 분할 중지*
