---
layout: post
title: 셈틀 게임 스터디_3 총알 로직 구현
author: Yejin an
tags:
- game
- unity
date: 2022-05-24 04:26 +0800
---

# 0526_samtle_game



총기류 발사 로직 예제를 통한 물리 현상 함수 공부



1. Projectile
   * 총알이 물리적으로 발사
     * 총알이 시각적으로 표현
     * 여러 개의 객체를 생성하면 게임의 속도 저하
   * 로우폴리 모델이나 평면 메시 모델에 텍스처 입혀 사용
   * 총알 => 객체를 미리 생성해 두고 번갈아 가며 발사되도록 구현 (오브젝트 풀)
2. Raycast
   * 실제 총알 발사 x
   * 눈에 보이지 않는 광선을 통해 물체 검출
   * 총알 발사 루틴 및 탐사 센서나 추적 기능 구현 가능



---

공부한 내용



1. Rigidbody
   * AddForce
   * AddRelativeForce

4. 물리엔진 속성 설정(Physics Manager)

5. Colider 컴포넌트
   * 충돌 감지 조건 (이벤트)
   * Tag 활용
   * OnCollisionEnter 콜백 함수
   * CompareTag 함수

8. 기즈모 활용



---



### 1. Rigidbody 컴포넌트

* 잘 부서지지 않는 대부분의 고체 속성!
  * 충돌 감지 , 물리 시뮬레이션을 위한 컴포넌트

```c#
rb.AddForce(transform.forward * force);
GetComponent<Rigidbody>().AddRelativeForce(Vector3.forward * force)
```

* void AddForce(Vecter3 force)
  * force: 월드에서 힘이 가해지는 좌표
  * 게임 오브젝트의 좌표를 기준으로 힘을 주려면 **transform.**forward * force
* void AddRelativeForce(Vector3 force)
  * 게임 오브젝트의 로컬 좌표를 기준으로 힘을 주는 함수



---



## 2. 물리 엔진 속성 설정

<img src="https://user-images.githubusercontent.com/93882395/170451426-f47a2c39-8335-4912-b8c9-397a3874c00b.png" alt="image-20220526145103010" style="zoom:67%;" />  

> [Edit] -> [Project Setting] -> [Physics]



* 유니티가 관리하는 물리 엔진의 설정값들을 확인 가능
* 대표적인 물리 엔진 속성
  * Gravity 
    * Rigidbody의 Use Gravity 속성에 체크할 경우 중력 작용
    * 중력가속도(-9.8m/s^2) 적용
  * Default Material
    * 두 물체가 충돌했을 때 반작용하는 물리적 속성
    * Rigidbody 컴포넌트에서 개별 설정 가능
  * Bound Threshold
    * 충돌한 두 물체의 상대 속도를 계산하여 충돌 여부 관리
    * 두 물체의 충돌 속도가 Bound Threshold보다 작을 경우 충돌 발생 x
  * Layer Collision Matrix
    * 충돌 감지 여부를 선택적으로 활성화하거나 비활성화할 수 있는 옵션
    * 게임 오브젝트가 특정 레이어에서만 물리 엔진의 영향을 받게 할 수 있다 !



---



## 3. Collider 컴포넌트

* 유니티에서 제공하는 충돌 감지 센서
  * 충돌하는 게임 오브젝트의 대상이 어떤지, 충돌 감지 계산을 얼마나 빠르게 하느냐에 따라 컴포넌트 변경

<img src="https://user-images.githubusercontent.com/93882395/170451357-b20b1689-770f-4dd3-b123-d301e49647fc.png" alt="image-20220526150242903" style="zoom:80%;" />  

1. Box Collider: 박스의 center와 size를 조정하여 형태 조절. 가장 다양한 용도로 활용 가능



<img src="https://user-images.githubusercontent.com/93882395/170451290-8cd048fe-2a81-4e2c-b50b-e594dd300c0a.png" alt="image-20220526150350125" style="zoom:80%;" />  

2. Sphere Collider: radius 속성으로 구체의 반지름 조절. 속도 처리 빠름,  대부분의 경우 Sphere Collider 사용



<img src="https://user-images.githubusercontent.com/93882395/170451167-ff797e5d-ed53-4f4d-a13e-14a0a5e249d8.png" alt="image-20220526150518363" style="zoom: 80%;" />  

3. capsule Collider :  캡슐의 높이 조정, 인체/나무/가로등과 같은 오브젝트의 충돌체

---

> 어떻게 사용할까?

 

