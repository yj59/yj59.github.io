---
title: "[이득우의 C++ 언리얼] #7 애니메이션 시스템 설계"

header:
  overlay_image: https://user-images.githubusercontent.com/93882395/212655591-8ae50295-2acd-4d6b-9a8d-b6924f334ab3.jpg
  overlay_filter: 0.7 # 투명도

categories:
  - Unreal Tutorials

tags:
  - unreal
  - tutorial
  - game 
  - IDWcpp

toc: true
toc_sticky: true
toc_label: "Unreal Tutorial"
toc_icon: "fa-regular fa-gamepad"
last_modified_at: 2023-01-17

use_math: true
---

> ***🤍ref.***
>
> 1. **이득우의 언리얼 C++ 게임 개발의 정석**,  *이득우, 에이콘출판사, 2018*
> 2. [Unreal Engine 4.27 Documentation](https://docs.unrealengine.com/4.27/ko/)
>     *   [유니티 개발자를 위한 언리얼 엔진4](https://docs.unrealengine.com/4.27/ko/Basics/UnrealEngineForUnityDevs/)

***✨info***<br>튜토리얼 교재는 4.19v을 사용하나, 4.27v으로 실습 후 게시글을 작성하였습니다.<br>새로 알게 된 내용과 추가로 공부한 내용 위주로 작성합니다. 배우는 과정이라 부족한 점이 많습니다.😊
{: .notice--info}

<br>

---

# **1. 애니메이션 기본 제작**

다양한 상황에 맞는 애니메이션을 재생할 수 있도록 애니메이션 블루프린트 기능을 활용한다.

-   **애님 인스턴스**: 애니메이션 블루프린트의 기반
    -   스켈레탈 메시를 소유하는 폰의 정보를 받아 애님 그래프가 참조할 데이터 제공
    -   블루프린트, C++ 모두 사용
-   **애님 그래프**: 시각적 도구를 사용해 애니메이션 시스템을 제작하도록 설계
    -   애님 인스턴스의 변수 값에 따라 변화하는 애니메이션 시스템을 설계하는 공간
    -    블루프린트로만 제작 가능
    -    체계적인 애니메이션 시스템 설계를 위해 **스테이트 머신** 제공

<br>

`CurrentPawnSpeed`라는 이름의 `float` 변수를 만들어 이동 속도에 따라 애니메이션을 다르게 구현해보자.

**ABAnimInstance.h**

```c++
#pragma once

#include "ArenaBattle.h"
#include "Animation/AnimInstance.h"
#include "ABAnimInstance.generated.h"

UCLASS()
class ARENABATTLE_API UABAnimInstance : public UAnimInstance
{
	GENERATED_BODY()
	
public:
	UABAnimInstance();

private:
	// CurrentPawnSpeed 변수를 애님 그래프에서 사용할 수 있도록 UPROPERTY와 EditAnywhere 추가
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = Pawn, Meta = (AllowPrivateAccess = true))
	float CurrentPawnSpeed;
};
```

**블루프린트 접근 방법**

*   `BlueprintReadOnly` 
*   `BlueprintReadWrite`

<br>

**ABAnimInstance.cpp**

```c++
#include "ABAnimInstance.h"

UABAnimInstance::UABAnimInstance()
{
	CurrentPawnSpeed = 0.0f;
}
```

코드를 컴파일하면 애님 인스턴스의 속성을 애니메이션 블루프린트에서 확인할 수 있다.

<br>

**애니메이션 블루프린트 적용**

1.   툴바의 **클래스 세팅**을 눌러 부모 클래스 정보 변경

     *   클래스 변경 후 컴파일

     <img src="https://user-images.githubusercontent.com/93882395/222753428-b5a33fd9-5cfa-4a11-8136-1595f1c9f9a7.png" alt="image" style="zoom: 80%;" />

     <br> 

2.   상속된 변수 표시 박스를 체크해 `CurrentPawnSpeed` 변수 노출

     <img src="https://user-images.githubusercontent.com/93882395/222753247-8e589f91-94b7-4bab-9173-9e1ed97f1634.png" alt="image" style="zoom: 80%;" />

     <br> 

3.   변수를 작업 공간에 드래그하고 `Get -`메뉴 선택

     *   *`BlueprintReadWrite`로 선언했다면 `Set`활성화됨* 

4.    `Get`노드를 드래그한 후 `float > float` 노드 선택

      <img src="https://user-images.githubusercontent.com/93882395/222908273-d63d5024-6287-48c9-93a1-612900abfbc7.png" alt="image" style="zoom:67%;" /> 

      <br>

5.   비교값이 `bool` 형식으로 나오므로 `bool로 포즈 블렌딩` 노드 추가

6.   `True/False`포즈에 맞는 애니메이션 연결

     <img src="https://user-images.githubusercontent.com/93882395/222908318-1b6edfd4-520a-49cb-bd5f-895193c4c646.png" alt="image" style="zoom:67%;" /> <br>

**애님 프리뷰 에디터**에서 `CurrentPawnSpeed` 값마다 다른 애니메이션을 재생하는 캐릭터를 확인할 수 있다.

<br>

## **1.2. 폰과 데이터 연동** 

게임을 플레이하면서 실제 폰의 속도에 따라 다른 애니메이션을 재생하기 위해서는 프레임마다 폰의 속력과 애님 인스턴스의 `CurrentPawnSpeed` 값이 동일해야 한다.

<br>

**폰과 애님 인스턴스 데이터 연동**

*   폰의 `Tick` 함수에서 애님 인스턴스의 `CurrentPawnSpeed` 조정
*   애님 인스턴스의 `Tick`에서 폰의 속도 정보를 가져와 `CurrentPawnSpeed` 업데이트
    *   애님 인스턴스 클래스에서 틱마다 호출되는 `NativeUpdateAnimation`  가상 함수 제공

<br>

**ABAnimInstance.h**

```c++
public:
	UABAnimInstance();
	virtual void NativeUpdateAnimation(float DeltaSeconds) override;
```



 <br>

**ABAnimInstance.cpp**

```c++
void UABAnimInstance::NativeUpdateAnimation(float DeltaSeconds)
{
	Super::NativeUpdateAnimation(DeltaSeconds);

	// 유효한 Pawn 오브젝트의 포인터 받아옴
	auto Pawn = TryGetPawnOwner();
	if (::IsValid(Pawn))
	{
		CurrentPawnSpeed = Pawn->GetVelocity().Size();
	}
}
```

*   `TryGetPawnOwner` : 폰에 접근해 데이터 제공

<br>

## **1.3. 스테이트 머신 제작**

기계가 반복해야 하는 동작을 스테이트로 정의하고 스테이트 머신이 지정된 스테이트 동작을 반복 수행한다.

캐릭터가 반복해서 재생해야 할 애니메이션 동작을 애니메이션 블루프린트 스테이트 머신이 처리하도록 한다.

1.   애님 그래프의 빈 공간을 우클릭해 `스테이트 머신 새로 추가` 클릭
2.   스테이트 머신을 더블 클릭하고, 머신 편집 윈도우에서 우클릭->`스테이트 추가`
3.   시작 노드와 추가한 스테이트를 트랜지션으로 연결
     *   **트랜지션**: 시작->목적 드래그시 생성되는 단방향 화살표
         *   서로 다른 스테이트끼리 이동이 가능하도록 연결
4.   추가한 스테이트를 더블 클릭하고 해당 스테이트의 애님 그래프 제작
5.   스테이트 작업을 완료하면 컴파일해 애니메이션 시스템 테스트

<br>

![image](https://user-images.githubusercontent.com/93882395/222880490-66147c32-b472-472c-9e88-b34977929876.png) 

>   애님 그래프 결과

<br>

![image](https://user-images.githubusercontent.com/93882395/222880524-59e690ac-a08f-4561-943e-cb6b1812e534.png) 

>   Ground 스테이트 애니메이션

<br>

## **1.4. 점프 기능 구현**

`ACharacter` 클래스의 `Jump` 멤버 함수를 `Jump` 입력과 연동시켜 캐릭터의 점프 기능을 구현할 수 있다. `Jump` 멤버 함수는 입력과 바인딩할 수 있도록 인자와 반환이 없는 함수로 선언되어 있다.

<br>

**ABCharacter.cpp**

```c++
AABCharacter::AABCharacter()
{
	...
        
	// 캐릭터 무브먼트 컴포넌트를 가져와 JumpZVelocity 값을 변경해 점프 높이 조절
	GetCharacterMovement()->JumpZVelocity = 800.0f;
}

void AABCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
        Super::SetupPlayerInputComponent(PlayerInputComponent);

	// 액션 입력 값과 바인딩
	PlayerInputComponent->BindAction(TEXT("Jump"), EInputEvent::IE_Pressed, this, &AABCharacter::Jump);
    
    ...
}
```

<br>

캐릭터가 점프하는 도중에는 속력이 발생하므로 달리기 애니메이션이 재생된다. 점프 모션을 재생하도록 애니메이션 블루프린트를 수정해보자.

폰의 무브먼트 컴포넌트에서 현재 캐릭터의 이동 상황을 얻어 애니메이션 블루프린트에게 전달할 수 있다.

<br>

**무브먼트 컴포넌트가 제공하는 캐릭터 움직임**

*   `IsFalling()` : 캐릭터가 공중에 떠 있음
*   `IsSwimming()` : 캐릭터가 수영 중임
*   `IsCrouching()` :캐릭터가 앉아 있음
*   `IsMoveOnGround()` : 캐릭터가 땅 위에서 이동

<br>

**ABAnimInstance.h**

```c++
UCLASS()
class ARENABATTLE_API UABAnimInstance : public UAnimInstance
{
    ...
        
private:
    // 애님 인스턴스에 캐릭터가 공중에 떠 있는지에 대한 불리언 속성 선언
	UPROPERTY(EditAntwhere, BlueprintReadOnly, Category = Pawn, Meta = (AllowPrivateAccess = true))
	bool IsInAir;
    
    ...
};
```

<br>

**ABAnimInstance.cpp**

```c++
UABAnimInstance::UABAnimInstance()
{
	CurrentPawnSpeed = 0.0f;
	IsInAir = false;
}

void UABAnimInstance::NativeUpdateAnimation(float DeltaSeconds)
{
	Super::NativeUpdateAnimation(DeltaSeconds);

	auto Pawn = TryGetPawnOwner();
	if (::IsValid(Pawn))
	{
		CurrentPawnSpeed = Pawn->GetVelocity().Size();
	}
	auto Character = Cast<ACharacter>(Pawn);
	if (Character)
	{
		// 폰 무브먼트 컴포넌트의 IsFalling 함수를 호출해 두 값을 일치시킴
		IsInAir = Character->GetMovementComponent()->IsFalling();
	}
}
```

<br>

**애니메이션 시스템 설계**

1.   스테이트 머신 편집 화면에서 `Jump` 스테이트 추가
2.   `Ground`와 `Jump` 간 양방향 트랜지션 추가
3.   `IsInAir`을 기반으로 트랜지션 발생 조건 부여
     *   트랜지션의 애니메이션 그래프에서 `IsInAir` 연결
     *   반전은 `Not Boolean` 노드 사용

<br>

 ![녹화_2023_03_04_21_29_07_8_AdobeExpress](https://user-images.githubusercontent.com/93882395/222901149-218ca9ba-244d-4a1b-aa09-a18d11b064fa.gif) 

>   결과 화면

<br>

---

# **2. 외부 애니메이션 적용**

## **2.1. 애니메이션 리타겟**

인간형 캐릭터에서 다른 마네킹의 애니메이션을 사용하고 싶을 경우 애니메이션 리타겟 기능을 활용해보자. 스켈레톤의 구성이 달라도 애니메이션 리타겟 기능을 사용하면 애니메이션을 재생할 수 있다.

<br>

**애니메이션 리타겟 사용 과정**

1.   애니메이션을 교환할 캐릭터들의 스켈레톤 세팅
     *   마네킹의 스켈레톤 메시 에디터 오픈
     *   툴바 `리타겟 매니저` -> `릭 셋업` -> `릭 선택` -> `인간형 릭`
     *   리타겟 설정이 완료된 마네킹 스켈레탈 애셋 저장
     *   캐릭터의 스켈레탈도 같은 인간형 릭에 매핑
2.   마네킹의 점프 애셋을 선택하고 우클릭 -> `애님 애셋 리타겟` -> `애님 애셋 복제 후 리타겟`
3.   애니메이션을 옮길 대상 스켈레톤 지정
4.   대체 메뉴에 변환할 정보 기입
5.   복제될 애니메이션 애셋이 저장될 대상 폴더 지정
6.   좌측 하단의 `리타겟` 버튼을 눌러 애니메이션 리타겟
7.   새로 생성된 애니메이션 애셋 모두 저장

<br>

![image](https://user-images.githubusercontent.com/93882395/222905799-ecb5ad7b-64c2-4fca-9a8d-bfc1e2a7f960.png)  

>   릭 매핑 이후 애니메이션 에셋 리타겟 과정

<br>

## **2.2. 점프 구현**

점프 동작*(도약-체공-착지)*에 따라 애니메이션 종류와 재생 시간을 다르게 해 자연스러운 점프 기능을 구현해보자.

도약과 착지 상황에서는 애니메이션을 한 번만 재생하고 체공 상황에서는 몸이 부양하는 애니메이션을 반복 재생한다.

<br>

**애니메이션 조절**

1.   스테이트 확장(`Ground`->`JumpStart`, `JumpLoop`->`JumpEnd`)
2.   스테이트별 트랜지션 조건을 다르게 지정
3.   각 스테이트마다 애니메이션 배치 후 최종 포즈와 연결
     *   한 번만 재생할 경우 `Loop Animation` 옵션 해제
4.   `JumpStart`->`JumpLoop` 트랜지션 생성
     *   스테이트에 애니메이션을 설정하면 트랜지션에서 해당 스테이트 애니메이션 관련 노드 사용 가능
5.   `Time Remaining` 노드 생성
     *   스테이션에서 사용한 애니메이션 재생의 남은 시간 반환
         *   `ratio` 속성 노드 선택시 애니메이션 재생 시간비 반환*(eg. 0.1 -> 10%)*
6.   남은 시간 비율이 0.1보다 작으면 스테이트를 이동하도록 노드 연결
7.   단발성 애니메이션 트랜지션의 `Automatic Rule Based on Sequence Player in State` 체크
     *   애니메이션 종료 시 자동으로 스테이트 전환 

<br>

![image](https://user-images.githubusercontent.com/93882395/222907507-929c7bbd-fbee-4ecc-9796-2ba3d7a85c80.png) 

>   트랜지션 시간비 설정
>
>   스테이트 트랜지션이 가능한지 평가해야 하므로 `bool`형식으로 변환해야 한다.

<br>

![녹화_2023_03_04_23_28_42_169_AdobeExpress](https://user-images.githubusercontent.com/93882395/222908150-05277b59-94fd-4de8-9a4f-bad578c057ed.gif) 

>   결과 화면
