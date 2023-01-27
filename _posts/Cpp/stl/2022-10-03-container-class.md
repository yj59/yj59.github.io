---
title: "[C++] STL 컨테이너 클래스 정리 & 종류"

categories:
  - CppSTL

tags:
  - cpp
  - stl

toc: true
toc_sticky: true
toc_label: "Container"
toc_icon: "Pen"
last_modified_at: 2022-10-03

use_math: true
---



> ***💚ref.***
>
> 1. [*cplusplus Reference*](https://cplusplus.com/reference/)
> 2. [**Microsoft Learn** *C++ STL Container*](https://learn.microsoft.com/ko-kr/cpp/standard-library/cpp-standard-library-reference?view=msvc-170)
> 3. [**geeksforgeeks** *The C++ Standard Template Library*](https://www.geeksforgeeks.org/the-c-standard-template-library-stl/)

<br>

---

# 1. 컨테이너 클래스란?

데이터나 객체를 저장해주는 클래스 템플릿이다. C++ 코드를 효율적으로 작성하기 위한 자료구조 클래스라고 생각하자. 컨테이너 변수를 선언할 때는 어떤 타입의 컨테이너를 선언할지 미리 명시한다.

<br>

배열, 스택, 큐 집합 등 여러 자료구조가 컨테이너 클래스에 정의되어 있다. STL 특징에 맞게 템플릿으로 작성되어 모든 타입에 컨테이너를 적용할 수 있다! 백준 문제를 풀 때 자료형을 상관하지 않고 부담없이 컨테이너 클래스인  `vector` 클래스를 사용했다.

<br>

**👀알아두기)** *이터레이터 (Iterator)*<br>컨테이너 클래스로 생성한 원소들을 참조하기 위해 이터레이터를 사용한다.<br>이터레이터는 클래스로 구현된 포인터와 유사한 객체로, 컨테이너 클래스의 원소들을 동일한 접근 방식으로 참조할 수 있다. <br>사용자가 어떤 알고리즘을 사용하느냐에 따라 다른 방식으로 컨테이너에 접근하기 때문에 이터레이터에도 다양한 종류가 있다!<br>
어렵게 생각하지 말기. `vector`클래스의 멤버 함수 `begin`, `end`, 비교연산자 등이 모두 이터레이터에 해당한다.
{: .notice--primary}
<br>

---

# 2. 컨테이너 클래스의 종류

>   블로그에서 관련 컨테이너를 따로 작성한 문서가 있으면 클래스명에 포스팅 링크 걸어두겠습니다!🤓

컨테이너 클래스를 사용하려면 해당 클래스의 헤더와 `std` 네임스페이스 선언이 필요하다.

<br>

### **2.1. 시퀀스 컨테이너**

시퀀스 컨테이너(*sequence container*). 데이터를 선형으로 저장한다. 원소들끼리 순서가 정해져있는 일반적인 자료구조에 해당한다. 특정한 위치의 원소에 접근해 값을 삽입하거나 삭제할 수 있다.

<br>

🎞️**클래스 종류**

*   **[vector](https://yj59.github.io/cppstl/vector/)**(`<vector>`): 가변 배열. 임의 위치 접근, 배열 말단 삽입/삭제 가능
*   **deque**(`<deque>`): Double-ended queue. 상단과 말단 모두 삽입/삭제 가능
*   **list**(`<list>`): Doubly-linked list. 임의 위치 삽입/삭제
*   **forward_list**(`<forward_list>`): Singly-linked list. 임의 위치 삽입/삭제
    *   list에서 성능이 향상된 버전의 리스트. 단, 성능 향상을 위해 양방향 이동 불가능
*   **array**(`<array>`): 고정 배열

<br>

원하는 위치의 원소 삽입과 삭제가 잦은 알고리즘에서는 vector보다 list 사용을 권장한다. **vector** 클래스는 배열 기반이므로 임의의 위치에 원소를 삽입하고 삭제하는 과정이 list보다 비효율적이다(배열 말단 부분의 삽입/삭제만 빠르다). 자료구조의 형태를 떠올리면 쉽게 예측할 수 있다.

배열의 최상단과 말단에서만 삽입/삭제가 빈번하면 **deque**가 바람직하다!

<img src="https://user-images.githubusercontent.com/93882395/213268871-90eb3cbe-b267-4d00-8cb7-c8613e3bbca5.png" alt="image" style="zoom: 33%;" />  

>   *(간단히 그려본 vector와 list의 구조)*
>
>   vector의 중간에 요소를 삽입하거나 삭제하면 시퀀스를 유지하기 위해 메모리를 재할당한다. 특정 위치에 원소를 삽입/제거하는 작업은 list가 효율적이다(원소 전체에 대한 메모리 할당이 필요하지 않음).
>
>   반대로 list는 인덱스로 원소에 접근할 수 없고 원소 순회가 느리다. 원소를 자주 순회해야 하는 경우엔 배열 형식의 자료구조를 활용하자.



<br>

### **2.2. 컨테이너 어댑터**

컨테이너 어댑터(*container adapter*). 시퀀스 컨테이너를 기반으로 만들어진 컨테이너 클래스이다. 기존 컨테이너의 인터페이스를 제한해 특정 기능에 중점을 두거나 임의의 기능으로 변환되어 동작한다. 예를 들어, 컨테이너 어댑터인 **stack**의 `push`는 내부적으로 **deque** 객체에서 `push_back` 멤버 함수를 호출한다.

<br>

🎞️**클래스 종류**

*   **stack**(`<stack>`): LIFO 스택
*   **[queue](https://yj59.github.io/cppstl/queue/)**(`<queue>`): FIFO 큐
    *   **priority_queue**(`<queue>`): 우선순위 큐

<br>

### **2.3. 연관 컨테이너**

연관 컨테이너(*associate container*). 원소의 키와 값을 한 쌍으로 저장한 컨테이너이다. 원하는 값/키에 대한 요소에 빠르게 접근할 수 있다. 특정 원소를 찾아야 하는 알고리즘을 생각할 때 연관 컨테이너를 가장 먼저 고려하자. 단, 원소가 삽입되는 위치를 정할 수 없다.

>   시퀀스 컨테이너와 장단점이 명확하게 갈린다!

<br>

🎞️**클래스 종류**

*   **set**(`<set>`): 집합
    *   **multiset**(`<set>`): 중복 허용 집합
*   **map**(`<map>`): 키-값 연결
    *   **multimap**(`<map>`): 키-중복키 연결

**set**은 키와 값이 동일해 한 객체에 반드시 한 개의 키만 존재한다. **map** 역시 하나의 키와 하나의 값이 연결된다. 

중복 데이터를 포함한 객체를 한 개의 키에 연결하고 싶을 경우 **multiset**을, 한 개의 키와 여러 개의 데이터를 연결하고 싶을 경우 **multimap**을 사용한다. **multiset**과 **multimap**은 모두 기존 클래스의 헤더 파일에 함께 정의되어 있다.

---

**👀알아두기)** *pair*<br>`<utility>` 헤더 파일 안에 **[pair 구조체](https://learn.microsoft.com/ko-kr/cpp/standard-library/pair-structure?view=msvc-170)**가 들어 있다. **pair**는 두 개의 원소를 단일 객체로 저장하는 템플릿 클래스이며 주로 컨테이너 클래스의 요소가 된다. 백준 문제를 풀 때 자주 활용하니 사용법에 꼭 익숙해지자.
{: .notice--primary}