* 물체가 다른 대상에 부딪힐 때의 충돌을 감지하려면 두 가지 조건을 만족해야 함

  1. 충돌을 일으키는 양쪽 게임 오브젝트 모두 Collider 컴포넌트가 추가되어 있어야 함

     * ( 3D 오브젝트를 생성했을 때, Default값으로 Collider 컴포넌트가 추가되어 있음 )

       <img src="https://user-images.githubusercontent.com/93882395/170451076-18434b48-e254-40b4-9e52-a6d2fa59199e.png" alt="image-20220526151054227" style="zoom:67%;" />  

  2. 두 게임 오브젝트 중 움직이는 쪽에는 반드시 Rigidbody 컴포넌트가 있어야 한다.



<img src="https://user-images.githubusercontent.com/93882395/170451016-c5118f2a-e999-4a32-a9c3-7c54e7fd3ec0.png" alt="image-20220526151518364" style="zoom:67%;" />  

> 충돌을 감지하기 위해 각 오브젝트에 필요한 컴포넌트 및 이벤트 스크립트



* Collider 컴포넌트를 추가할 경우, Collider의 크기를 해당 게임 오브젝트의 크기와 맞도록 조정해야 한다.(오브젝트 충돌 크기 확인)

---



### 4. 충돌 이벤트

* 게임 오브젝트 간 충돌이 발생하면 OnCollision 이벤트 함수 또는 OnTrigger 이벤트 함수 발생
  * 왜 이벤트 함수? => 언제 충돌이 발생할 지 모른다 ! 충돌이 감지되는 시점에 시스템에서 호출해줌
* OnColiision : 부딪힐 경우 충돌
* OnTrigger : 부딪히지 않고 관통 (감지 센서 역할로 쓰임)
* 각 이벤트 함수의 세부 함수명
  * ~ Enter : 물체 간 충돌이 일어나기 시작
  * ~ Stay : 충돌이 지속됨
  *  ~ Exit : 물체가 서로 떨어짐



**Tag 활용**

* Collider 컴포넌트를 가진 어떤 게임 오브젝트끼리 충돌이 일어났는지 알 수 있도록 오브젝트에 Tag 지정

  * 왜? 모든 오브젝트의 이름으로 구별하려면 경우의 수가 너무 많다 !

  

<img src="https://user-images.githubusercontent.com/93882395/170450960-2b3893ba-33a5-4bb2-a395-5beb3f237d0a.png" alt="image-20220526152322211" style="zoom:67%;" />  

> Inspector창의 Tag 메뉴를 통해 오브젝트의 태그 지정 및 새로 추가 가능



**CompareTag 함수**

```c#
    // 충돌이 시작할 때 발생하는 이벤트
    void OnCollisionEnter(Collision coll)
    {
        // 충돌한 게임오브젝트의 태그값 비교
        if (coll.collider.CompareTag("BULLET"))
        {
            // 충돌한 게임오브젝트 삭제
            Destroy(coll.gameObject);
        }
    }
```

>  충돌 시 게임 오브젝트의 태그를 확인하고 True일 경우 게임 오브젝트를 삭제하는 함수

* CompareTag("태그 문자열");
  * 비교할 태그명을 tag 인자로 넘겨주면 해당 게임 오브젝트의 태그와 비교 후 True/False 반환



----



## 5. 총알 발사 로직



총알 발사 ? 

* 하이러키 뷰 -> 플레이어의 본 구조를 따라 내려가 구현해둔 총 오브젝트의 하위로 총알 발사 위치 오브젝트 생성

* 런 모드에서 총구의 위치를 가늠하면서 총알 초기 위치 지정 가능

  

```c#
    void Start()
    {
        audio = GetComponent<AudioSource>();

        // FirePos 하위에 있는 MuzzleFlash의 Material 컴포넌트를 추출
        muzzleFlash = firePos.GetComponentInChildren<MeshRenderer>();
        // 처음 시작할 때 비활성화
        muzzleFlash.enabled = false;
    }

    void Update()
    {
        // 마우스 왼쪽 버튼을 클릭했을 때 Fire 함수 호출
        if (Input.GetMouseButtonDown(0))
        {
            Fire();
        }
    }
```



<img src="https://user-images.githubusercontent.com/93882395/170450887-52f774cf-ac51-48dd-8e1a-d503a381ea8f.png" alt="image-20220526165855366" style="zoom:67%;" /> 

* 총알 발사 스크립트를 플레이어에 적용
* 발사 시 이동할 오브젝트 Bullet 프리팹과 오브젝트 초기 위치 FirePos를 FireCtrl의 각 변수에 연결



---

## 6. 기즈모 활용



> 기즈모를 활용해 총구(총알 발사 위치)의 색상을 지정할 수 있음

```C#
void OnDrawGizmos()
    {
        // 기즈모 색상 설정
        Gizmos.color = _color;
        // 구체 모양의 기즈모 생성. 인자는 (생성 위치, 반지름)
        Gizmos.DrawSphere(transform.position, _radius);
    }
```



---



### 코드 작성 결과







![녹화_2022_05_26_17_49_46_950](C:\Users\lucet\Downloads\녹화_2022_05_26_17_49_46_950.gif)
