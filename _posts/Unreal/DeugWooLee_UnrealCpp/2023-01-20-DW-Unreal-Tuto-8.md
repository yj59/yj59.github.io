---
title: "[이득우의 C++ 언리얼] #8 애니메이션 시스템 활용"

date: 2023-01-20 00:00:00 +0900
author: yejin

categories: [Unreal, 이득우의 c++]
tags: [unreal, tuto, lecture]

math: true
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

# **1. 애니메이션 몽타주**

## **1.1. 몽타주 제작**

과도한 스테이트의 확장은 스테이트 머신 설계를 어지럽힌다. 애니메이션 몽타주 기능을 사용해 스테이트 머신을 확장하지 않고 원하는 애니메이션을 재생시켜보자.

몽타주는 여러 애니메이션 클립들의 일부를 모아 새로운 애니메이션을 만드는 기법으로, 섹션 단위로 애니메이션을 관리한다. 몽타주 편집 윈도우에서 애니메이션을 자르거나 붙이는 작업을 수행할 수 있다.

<br>

**제작 과정**

1.   애니메이션 창 `애셋 생성`->`애님 몽타주`  클릭해 애니메이션 애셋 생성
2.   몽타주에 추가된 섹션 선택*(`Default`->`Attack1`이름 변경)*
3.   공격 애니메이션을 차례대로 몽타주 그룹에 배치
4.   각 애니메이션이 자연스럽게 발생하도록 애니메이션 재생 시간 조절
5.   섹션 저장

<br>

## **1.2. 몽타주 재생**

플레이어가 공격 명령을 입력하면 `Attack1` 섹션이 재생되도록 입력 설정을 추가해 연결해보자.

1.   캐릭터에 `Attack` 입력 처리 함수 추가
2.   애님 인스턴스에 몽타주 애니메이션을 재생하는 멤버 함수 및 변수 생성
     *   몽타주 애셋 레퍼런스 복사 후 멤버 변수에 미리 저장
     *   `Montage_IsPlaying` 함수를 통해 현재 몽타주가 재생하는지 파악
     *   재생 중이 아니라면 `Montage_Play`함수를 사용해 몽타주 재생
3.   몽타주 재생 노드를 애님 그래프에 추가
     *   애님 그래프의 최종 포즈와 스테이트 머신 사이에 몽타주 재생 노드(`DefaultSlot 슬롯`) 추가

>   왜 최종 포즈와 스테이트 머신 사이에 노드를 추가할까?
>
>   모든 상황에서 공격 애니메이션를 재생할 것이므로 스테이트 머신이 최종 포즈가 되기 직전 몽타주가 재생되도록 두 노드 사이에 몽타주 노드를 추가해야 한다.

<br>

**AnimInstance.h**

```c++
public:
	...

	void PlayAttackMontage();

private:
	...

	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = Pawn, Meta = (AllowPrivateAccess = true))
	UAnimMontage* AttackMontage;
```



<br>

**AnimInstance.cpp**

```c++
UABAnimInstance::UABAnimInstance()
{
	...

	static ConstructorHelpers::FObjectFinder<UAnimMontage> ATTACK_MONTAGE(TEXT("/Game/Book/Animations/SK_Mannequin_Skeleton_Montage.SK_Mannequin_Skeleton_Montage"));
	if (ATTACK_MONTAGE.Succeeded())
	{
		AttackMontage = ATTACK_MONTAGE.Object;
	}
}

void UABAnimInstance::PlayAttackMontage()
{
	if (!Montage_IsPlaying(AttackMontage))
	{
		Montage_Play(AttackMontage, 1.0f);
	}
}
```



<br>

**ABCharacter.h**

```c++
private:
	...
	
	void Attack();
```

<br>

**ABCharacter.cpp**

