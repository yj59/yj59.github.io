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

# **1. 캐릭터 모델**

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

AABCharacter::AABCharacter()
{
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

# **2. 컨트롤 회전**

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

<br>

![녹화_2023_03_03_12_32_22_968_AdobeExpress](https://user-images.githubusercontent.com/93882395/222625882-fa8a929b-22e0-49c7-9788-372ee8715cb5.gif) 

> 결과 화면
>
> 마우스 좌우 이동에 따라 캐릭터와 카메라 축이 동시에 회전한다.

`~` 키를 눌러 콘솔 명령어 입력 창을 띄우고 플레이어 컨트롤러의 `ControlRotation` 속성 값을 확인해보자. `Shift+F1`키로 뷰포트 포커스를 해제할 수 있다.

*   콘솔 명령어: `DisplayAll PlayerController ControlRotation`

<br>

![image](https://user-images.githubusercontent.com/93882395/222607585-e73146b9-2fd4-40a6-9af1-9daa3a9a0127.png) 

`ABCharacter`의 폰 섹션을 확인하면 컨트롤 회전의 Yaw 회전 값과 폰의 Yaw값이 연동되어 있음을 알 수 있다.

![image](https://user-images.githubusercontent.com/93882395/222608367-57eff5f6-603b-4849-98f5-246a45256859.png) 

플레이어 컨트롤러에 내장된 속성 값을 변경하면 컨트롤 회전 값 정도가 달라진다.

<br>

---

# **3. 삼인칭 컨트롤 - GTA**

언리얼에서 제공하는 흰색 마네킹과 동일한 기능을 구현할 수 있다.

**흰색 마네킹 설정**

- **캐릭터의 이동**: 현재 보는 시점을 기준으로 상하, 좌우 방향으로 마네킹 이동(카메라 회전 x)
- **캐릭터의 회전**: 캐릭터가 이동하는 방향으로 마네킹 회전
- **카메라 지지대 길이**: 450cm
- **카메라 회전**: 마우스 상하좌우 이동에 대응해 카메라 지지대 회전
- **카메라 줌**: 카메라 시선과 캐릭터 사이에 장애물이 감지되면 캐릭터가 보이도록 카메라를 장애물 앞으로 줌인

<br>

`ABCharacter` 클래스에 `SetControlMode` 멤버 함수를 만들어 `SpringArm` 컴포넌트를 활용한 카메라 설정을 구현해보자.

**ABCharacter.h**

```c++
...

protected:
	void SetControlMode(int32 ControlMode);
	
...
```

<br>

**ABCharacter.cpp**

```c++
AABCharacter::AABCharacter()
{
    
	...
    
	SetControlMode(0);
}

...

void AABCharacter::SetControlMode(int32 ControlMode)
{
	if (ControlMode == 0)
	{
		SpringArm->TargetArmLength = 450.0f;
		SpringArm->SetRelativeRotation(FRotator::ZeroRotator);
		SpringArm->bUsePawnControlRotation = true;
		SpringArm->bInheritPitch = true;
		SpringArm->bInheritYaw = true;
		SpringArm->bInheritRoll = true;
		SpringArm->bDoCollisionTest = true;
		bUseControllerRotationYaw = false;
	}
}
```

<br>

![녹화_2023_03_03_12_33_15_696_AdobeExpress](https://user-images.githubusercontent.com/93882395/222626195-dd788f6f-1999-408b-89a4-bbf8d5ca076c.gif) 

> 결과 화면
>
> 캐릭터는 회전하지 않지만 카메라 지지대가 마우스 움직임에 따라 회전한다.

<br>

캐릭터가 카메라 방향을 중심으로 움직이도록 이동 방향을 변경해주어야 한다.  회전 값인 `FRotator` 데이터에서 방향 값 `FVector`데이터를 얻을 수 있다.

> 액터의 회전 값 (0, 0, 0)은 그 액터가 바라보는 월드의 X축 방향 (1, 0, 0)임을 의미한다. 액터가 회전하면 액터의 시선 방향도 자연스럽게 다른 값으로 변하게 된다.

카메라가 바라보는 방향인 컨트롤 회전 값으로부터 회전 행렬을 생성하고 원하는 방향 축을 대입해 캐릭터가 움직일 방향을 가져올 수 있다.

*(X축: 시선 방향, Y축: 우측 방향)*

<br>

캐릭터가 움직일 방향만 가져오게 되면, X축만 바라보며 시선 방향으로 이동하므로 부자연스러운 결과물이 나온다..

`bOrientRotationToMovement` 기능을 사용해 캐릭터가 움직이는 방향을 따라 자동으로 회전시켜주도록 한다.

<br>

**ABCharacter.cpp**

```c++
void AABCharacter::SetControlMode(int32 ControlMode)
{
	if (ControlMode == 0)
	{
		...
            
		// 캐릭터가 움직이는 방향으로 캐릭터 자동 회전
		GetCharacterMovement()->bOrientRotationToMovement = true;
		// 회전 속도를 지정해 캐릭터가 부드럽게 회전하도록 함
		GetCharacterMovement()->RotationRate = FRotator(0.0f, 720.0f, 0.0f);
	}
}

...
    
void AABCharacter::UpDown(float NewAxisValue) 
{
	// 회전 값으로부터 시선 방향(X)과 우측 방향(Y)의 벡터값 가져옴
	// AddMovementInput(GetActorForwardVector(), NewAxisValue);
	AddMovementInput(FRotationMatrix(GetControlRotation()).GetUnitAxis(EAxis::X), NewAxisValue);
}

void AABCharacter::LeftRight(float NewAxisValue) 
{
	// AddMovementInput(GetActorRightVector(), NewAxisValue);
	AddMovementInput(FRotationMatrix(GetControlRotation()).GetUnitAxis(EAxis::Y), NewAxisValue);
}
```

<br>

![녹화_2023_03_03_13_00_57_562_AdobeExpress](https://user-images.githubusercontent.com/93882395/222628488-74c5516a-e46e-49bd-96cc-a80da8852e9f.gif) 

> 결과 화면

<br>

---

# **4. 삼인칭 컨트롤 - 디아블로**

고정된 삼인칭 시점에서 캐릭터를 따라다니는 컨트롤을 구현해보자.

- **캐릭터의 이동** : 상하좌우 키를 조합해 캐릭터가 이동할 방향 결정
- **캐릭터의 회전** : 캐릭터는 입력한 방향으로 회전
- **카메라 지지대 길이** : 800cm
- **카메라 회전** : 항상 시선 고정 (45도)
- **카메라 줌** : 없음
  - 카메라와 캐릭터 사이에 장애물이 있는 경우 외곽선 처리

<br>

**컨트롤 회전 값 비교**

* GTA: `SpringArm`의 회전에 사용 (카메라와 캐릭터가 동시에 회전)
* 디아블로: 캐릭터의 방향에 사용 (카메라 고정)

<br>

## **4.1. 컨트롤 설정 변경**

![image](https://user-images.githubusercontent.com/93882395/222707530-65a271e3-7153-4314-8875-bb7d305745dd.png) 

`ViewChange`라는 액션 매핑을 추가하고 `Shift+V` 키를 누를 때마다 `SetControlMode`함수에 다른 인자값이 들어가도록 코드를 작성하자.  `SetupPlayerInputComponent` 멤버함수에 `BindAction` 내부 함수를 선언해 입력 키가 눌렸을 때 `ViewChange` 함수를 호출하도록 한다.

* `BindAction` : 버튼이 눌렸는지, 떼어졌는지에 대한 부가 인자 지정 가능

    * `EInputEvent::IE_Pressed` : 버튼을 누른 직후
    * `EInputEvent::IE_Released` : 버튼을 떼었을 때

* `InterpTo` : 지정 속력으로 목표 지점까지 이동(`FMath`에서 제공)

    * `float`형: `FInterpTo`

    * `Vector`형: `VInterpTo`

    * `Rotator`형: `RInterpTo`

        *(eg. 회전 보간 : `FMath::RInterpTo(이전 회전값, 목표 회전값, DeltaTime, 회전 속도)`)*

<br>

**ABCharacter.h**

```c++
protected:
	enum class EControlMode
	{
		GTA,
		DIABLO
	};

	void SetControlMode(EControlMode NewControlMode);
	EControlMode CurrentControlMode = EControlMode::GTA;
	
	// UPROPERTY를 사용하지 않는 값 타입 변수들은 초기값 미리 지정
	FVector DirectionToMove = FVector::ZeroVector;

	// ViewChange 키 입력시 시점 부드럽게 전환
	// SetControlMode 함수에서 정의 후 Tick 함수의 인자로 쓰임
	float ArmLengthTo = 0.0f;
	FRotator ArmRotationTo = FRotator::ZeroRotator;
	float ArmLengthSpeed = 0.0f;
	float ArmRotationSpeed = 0.0f;

private:
	void ViewChange();
```

<br>

**ABCharacter.cpp**

```c++
AABCharacter::AABCharacter()
{
	...
	
	SetControlMode(EControlMode::DIABLO);
	
	ArmLengthSpeed = 3.0f;
	ArmRotationSpeed = 10.0f;
}

void AABCharacter::SetControlMode(EControlMode NewControlMode)
{
	// 새 컨트롤 모드에 맞게 모드 설정값 변경
	CurrentControlMode = NewControlMode;

	switch (CurrentControlMode)
	{
	case EControlMode::GTA:
		// 카메라 지지대 길이 450cm
		// SpringArm->TargetArmLength = 450.0f;
		// SpringArm->SetRelativeRotation(FRotator::ZeroRotator);
		
		ArmLengthTo = 450.0f;
		SpringArm->bUsePawnControlRotation = true;
		SpringArm->bInheritPitch = true;
		SpringArm->bInheritRoll = true;
		SpringArm->bInheritYaw = true;
		SpringArm->bDoCollisionTest = true;
		bUseControllerRotationYaw = false;
		GetCharacterMovement()->bOrientRotationToMovement = true;
		GetCharacterMovement()->bUseControllerDesiredRotation = false;
		GetCharacterMovement()->RotationRate = FRotator(0.0f, 720.0f, 0.0f);
		break;
	case EControlMode::DIABLO:
		// 카메라 길이 800, 45도에서 시점 고정
		// 마우스 입력 전부 해제(키보드 입력만 받음)
		// SpringArm->TargetArmLength = 800.0f;
		// SpringArm->SetRelativeRotation(FRotator(-45.0f, 0.0f, 0.0f));
		
		ArmLengthTo = 800.0f;
		ArmRotationTo = FRotator(-45.0f, 0.0f, 0.0f);
		
		SpringArm->bUsePawnControlRotation = false;
		SpringArm->bInheritPitch = false;
		SpringArm->bInheritRoll = false;
		SpringArm->bInheritYaw = false;
		SpringArm->bDoCollisionTest = false;
		
		// 회전 끊김 방지 => 속성 해제 후 회전 보완 코드 작성
		// bOrientRotationToMovement 로 대체됨(UE4.26이상)
		bUseControllerRotationYaw = false;
		
		// 캐릭터 자동 회전 (키보드로 캐릭터 회전시키므로 해제)
		GetCharacterMovement()->bOrientRotationToMovement = false;
		
		// 컨트롤 회전이 가리키는 방향으로 캐릭터 회전
		GetCharacterMovement()->bUseControllerDesiredRotation = true;
		
		// 캐릭터가 부드럽게 회전하도록 보완 (회전 속도 지정)
		GetCharacterMovement()->RotationRate = FRotator(0.0f, 720.0f, 0.0f);
		break;
	}
}

void AABCharacter::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	SpringArm->TargetArmLength = FMath::FInterpTo(SpringArm->TargetArmLength, ArmLengthTo, DeltaTime, ArmLengthSpeed);

	switch (CurrentControlMode)
	{
	case EControlMode::DIABLO:
		// SpringArm의 길이와 회전값이 목표 지점까지 변경
		// 변경 속도: ArmRotationSpeed
		SpringArm->GetRelativeRotation() = FMath::RInterpTo(SpringArm->GetRelativeRotation(), ArmRotationTo, DeltaTime, ArmRotationSpeed);
		break;
	}

	switch (CurrentControlMode)
	{
	case EControlMode::DIABLO:
		if (DirectionToMove.SizeSquared() > 0.0f)
		{
			GetController()->SetControlRotation(FRotationMatrix::MakeFromX(DirectionToMove).Rotator());
			AddMovementInput(DirectionToMove);
		}
		break;
	}
}

void AABCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

	// 버튼을 누른 직후 ViewChange함수 호출
	PlayerInputComponent->BindAction(TEXT("ViewChange"), EInputEvent::IE_Pressed, this, &AABCharacter::ViewChange);
    ...
}

void AABCharacter::UpDown(float NewAxisValue)
{
	switch (CurrentControlMode)
	{
	case EControlMode::GTA:
		AddMovementInput(FRotationMatrix(FRotator(0.0f, GetControlRotation().Yaw, 0.0f)).GetUnitAxis(EAxis::X), NewAxisValue);
		break;
	case EControlMode::DIABLO:
		DirectionToMove.X = NewAxisValue;
		break;
	}
}

void AABCharacter::LeftRight(float NewAxisValue)
{

	switch (CurrentControlMode)
	{
	case EControlMode::GTA:
		AddMovementInput(FRotationMatrix(FRotator(0.0f, GetControlRotation().Yaw, 0.0f)).GetUnitAxis(EAxis::Y), NewAxisValue);
		break;
	case EControlMode::DIABLO:
		DirectionToMove.Y = NewAxisValue;
		break;
	}
}

void AABCharacter::LookUp(float NewAxisValue)
{
	switch (CurrentControlMode)
	{
	case EControlMode::GTA:
		AddControllerPitchInput(NewAxisValue);
		break;
	}
}

void AABCharacter::Turn(float NewAxisValue)
{
	switch (CurrentControlMode)
	{
	case EControlMode::GTA:
		AddControllerYawInput(NewAxisValue);
		break;
	}
}

// GTA <-> DIABLO 시점 변경 함수
void AABCharacter::ViewChange()
{
	switch (CurrentControlMode)
	{
	case EControlMode::GTA:
		// 시점 변경이 자연스럽도록 회전 값 미리 부여
		// GTA->DIABLO이므로 캐릭터의 회전값 지정
		GetController()->SetControlRotation(GetActorRotation());
		SetControlMode(EControlMode::DIABLO);
		break;
	case EControlMode::DIABLO:
		// DIABLO->GTA이므로 SpringArm 회전값 지정
		// UE4.24이상부터 RelativeRotation->GetRelativeRotation()으로 변경
		GetController()->SetControlRotation(SpringArm->GetRelativeRotation());
		SetControlMode(EControlMode::GTA);
		break;
	}
}
```



<br>

![녹화_2023_03_03_13_38_07_303_AdobeExpress](https://user-images.githubusercontent.com/93882395/222632931-c3c0cb09-104d-46f8-a9db-80eacbe1f1d5.gif) 

>   결과 화면
>
>   `Shift+V`키를 누르면 부드럽게 시점이 변경되는 모습을 확인할 수 있다!
