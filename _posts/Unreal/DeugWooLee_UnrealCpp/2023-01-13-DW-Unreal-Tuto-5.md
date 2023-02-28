---
title: "[이득우의 C++ 언리얼] #5 폰의 제작과 조작"

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
last_modified_at: 2023-01-13

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

# 1. 폰의 구성요소

<br>

**인간형 폰 제작 고려요소**

* **시각적 요소**: 리깅 데이터 추가(스켈레탈 메시), 스켈레탈 메시 컴포넌트
* **충돌 요소**: 인간형의 경우 캡슐 컴포넌트 사용
* **움직임 요소**: 폰 무브먼트 컴포넌트(플레이어 입력 반응)
  * `FloatingPawnMovement`, `CharactorMovement`
* **내비게이션**: 목적지를 알려주면 스스로 이동
* **카메라 출력**: 폰에 부착된 카메라 상을 화면으로 전송  

<br>

**ABPawn.h**

```c++
#include "ArenaBattle.h"
#include "GameFramework/Pawn.h"
#include "GameFramework/FloatingPawnMovement.h"
#include "ABPawn.generated.h"

UCLASS()
class ARENABATTLE_API AABPawn : public APawn
{
...
	
	UPROPERTY(VisibleAnywherem Category = Collision)
		UCapsuleComponent* Capsule;

	UPROPERTY(VisibleAnywhere, Category = Visual)
		USkeletalMeshComponent* Mesh;

	UPROPERTY(VisibleAnywhere, Category = Movement)
		UFloatingPawnMovement* Movement;

	UPROPERTY(VisibleAnywhere, Category = Camera)
		USpringArmComponent* SpringArm;

	UPROPERTY(VisibleAnywherem Category = Camera)
		UCameraCompoenet* Camera;
};
```

* `Capsule` : 폰의 움직임을 담당하는 충돌 컴포넌트
  * 폰을 대표해 움직임을 담당하므로 루트 컴포넌트로 설정
* `SkeletalMesh` : 캐릭터 애셋 출력과 애니메이션 담당
  * 모델링 프로그램과 언리얼 엔진의 좌표계가 다를 수 있으므로 두 좌표계 맞춤
    * 일반적으로 좌표계를 맞춘 후 z축으로 애셋의 절반 높이만큼 내려야 함
* `FloatingPawnMovement` : 플레이어의 입력에 따라 폰 움직임
  * 중력 고려 x
* `SpringArm` : 삼인칭 시점으로 카메라 구도 설정
* `Camera` : 카메라가 바라보는 게임 세계 화면을 플레이어 디스플레이로 송출
  * 카메라 컴포넌트를 스프링암 컴포넌트의 자식으로 설정 후 트랜스폼 정보 초기화 => 카메라가 스프링암의 끝에 자동 위치

<br>

**ABPawn.cpp**

```c++
#include "ABPawn.h"

AABPawn::AABPawn()
{
	PrimaryActorTick.bCanEverTick = true;
	
    // 컴포넌트 선언
	Capsule = CreateDefaultSubobject<UCapsuleComponent>(TEXT("CAPSULE"));
	Mesh = CreateDefaultSubobject<USkeletalMeshComponent>(TEXT("MESH"));
	Movement = CreateDefaultSubobject<UFloatingPawnMovement>(TEXT("MOVEMENT"));
	SpringArm = CreateDefaultSubobject<USpringArmComponent>(TEXT("SPRINGARM"));
	Camera = CreateDefaultSubobject<UCameraComponent>(TEXT("CAMERA"));
	
    // Capsule 컴포넌트를 루트 컴포넌트로 설정
	RootComponent = Capsule;
	Mesh->SetupAttachment(Capsule);
	SpringArm->SetupAttachment(Capsule);
    
    // 카메라를 스프링암의 자식으로 설정
	Camera->SetupAttachment(SpringArm);

    // 사용할 캐릭터 애셋이 캡슐 컴포넌트에 들어가도록 절반 높이(HalfHeight)와 둘레(Radius) 설정
	Capsule->SetCapsuleHalfHeight(88.0f);
	Capsule->SetCapsuleRadius(34.0f);
    
    // 좌표계를 맞추기 위해 메시의 회전값과 위치 조정
	Mesh->SetRelativeLocationAndRotation(FVector(0.0f, 0.0f, -88.0f), FRotator(0.0f, -90.0f, 0.0f));
    
    // 카메라 지지대 길이는 4미터(400.0f), 컴포넌트의 y축 회전을 -15로 설정(어깨 너머로 배경 출력)
	SpringArm->TargetArmLength = 400.0f;
	SpringArm->SetRelativeRotation(FRotator(-15.0f, 0.0f, 0.0f));

    // 스켈레탈 메시의 캐릭터 애셋 가져옴
	static ConstructorHelpers::FObjectFinder<USkeletalMesh> SK_CARDBOARD(TEXT("/Game/InfinityBladeWarriors/Character/CompleteCharacters/SK_CharM_Cardboard.SK_CharM_Cardboard"));
	if (SK_CARDBOARD.Succeeded())
	{
		Mesh->SetSkeletalMesh(SK_CARDBOARD.Object);
	}
}
...
```