```c++
#include "ABAnimInstance.h"

void AABCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	...
        
	PlayerInputComponent->BindAction(TEXT("Attack"), EInputEvent::IE_Pressed, this, &AABCharacter::Attack);
}

void AABCharacter::Attack()
{
	auto AnimInstance = Cast<UABAnimInstance>(GetMesh()->GetAnimInstance());
	if (nullptr == AnimInstance) return;

	AnimInstance->PlayAttackMontage();
}
```

<br>

![image](https://user-images.githubusercontent.com/93882395/222919369-3dcdf4d8-d289-4376-a0ac-10fe0c78dd14.png) 

>   몽타주 재생 담당 슬롯을 추가한 애님 그래프

<br>

![녹화_2023_03_05_02_24_33_305_AdobeExpress](https://user-images.githubusercontent.com/93882395/222920143-25765e90-6d05-4576-8d3b-e20bd42f334c.gif) 

>   결과 화면

<br>

---

# **2. 델리게이트**

## **2.1. 델리게이트 바인딩**

공격 명령을 내릴 때마다 몽타주 시스템 구동 여부를 확인하는 방식보다 몽타주 재생이 끝나면 폰에게 알려주는 방식이 더 효과적이다. 언리얼 델리게이트 프레임워크를 사용해 공격 기능을 구현해보자.

<br>

**🪄[델리게이트 (Delegate)](https://docs.unrealengine.com/4.27/ko/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Delegates/)**<br>
메서드를 파라미터로서 다른 메서드로 전달해 호출이 가능하게 만들어주는 기능이다(호출자의 인자, 피호출자의 리턴값 등).<br>
C/C++의 함수 포인터를 개선한 형태로, 한 집단을 대표하는 대리인이 나머지를 호출하는 방식!<br>
함수 여러 개를 델리게이트가 관리하고, 호출할 함수가 사라져도 안전하게 프로그램이 동작할 수 있다. 단, 활용 시 가능한 참조 전달으로 작성하자.
{: .notice--primary}

<br>

1.   `OnMontageEnded` 델리게이트에 `UAnimMontage *`, `bool` 인자 등록
     *   몽타주 재생이 끝나는 지점 확인 가능
2.   `ABCharacter` 액터에 함수 형식과 `UFUNCTION` 매크로 선언
3.   `ABCHECK` 매크로 추가
4.   캐릭터 클래스에 함수 선언
5.   `PostInitialize Components`에서 애님 인스턴스의 `OnMontageEnded` 델리게이트에 바인딩

     *   `bool` 변수를 추가 선언해 공격 여부 파악

     *   `OnMontageEnded` 델리게이트와 `OnAttackMontageEnded` 연결
         *   델리게이트 수행 전까지 몽타주 재생 불가

<br>

**ArenaBattle.h**

```c++
#define ABCHECK(Expr, ...) { if(!(Expr)){ ABLOG(Error, TEXT("ASSERTION : %s"), TEXT("'"#Expr"'")); return __VA_ARGS__; } }
```

<br>

**ABCharacter.h**

```c++
class ARENABATTLE_API AABCharacter : public ACharacter
{
	...

public:
	virtual void PostInitializeComponents() override;
	...

private:
	...
        
	UFUNCTION()
	void OnAttackMontageEnded(UAnimMontage* Montage, bool bInterrupted);

private:
	UPROPERTY(VisibleInstanceOnly, BlueprintReadOnly, Category = Attack, Meta = (AllowPrivateAccess = true))
	bool IsAttacking;
};
```

<br>

**ABCharacter.cpp**

```c++
AABCharacter::AABCharacter()
{
	...

	IsAttacking = false;
}

void AABCharacter::PostInitializeComponents()
{
	Super::PostInitializeComponents();
	auto AnimInstance = Cast<UABAnimInstance>(GetMesh()->GetAnimInstance());
	ABCHECK(nullptr != AnimInstance);
	AnimInstance->OnMontageEnded.AddDynamic(this, &AABCharacter::OnAttackMontageEnded);
}

void AABCharacter::Attack()
{
	if (IsAttacking) return;
	...

	AnimInstance->PlayAttackMontage();
	IsAttacking = true;
}

void AABCharacter::OnAttackMontageEnded(UAnimMontage* Montage, bool bInterrupted)
{
	ABCHECK(IsAttacking);
	IsAttacking = false;
}
```

<br>

## **2.2. 멤버 변수 전방 선언**

자주 사용하는 클래스를 멤버 변수로 선언해 런타임에서 활용할 수 있도록 하자. 멤버 변수를 전방 선언으로 작성하면 컴파일 시간을 단축시키고 의존성을 줄일 수 있다.

<br>

**`UABAnimInstance` 클래스의 멤버 변수 선언**

1.   `ABCharacter` 액터에 클래스 멤버 변수 선언
2.   폰 로직에서 입력을 받으면 애님 인스턴스의 `PlayAttack` 호출
3.   `Montage_IsPlaying` 함수 제거

<br>

**ABCharacter.h**

```c++
class ARENABATTLE_API AABCharacter : public ACharacter
{
	...
	
private:
	...

	UPROPERTY(VisibleInstanceOnly, BlueprintReadOnly, Category = Attack, Meta = (AllowPrivateAccess = true))
	class UABAnimInstance* ABAnim;
};
```

<br>

**ABCharacter.cpp**

```c++
void AABCharacter::PostInitializeComponents()
{
	Super::PostInitializeComponents();
	// auto AnimInstance = Cast<UABAnimInstance>(GetMesh()->GetAnimInstance());
	// ABCHECK(nullptr != AnimInstance);
	// AnimInstance->OnMontageEnded.AddDynamic(this, &AABCharacter::OnAttackMontageEnded);
	ABAnim = Cast<UABAnimInstance>(GetMesh()->GetAnimInstance());
	ABCHECK(nullptr != ABAnim);
	ABAnim->OnMontageEnded.AddDynamic(this, &AABCharacter::OnAttackMontageEnded);
}

void AABCharacter::Attack()
{
	if (IsAttacking) return;
	// auto AnimInstance = Cast<UABAnimInstance>(GetMesh()->GetAnimInstance());
	// if (nullptr == AnimInstance) return;
	// AnimInstance->PlayAttackMontage();
	ABAnim->PlayAttackMontage();
	IsAttacking = true;
}
```

<br>

**ABAnimInstance.cpp**

```c++
void UABAnimInstance::PlayAttackMontage()
{
	// if (!Montage_IsPlaying(AttackMontage))
	Montage_Play(AttackMontage, 1.0f);
}
```

<br>

---

# **3. 노티파이**

## **3.1. 노티파이 추가**

[애니메이션 노티파이](https://docs.unrealengine.com/4.27/ko/AnimatingObjects/SkeletalMeshAnimation/Sequences/Notifies/) 애니메이션의 특정 재생 타이밍에 애님 인스턴스로 신호를 보내는 기능이다.

<br>

**공격 판정 타이밍 설정**

1.   몽타주 윈도우에서 캐릭터가 팔을 뻗는 지점으로 타임라인 이동
2.   `노티파이`->`노티파이 추가`->`새 노티파이`
3.   타이밍에 맞게 노티파이 배치
4.   애님 인스턴스에 `UFUNCTION` 매크로 작성 (명명규칙 확인)
     *   노티파이 지점에서 자동으로 애님 인스턴스 클래스 `AnimNotify_노티파이명` 멤버 함수 호출

<br>

**ABAnimInstance.h**

```c++
class ARENABATTLE_API UABAnimInstance : public UAnimInstance
{
	...
        
private:
	UFUNCTION()
	void AnimNotify_AttackHItCheck();
}
```

<br>

**ABAnimInstance.cpp**

```c++
void UABAnimInstance::AnimNotify_AttackHItCheck()
{
	ABLOG_S(Warning);
}
```

<br>

<img src="https://user-images.githubusercontent.com/93882395/222956032-756e7c58-b099-409a-b3d3-f40b01ec6460.png" alt="image" style="zoom:80%;" /> 

>   애니메이션(몽타주) 윈도우

<br>

![image](https://user-images.githubusercontent.com/93882395/222956197-a358aa9b-ddb1-4a85-8f45-46d189f320b5.png) 

>   노티파이 발생 로그 출력

<br>

## **3.2. 콤보 공격 구현**

몽타주의 섹션을 분리해 콤보 공격을 구현할 수 있다.

<br>

**애니메이션 섹션 분리**

1.   섹션 분리 후 섹션별 공격 애니메이션 할당

2.   몽타주 섹션 윈도우에서 섹션 순서 리셋 (섹션 독립 구동)

3.   공격 판정용 노티파이 타이밍 재설정

4.   프레임에 즉각 반응하도록 틱 타입 변경(`Branching Point`)

5.   콤보 동작 과정을 나누어 캐릭터 변수로 선언

     `MaxCombo`, `CurrentCombo`, `CanNextCombo`, `IsComboInputOn`

6.   공격 시작과 종료 시 속성을 지정하기 위해 각각 함수 선언

7.   애님 인스턴스 클래스에서 콤보 카운트마다 몽타주 섹션 재생하도록 구현

     *   `NextAttackCheck` 노티파이 발생 시 캐릭터에 전할 델리게이트 선언
         *   여러 개의 함수 등록을 위해 멀티캐스트로 선언(`Brocast`)

     *   애니메이션 노티파이 함수에서 델리게이트 호출

8.   `CanNextCombo` 속성을 사용해 공격 입력 타이밍 파악

     *   `NextAttackCheck` 이전에 입력을 받으면  해당 타이밍에 다음 콤보 공격 시작

9.   인스턴스에 작성한 델리게이트와 로직을 캐릭터에 선언

     *   C++ 람다식 구문 사용

<br>

<img src="https://user-images.githubusercontent.com/93882395/222957831-8008916e-2909-4cca-ba2c-761a475afb26.png" alt="image" style="zoom:80%;" />  

>   섹션/노티파이(틱 타입) 설정

<br>

**ABCharacter.h**

```c++
class ARENABATTLE_API AABCharacter : public ACharacter
{
private:
	// 6. 공격 시작/종료 함수 선언
	void AttackStartComboState();
	void AttackEndComboState();

private:
	// 4. 콤보 동작 과정 변수
	UPROPERTY(VisibleInstanceOnly, BlueprintReadOnly, Category = Attack, Meta = (AllowPrivateAccess = true))
	bool CanNextCombo;

	UPROPERTY(VisibleInstanceOnly, BlueprintReadOnly, Category = Attack, Meta = (AllowPrivateAccess = true))
	bool IsComboInputOn;

	UPROPERTY(VisibleInstanceOnly, BlueprintReadOnly, Category = Attack, Meta = (AllowPrivateAccess = true))
	int32 CurrentCombo;

	UPROPERTY(VisibleInstanceOnly, BlueprintReadOnly, Category = Attack, Meta = (AllowPrivateAccess = true))
	int32 MaxCombo;
};
```

<br>

**ABCharacter.cpp**

```c++
AABCharacter::AABCharacter()
{
	// 6. 
	MaxCombo = 4;
	AttackEndComboState();
}

void AABCharacter::PostInitializeComponents()
{
	// 9. 델리게이트와 등록할 로직 선언(람다)
	ABAnim->OnNextAttackCheck.AddLambda([this]() -> void {
		ABLOG(Warning, TEXT("OnNextAttackCheck"));
		CanNextCombo = false;

		if (IsComboInputOn)
		{
			AttackStartComboState();
			ABAnim->JumpToAttackMontageSection(CurrentCombo);
		}

	});
}

void AABCharacter::Attack()
{
	// 8. 캐릭터 공격 동작 여부 확인
	if (IsAttacking)
	{
		ABCHECK(FMath::IsWithinInclusive<int32>(CurrentCombo, 1, MaxCombo));
		// 8. 다음 콤보 공격 명령 확인
		if (CanNextCombo)
		{
			IsComboInputOn = true;
		}
	}
	else
	{
		ABCHECK(CurrentCombo == 0);
		AttackStartComboState();
		ABAnim->PlayAttackMontage();
		ABAnim->JumpToAttackMontageSection(CurrentCombo);
		IsAttacking = true;
	}
}

// 8. 추가 공격 명령이 없으면 콤보 공격 종료
void AABCharacter::OnAttackMontageEnded(UAnimMontage* Montage, bool bInterrupted)
{
	ABCHECK(IsAttacking);
	ABCHECK(CurrentCombo > 0);
	IsAttacking = false;
	AttackEndComboState();
}

// 6. 공격 시작/종료 시 속성 재설정 함수 구현
void AABCharacter::AttackStartComboState()
{
	CanNextCombo = true;
	IsComboInputOn = false;
	ABCHECK(FMath::IsWithinInclusive<int32>(CurrentCombo, 0, MaxCombo - 1));
	CurrentCombo = FMath::Clamp<int32>(CurrentCombo + 1, 1, MaxCombo);
}

void AABCharacter::AttackEndComboState()
{
	IsComboInputOn = false;
	CanNextCombo = false;
	CurrentCombo = 0;
}
```

<br>

**ABAnimInstance.cpp**

```c++
// 7. 멀티캐스트 델리게이트 생성
DECLARE_MULTICAST_DELEGATE(FOnNextAttackCheckDelegate);
DECLARE_MULTICAST_DELEGATE(FOnAttackHitCheckDelegate);

class ARENABATTLE_API UABAnimInstance : public UAnimInstance
{
public:
	// 7. 델리게이트와 연결할 몽타주 섹션 함수 선언
	void JumpToAttackMontageSection(int32 NewSection);

// 7. 델리게이트 선언
public:
	FOnNextAttackCheckDelegate OnNextCheck;
	FOnAttackHitCheckDelegate OnAttackHitCheck;

private:
	// 7. 애님 인스턴스가 호출할 노티파이 멤버 함수 선언
	UFUNCTION()
	void AnimNotify_NextAttackCheck();

	// 7. 애니메이션 섹션 반환
	FName GetAttackMontageSectionName(int32 Section);
}
```

<br>

**ABAnimInstance.cpp**

```c++
// 7. 애님 인스턴스 클래스 함수 구현
void UABAnimInstance::JumpToAttackMontageSection(int32 NewSection)
{
	ABCHECK(Montage_IsPlaying(AttackMontage));

	// 7. 애님 인스턴스가 제공하는 함수로, 인자와 같은 이름의 섹션 실행
	Montage_JumpToSection(GetAttackMontageSectionName(NewSection), AttackMontage);
}

// 7. 델리게이트에 추가한 함수 호출
void UABAnimInstance::AnimNotify_AttackHItCheck()
{
	OnAttackHitCheck.Broadcast();
}

void UABAnimInstance::AnimNotify_NextAttackCheck()
{
	OnNextAttackCheck.Broadcast();
}

// 7. 현재 섹션명 반환
FName UABAnimInstance::GetAttackMontageSectionName(int32 Section)
{
	ABCHECK(FMath::IsWithinInclusive<int32>(Section, 1, 4), NAME_None);
	return FName(*FString::Printf(TEXT("Attack%d"), Section));
}
```

<br>

![녹화_2023_03_05_21_23_55_534_AdobeExpress](https://user-images.githubusercontent.com/93882395/222960414-a1428683-5632-493a-90e6-35619c853c17.gif) 

>   결과 화면
>
>   마우스를 빠르게 클릭하면 캐릭터는 다음 콤보 공격을 수행한다.

