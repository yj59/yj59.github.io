---
title: "[이득우의 C++ 언리얼] #6 캐릭터 제작과 컨트롤"

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
last_modified_at: 2023-01-15

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

# 1. 캐릭터 모델

새로 생성한 C++ 클래스 `ABCharactor`는 `ACharacter`를 상속받는다. `ACharacter` 선언을 살펴보면 `ABPawn`과 동일하게 `APawn`을 상속받고 있는 것을 확인할 수 있다.

`ACharacter` 클래스는 `Capsule`, `SkeletalMesh` 컴포넌트와 더불어 `CharacterMovement` 컴포넌트를 사용해 움직임을 관리한다.

<br>

**ACharacter 클래스 제공 함수**: `GetCapsuleComponent`, `GetMesh`, `GetCharacterMovement`

* `private`으로 선언된 컴포넌트의 포인터에 접근 가능케 함

<br>

`ABPawn`과 동일한 액터를 캐릭터로 구현한 후, 게임 모드에서 기본 폰을 `ABCharacter`으로 변경해주자.

**ABCharacter.h** 

```c++
#pragma once

#include "ArenaBattle.h"
#include "GameFramework/Character.h"
#include "ABCharacter.generated.h"

UCLASS()
class ARENABATTLE_API AABCharacter : public ACharacter
{
...

	UPROPERTY(VisibleAnywhere, Category = Camera)
		USpringArmComponent* SpringArm;

	UPROPERTY(VisibleAnywhere, Category = Camera)
		UCameraComponent* Camera;

private:
	void UpDown(float NewAxisValue);
	void LeftRight(float NewAxisValue);
};
```

<br>

**ABCharacter.cpp**

```c++
#include "ABCharacter.h"

// Sets default values
AABCharacter::AABCharacter()
{
 	// Set this character to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;

	SpringArm = CreateDefaultSubobject<USpringArmComponent>(TEXT("SPRINGARM"));
	Camera = CreateDefaultSubobject<UCameraComponent>(TEXT("CAMERA"));

	// ACharacter가 제공하는 함수로 인자 수정. Capsule -> GetCapsuleComponent()
	SpringArm->SetupAttachment(GetCapsuleComponent());
	Camera->SetupAttachment(SpringArm);

	// Mesh -> GetMesh()
	GetMesh()->SetRelativeLocationAndRotation(FVector(0.0f, 0.0f, -88.0f), FRotator(0.0f, -90.0f, 0.0f));
	SpringArm->TargetArmLength = 400.0f;
	SpringArm->SetRelativeRotation(FRotator(-15.0f, 0.0f, 0.0f));

	static ConstructorHelpers::FObjectFinder<USkeletalMesh> SK_CARDBOARD(TEXT("/Game/InfinityBladeWarriors/Character/CompleteCharacters/SK_CharM_Cardboard.SK_CharM_Cardboard"));
	if (SK_CARDBOARD.Succeeded())
	{
		GetMesh()->SetSkeletalMesh(SK_CARDBOARD.Object);
	}

	GetMesh()->SetAnimationMode(EAnimationMode::AnimationBlueprint);
	static ConstructorHelpers::FClassFinder<UAnimInstance> WARRIOR_ANIM(TEXT("/Game/Book/Animations/WarriorAnimBlueprint.WarriorAnimBlueprint_C"));
	if (WARRIOR_ANIM.Succeeded())
	{
		GetMesh()->SetAnimInstanceClass(WARRIOR_ANIM.Class);
	}
}

...

// Called to bind functionality to input
void AABCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

	PlayerInputComponent->BindAxis(TEXT("UpDown"), this, &AABCharacter::UpDown);
	PlayerInputComponent->BindAxis(TEXT("LeftRight"), this, &AABCharacter::LeftRight);
}

void AABCharacter::UpDown(float NewAxisValue) 
{
	AddMovementInput(GetActorForwardVector(), NewAxisValue);
}

void AABCharacter::LeftRight(float NewAxisValue) 
{
	AddMovementInput(GetActorRightVector(), NewAxisValue);
}
```

<br>

**ABGameMode.cpp**

```c++
#include "ABGameMode.h"
// #include "ABPawn.h"
#include "ABCharacter.h"
#include "ABPlayerController.h"

AABGameMode::AABGameMode()
{
    // 기본 폰을 ABCharacter로 설정
	DefaultPawnClass = AABCharacter::StaticClass();
	PlayerControllerClass = AABPlayerController::StaticClass();
}
```

<br>

**`FloatingPawnMovement와 비교한 캐릭터 무브먼트 컴포넌트의 장점`**

1. 중력을 반영한 움직임 제공 *(점프 등)*
2. 다양한 움직임(이동 모드) 설정 가능, 비교적 더 많은 정보 전달
3. 멀티 플레이 네트워크 환경에서 움직임 자동 동기화

<br>

---

## 2. 컨트롤 회전

플레이어 컨트롤러는 플레이어의 의지를 나타내는 `컨트롤 회전`이라는 속성을 제공한다.

마우스 움직임에 따라 폰이 일정 속도로 회전하는 기능을 제작해보자. 바인딩 탭의 `Turn`과 `LookUp`의 축 입력 설정으로 마우스 움직임을 받아올 수 있다.

(-3 ~ 3)

입력 값에 따라 캐릭터가 회전하도록 `AddControllerInputYaw`, `Roll`, `Pitch`  명령어를 제공한다.

<br>

**ABCharacter.h**

```c++
...
private:
	void UpDown(float NewAxisValue);
	void LeftRight(float NewAxisValue);
	void LookUp(float NewAxisValue);
	void Turn(float NewAxisValue);
};
```

<br>

**ABCharacter.cpp**

```c++
void AABCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
    
...
    
	PlayerInputComponent->BindAxis(TEXT("LookUp"), this, &AABCharacter::LookUp);
	PlayerInputComponent->BindAxis(TEXT("Turn"), this, &AABCharacter::Turn);
}

...
    
void AABCharacter::LookUp(float NewAxisValue)
{
	AddControllerPitchInput(NewAxisValue);
}

void AABCharacter::Turn(float NewAxisValue)
{
	AddControllerYawInput(NewAxisValue);
}
```

* 