<br>

---

# **2. 폰 움직임 조작**

## **2.1. 입력 설정**

`프로젝트 세팅`의 `입력` 설정에서 플레이어의 입력에 대한 처리 로직을 설정할 수 있다.

![image](https://user-images.githubusercontent.com/93882395/221734491-fb75cd45-90fd-4de5-a354-0e726e9b658f.png) 

* `액션 매핑` ; 조이스틱 번호의 신호 설정

* `축 매핑` : 조이스틱 레버의 신호 설정

  * 레버의 위치에 따라 `-1`, `0`, `1` 을 게임 로직으로 전달

    *eg) `A`키를 누르면 -1, `D`키를 누르면 1 전달. 아무 키도  누르지 않는다면 지속적으로 0 발생*

입력 설정 항목과 대응 키 값을 설정한 후 `InputComponent`를 사용해 폰의 멤버 함수와 입력 설정을 바인딩해보자. 입력 신호가 폰의 `SetupInputComponent` 함수를 통해 멤버 함수 인자로 전달된다.

<br>

**`InputComponent` 제공 함수**

* `BindAxis` 
* `BindAction`

<br>

**ABPawn.h**

```c++
#pragma once

#include "ArenaBattle.h"
#include "GameFramework/Pawn.h"
#include "GameFramework/FloatingPawnMovement.h"
#include "ABPawn.generated.h"

UCLASS()
class ARENABATTLE_API AABPawn : public APawn
{
	...

private:
	void UpDown(float NewAxisValue);
	void LeftRight(float NewAxisValue);
};
```

<br>

**ABPawn.cpp**

```c++
#include "ABPawn.h"

...

// Called to bind functionality to input
void AABPawn::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);
	
    //BindAxis 함수를 사용해 입력 설정의 이름(eg. UpDown)과 연동할 오브젝트 인스턴스의 함수 포인터 지정
	PlayerInputComponent->BindAxis(TEXT("UpDown"), this, &AABPawn::UpDown);
	PlayerInputComponent->BindAxis(TEXT("LeftRight"), this, &AABPawn::LeftRight);
}

void AABPawn::UpDown(float NewAxisValue) {
    //입력 값 확인 로그
	//ABLOG(Warning, TEXT("%f"), NewAxisValue);
    
    //입력 값을 무브먼트 컴포넌트에 전달
	AddMovementInput(GetActorForwardVector(), NewAxisValue);
}

void AABPawn::LeftRight(float NewAxisValue) {
	//ABLOG(Warning, TEXT("%f"), NewAxisValue);
	AddMovementInput(GetActorRightVector(), NewAxisValue);
}
```

* `AddMovementInput(WorldDirection, (입력 신호))` : -1~1 사이의 입력 값을 폰 무브먼트 컴포넌트에 전달*(엑셀)*
* `WorldDirection` : 추가로 이동할 방향의 벡터 데이터 전달*(핸들)*
  * `GetActorForwardVector()` : 전진방향
  * `GetActorRightVector()` : 좌우방향

<br>

플레이어의 입력값 로직은 폰 클래스에 구현하는 것이 일반적이다. 단, 플레이어의 모든 명령어는 플레이어 컨트롤러를 거치므로 플레이어 컨트롤러에 입력값 대응 코드를 작성하면 해당 코드가 우선 처리되고 폰으로 전달되지 않는다.

*(eg. 플레이어 컨트롤러에 좌우 멈춤 코드를 작성하면 좌우 방향키를 입력해도 폰이 움직이지 않는다.)*

<br>

## **1.2. 플레이어 컨트롤러 인터페이스**

언리얼 에디터에서 플레이 버튼을 누른 후 뷰포트로 포커스를 잡지 않아도 명령어가 전달되도록 플레이어 컨트롤러에 로직을 작성할 수 있다.

<br>

**ABPlayerController.h**

```c++
#pragma once

#include "ArenaBattle.h"
#include "GameFramework/PlayerController.h"
#include "ABPlayerController.generated.h"

UCLASS()
class ARENABATTLE_API AABPlayerController : public APlayerController
{
...
protected:
	virtual void BeginPlay() override;
};
```

<br>

**ABPlayerController.cpp**

```c++
#include "ABPlayerController.h"

...

void AABPlayerController::BeginPlay()
{
	Super::BeginPlay();

	FInputModeGameOnly InputMode;
	SetInputMode(InputMode);
}
```

<br>

---

# **3. 애니메이션 설정**

## 3.1. 애니메이션 애셋 적용

스켈레탈 메시에 애니메이션을 적용해 폰의 움직임을 자연스럽게 만들 수 있다.

<br>

**애니메이션 추가 과정**

1. `콘텐츠 브라우저`에서 애니메이션 애셋 생성 후 `임포트`
2. `애니메이션 임포트` 설정 창에서 해당 애니메이션이 적용될 스켈레톤 연결
3. `콘텐츠 브라우저`의 `모두 저장` 버튼을 눌러 변경된 애셋 저장
4. 애니메이션 애셋을 더블 클릭해 애니메이션이 잘 동작하는지 확인
   * `프리뷰 씬 세팅` 메뉴에서 스켈레톤 메시를 공유하는 다른 캐릭터로 애니메이션 재생 가능
5. 구현할 기능에 해당하는 애니메이션의 레퍼런스 주소 복사
   * *eg) 달리기를 캐릭터의 기본 애니메이션으로 지정 => 달리기 애니메이션 레퍼런스 복사*
6. `BeginPlay`함수에 `LoadObject<타입>` 명령어를 사용해 애니메이션 애셋 로드

<br>

**ABPawn.cpp**

```c++
#include "ABPawn.h"

...

void AABPawn::BeginPlay()
{
	Super::BeginPlay();
	
	Mesh->SetAnimationMode(EAnimationMode::AnimationSingleNode);

	UAnimationAsset* AnimAsset = LoadObject<UAnimationAsset>(nullptr, TEXT("/Game/Book/Animations/WarriorRun.WarriorRun"));
	if (AnimAsset != nullptr)
	{
		Mesh->PlayAnimation(AnimAsset, true);
	}
}
```

<br>

## 3.2. 블루프린트 활용

애니메이션 블루프린트를 사용해 언리얼에서의 애니메이션을 보다 체계적으로 관리할 수 있다.

<br>

**애니메이션 블루프린트 애셋 생성**

1. `콘텐츠 브라우저` -> `신규 추가` -> `애니메이션` -> `애니메이션 블루프린트`

2. 블루프린트와 연결될 스켈레톤 애셋 지정

3. 블루프린트를 연 후 애셋 브라우저에서 애니메이션을 애님 그래프(`Anim Graph`)에 로드

   ![image](https://user-images.githubusercontent.com/93882395/221755664-6ed1bb98-4184-4b5c-84c1-ed5884968194.png) 

4. 재생 노드에서 사람 모양의 핀을  최종 애니메이션 노드에 연결

   <img src="https://user-images.githubusercontent.com/93882395/221756040-0eae2c29-0bf6-4d36-a86a-d6a4f5ab2154.png" alt="image" style="zoom:80%;" /> 

<br>

`컴파일`과 `저장` 버튼을 눌러 프리뷰 창에서 애니메이션 재생 결과를 확인할 수 있다.

![ezgif com-video-to-gif (1)](https://user-images.githubusercontent.com/93882395/221756665-d0c3ffbd-fb03-4f8b-8b88-fec9aef5b3bb.gif) 

> 애니메이션 재생 프리뷰

애니메이션 블루프린트는 애님 그래프 로직에 따라 동작하는 **캐릭터 애니메이션 시스템**을 구동시킨다.

* 캐릭터 애니메이션 시스템: `애님 인스턴스(Anim Instance)` 클래스로 관리

  * 메시 컴포넌트가 자신이 관리하는 캐릭터의 애니메이션을 애님 인스턴스에 위임

    => 메시 컴포넌트가 애니메이션 블루프린트를 실행시키기 위해 **애셋의 클래스 정보를 애님 인스턴스 속성에 지정**

<br>

**애니메이션 블루프린트를 게임에 적용하는 과정**

1. 애니메이션 블루프린트 애셋 경로 복사

2. 원하는 메시 컴포넌트에 애니메이션 블루프린트 클래스 정보 등록

   * 컴포넌트가 인스턴스를 생성하고 애니메이션을 관리하도록 함
   * 애셋 **경로 후미에 `_C`를 붙여** 클래스 정보를 가져오는 경로 생성

   

<br>

**ABPawn.cpp**

```c++
#include "ABPawn.h"

AABPawn::AABPawn()
{

...

	Mesh->SetAnimationMode(EAnimationMode::AnimationBlueprint);
	
	static ConstructorHelpers::FClassFinder<UAnimInstance> WARRIOR_ANIM(TEXT("/Game/Book/Animations/WarriorAnimBlueprint.WarriorAnimBlueprint_C"));
	if (WARRIOR_ANIM.Succeeded())
	{
		Mesh->SetAnimInstanceClass(WARRIOR_ANIM.Class);
	}
}
```

