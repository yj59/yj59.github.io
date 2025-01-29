---
title: "[이득우의 C++ 언리얼] #9 물리 엔진 설정"

date: 2023-01-22 00:00:00 +0900
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

# **1. 콜리전 설정**

물리 엔진을 활용하기 위해 콜리전으로 충돌 영역을 설정하고 어떻게 활용할지 설계해야 한다. 스태틱메시 애셋은 `BlockAll` 설정이 있으나 액터 간 자세한 상호작용을 위해 물리 설정을 알아두자.

<br>

**콜리전 제작 종류(충돌 영역 설정)**

*   **스태틱메시 애셋**: 스태틱메시 에디터에 콜리전 영역 설정
    *   스태틱메시 컴포넌트에서 비주얼/충돌 기능 설정 가능
*   **기본 도형 컴포넌트**: 기본 도형을 사용해 충돌 영역 지정
    *   스켈레탈 메시를 움직일 때 주로 사용
*   **피직스 애셋**: 스켈레탈 메시에만 사용 가능

<br>

**물리 설정**

1.   콜리전 채널과 기본 반응
2.   콜리전 채널의 용도
3.   다른 콜리전 채널과의 반응

<br>

충돌체에는 반드시 하나의 콜리전 채널을 설정해야 한다.  예를 들어, 캐릭터의 루트 컴포넌트인 캡슐 컴포넌트에는 `Pawn` 콜리전 채널이 설정된다.

**언리얼 콜리전 채널**

*   `WorldStatic` : 정적 배경 액터에 사용 (스태틱메시 컴포넌트)
*   `WorldDynamic` : 움직이는 액터에 사용 (스태틱메시 컴포넌트)
*   `Pawn` : 플레이어가 조종하는 물체에 주로 사용 (캡슐 컴포넌트<-*캐릭터 충돌*)
*   `Visibility` : 피킹 기능 구현 시 사용
    *   배경 물체의 가시성 탐지
*   `Camera` : 카메라와 목표물 간 장애물이 있는지 탐지
*   `PisicsBody` : 물리 시뮬레이션으로 움직이는 컴포넌트에 설정

<br>

![image](https://user-images.githubusercontent.com/93882395/222999201-35baf621-0b7f-454a-bf48-1401fd84999d.png) 

>   `ABCharacter`의 콜리전 채널이 `Pawn`으로 설정되어 있다.
>
>   콜리전 프리셋의 `Pawn`과 Object Type의 `Pawn`을 헷갈리지 않도록 조심하자!

<br>

`Collision Enabled` 항목에서 물리 기능을 어떻게 사용할지 설정할 수 있다. `Query and Pysics` 기능을 사용하면 물리 엔진의 계산량이 늘어나므로 액터에 필요한 설정만 적용하는 것이 효과적이다.

*   `Query` : 두 물체이 충돌 영역이 서로 겹치는지 테스트하는 설정
    *   `BeginOverlap` : 충돌 영역이 겹치면 해당 이벤트 발생
    *   `Raycast`, `Sweep` : 물체 충돌 탐지
*   `Pysics` : 물리적인 시뮬레이션 사용
*   `Query and Pysics`

<br>

콜리전 채널이 다른 컴포넌트의 콜리전 채널과 어떻게 반응할지 지정하는 작업이 필요하다.

*   **무시(Ignore)**: 아무 충돌이 일어나지 않음
*   **겹침(Overlap)**: 통과가 가능하되, 이벤트 발생 (`BeginOverlap`)
*   **블록(Block)**: 통과 불가능 (`Hit`)

두 물체가 반응 값이 다를 경우, 무시를 우선시하고 블록을 가장 낮은 우선순위로 둔다.

`Hit` 이벤트과 `BeginOverlap` 이벤트를 모두 발생시키려면 `Generate Overlap Events` 항목이 양쪽 컴포넌트에 모두 체크되어 있어야 한다.

<br>

구체적인 게임 구현을 위해 새로운 콜리전 채널을 만들 수 있다.

캐릭터에 `Pawn`이 아닌 새로운 채널을 만들어 지정해보자. 콜리전 채널은 크게 두 가지 영역으로 나뉜다.

*   **오브젝트 채널**: 콜리전 영역에 지정 (나머지 채널)
*   **트레이스 채널**: 어떤 행동에 설정(`Visibility`, `Camera`)

<img src="https://user-images.githubusercontent.com/93882395/223000846-f08c8dfb-666b-44ff-abe9-2387dc49e6db.png" alt="image" style="zoom:80%;" /> 

>   프로젝트 설정 윈도우에서 새 콜리전 채널을 추가할 수 있다.

<br>

기본 반응을 블록으로 지정했지만, 블록 반응을 하면 안 되는 다른 콜리전 프리셋이 있다. 다른 콜리전 프리셋과 반응이 문제되지 않도록 프리셋을 조율하자.

콜리전 세팅의 `Preset` 섹션에서 사용 가능한 프리셋 목록을 볼 수 있으며, 새 프리셋을 생성할 수도 있다.

`ABCharacter` 프리셋을 추가하고 블록 반응을 하면 안 되는 콜리전 프리셋을 찾아 설정을 변경해두자. 예를 들어, `Trigger` 프리셋은 플레이어의 진로를 방해하지 말아야 하는 속성이므로 `ABCharacter` 채널과의 반응을 겹침으로 바꾸어준다.

<img src="https://user-images.githubusercontent.com/93882395/223011190-e70b0f34-c9ff-4b78-b973-963625d8ff27.png" alt="image" style="zoom:80%;" /> 

>   프리셋 목록

<br>

**주의해야 할 프리셋**

*   `OverlapAll` : 겹침 설정
*   `OverlapAllDynamic` : 겹침 설정
*   `IgnoreOnlyPawn` : 폰 충돌 무시. 무시 설정
*   `OverlapOnlyPawn` : 폰 겹침 이벤트 발생. 겹침 설정
*   `Spector` : 외부 관중과의 충돌. 무시 설정
*   `CharacterMesh` : 캐릭터 메시에 사용. 무시 설정
*   `RagDoll` : 스켈레탈 메시의 피직스 애셋 물리 가동. 무시 설정
*   `Trigger` : 영역별 이벤트 발동. 겹침 설정
*   `UI` : UI요소에 사용. 겹침 설정

<br>

캡슐 컴포넌트가 `ABCharacter` 프리셋을 사용하도록 프리셋 기본값 코드를 작성한다.

<br>

**ABCharacter.cpp**

```c++
AABCharacter::AABCharacter()
{
	GetCapsuleComponent()->SetCollisionProfileName(TEXT("ABCharacter"));
}
```

<br>

<img src="https://user-images.githubusercontent.com/93882395/223012729-173b4ad0-3538-4d7e-8ac6-d4d08ee864b3.png" alt="image" style="zoom:80%;" /> 

>   프리셋 기본값 변경 결과

<br>

---

# **2. 트레이스 채널 활용**

## **2.1. 트레이스 채널 개념**

어떠한 행동에 대한 판정을 위해 트레이스 채널을 활용할 수 있다.

캐릭터 공격 판정 역할을 수행하는 트레이스 채널을 무시 속성으로 추가하고 `ABCharacter` 채널에만 블록으로 설정한다.

<br>

트레이스 채널을 사용한 `SweepSingleByChannel` 함수가 물리적 충돌 여부를 판정한다. 물리는 월드의 기능이므로 `GetWorld()` 함수를 사용한다.

**`SweepSingleByChannel` 함수 인자**

*   `HitResult` : 물리적 충돌이 탐지된 경우 관련 정보를 저장할 구조체
*   `Start` : 탐색 시작 위치
*   `End` : 탐색 종료 위치
*   `Rot` : 탐색에 사용할 도형 회전
*   `TraceChannel` : 물리 충돌 감지에 사용할 트레이스 채널 정보
*   `CollisionShape` : 탐색에 사용할 기본 도형 정보
*   `Params` : 탐색 방법에 대한 설정 값을 모아둔 구조체
*   `ResponseParams` : 탐색 반응을 설정하기 위한 구조체

<br>

코드에 사용할 채널 값은 언리얼 엔진에서 정의한 `ECollisionChannel` 열거형으로 가져올 수 있다. 언리얼 엔진은 게임에서 활용할 수 있도록 총 32개의 콜리전 채널을 제공하며, 기본으로 사용되는 채널을 제외하면 게임 프로젝트 전용으로 18개를 사용할 수 있다.

<br>

**EngineTypes.h**

```c++
UENUM(BlueprintType)
enum ECollisionChannel
{

	ECC_WorldStatic UMETA(DisplayName="WorldStatic"),
	ECC_WorldDynamic UMETA(DisplayName="WorldDynamic"),
	ECC_Pawn UMETA(DisplayName="Pawn"),
	ECC_Visibility UMETA(DisplayName="Visibility" , TraceQuery="1"),
	ECC_Camera UMETA(DisplayName="Camera" , TraceQuery="1"),
	ECC_PhysicsBody UMETA(DisplayName="PhysicsBody"),
	ECC_Vehicle UMETA(DisplayName="Vehicle"),
	ECC_Destructible UMETA(DisplayName="Destructible"),

	/** Reserved for gizmo collision */
	ECC_EngineTraceChannel1 UMETA(Hidden),

	ECC_EngineTraceChannel2 UMETA(Hidden),
	ECC_EngineTraceChannel3 UMETA(Hidden),
	ECC_EngineTraceChannel4 UMETA(Hidden), 
	ECC_EngineTraceChannel5 UMETA(Hidden),
	ECC_EngineTraceChannel6 UMETA(Hidden),

	ECC_GameTraceChannel1 UMETA(Hidden),
	ECC_GameTraceChannel2 UMETA(Hidden),
	ECC_GameTraceChannel3 UMETA(Hidden),
	ECC_GameTraceChannel4 UMETA(Hidden),
	ECC_GameTraceChannel5 UMETA(Hidden),
	ECC_GameTraceChannel6 UMETA(Hidden),
	ECC_GameTraceChannel7 UMETA(Hidden),
	ECC_GameTraceChannel8 UMETA(Hidden),
	ECC_GameTraceChannel9 UMETA(Hidden),
	ECC_GameTraceChannel10 UMETA(Hidden),
	ECC_GameTraceChannel11 UMETA(Hidden),
	ECC_GameTraceChannel12 UMETA(Hidden),
	ECC_GameTraceChannel13 UMETA(Hidden),
	ECC_GameTraceChannel14 UMETA(Hidden),
	ECC_GameTraceChannel15 UMETA(Hidden),
	ECC_GameTraceChannel16 UMETA(Hidden),
	ECC_GameTraceChannel17 UMETA(Hidden),
	ECC_GameTraceChannel18 UMETA(Hidden),
	
	...
        
	ECC_MAX,
};
```

<br>

우리가 생성한 콜리전이 어떤 채널값을 배정받았는지 확인하려면 `Config` 폴더의 `DefaultEngine.ini`를 확인해보자.

<br>

**DefaultEngine.ini**

```c++
+DefaultChannelResponses=(Channel=ECC_GameTraceChannel1,DefaultResponse=ECR_Block,bTraceType=False,bStaticObject=False,Name="ABCharacter")
+DefaultChannelResponses=(Channel=ECC_GameTraceChannel2,DefaultResponse=ECR_Ignore,bTraceType=True,bStaticObject=False,Name="Attack")
```

<br>

## **2.2. 공격 판정 구현**

1.   `SweepSingleByChannel` 인자 작성

     *   액터의 충돌이 감지된 경우 구조체 반환
         *   `FHitResult` 구조체 생성 후 인자값으로 입력
     *   캐릭터 위치에서 정면 방향으로 2m 떨어진 곳까지 탐색
     *   콜리전 채널 지정(`Channel=ECC_GameTraceChannel2`)

     *   공격 범위를 판정하기 위해 반지름이 50cm인 구 생성 (`FCollisionShape::MakeSphere`)

     *   탐색 방법 설정
         *   탐색 반응 설정으로 구조체 기본값 사용

2.   자신은 탐색에 감지되지 않도록 `this` 포인터를 무시 액터 목록에 기재

<br>

**ABCharacter.h**

```c++
UCLASS()
class ARENABATTLE_API AABCharacter : public ACharacter
{
...
    
private:
	void AttackCheck();
    
...
}
```



<br>

**ABCharacter.cpp**

```c++
void AABCharacter::PostInitializeComponents()
{
    ...
        
	// OnAttackHitCheck 델리게이트에 AttackCheck 함수 등록
	ABAnim->OnAttackHitCheck.AddUObject(this, &AABCharacter::AttackCheck);
}

void AABCharacter::AttackCheck()
{
	FHitResult HitResult;
	FCollisionQueryParams Params(NAME_None, false, this);
	bool bResult = GetWorld()->SweepSingleByChannel(
		HitResult,
		GetActorLocation(),
		GetActorLocation() + GetActorForwardVector() * 200.0f,
		FQuat::Identity,
		ECollisionChannel::ECC_GameTraceChannel2,
		FCollisionShape::MakeSphere(50.0f),
		Params);

	if (bResult)
	{
		if (HitResult.Actor.IsValid())
		{
			ABLOG(Warning, TEXT("Hit Actor Name : %s"), *HitResult.Actor->GetName());
		}
	}
}
```

<br>

![녹화_2023_03_06_15_27_03_587_AdobeExpress](https://user-images.githubusercontent.com/93882395/223035372-840ad460-7c78-4d2b-8631-99bcc9a661b6.gif) 

>   결과 화면
>
>   타격 시 공격 판정 로그 출력

<br>

---

# **3. 디버그 드로잉**

디버그 드로잉을 활용해 공격 범위를 시각적으로 확인할 수 있다. 캐릭터 소스코드 상단에 `DrawDebugHelpers.h` 헤더를 추가하고 공격 탐색 궤적을 구현해보자.

<br>

**DrawDebugHelpers.h**

```c++
...
FORCEINLINE void DrawDebugCapsule(const UWorld* InWorld, FVector const& Center, float HalfHeight, float Radius, const FQuat& Rotation, FColor const& Color, bool bPersistentLines = false, float LifeTime = -1.f, uint8 DepthPriority = 0, float Thickness = 0) {}
...
```

`DrawDebugCapsule` 함수를 사용해 탐색을 위해서 원이 움직인 궤적을 표현해보자. 

**원하는 모양의 캡슐 모양 구하기**

1.   캡슐의 반지름을 50으로 설정
2.   탐색 시작 위치에서 탐색 끝 위치로 향하는 벡터 계산
3.   벡터의 중점 위치와 벡터 길이의 절반 대입
4.   회전 행렬을 적용해 캡슐 방향을 수평으로 변경
5.   공격 범위에 맞게 캡슐 길이 조정

<br>

**ABCharacter.h**

```c++
UCLASS()
class ARENABATTLE_API AABCharacter : public ACharacter
{
	
	...

	UPROPERTY(VisibleInstanceOnly, BlueprintReadOnly, Category = Attack, Meta = (AllowPrivateAccess = true))
	float AttackRange;

	UPROPERTY(VisibleInstanceOnly, BlueprintReadOnly, Category = Attack, Meta = (AllowPrivateAccess = true))
	float AttackRadius;
};
```

<br>

**ABCharacter.cpp**

```c++
#include "DrawDebugHelpers.h"

AABCharacter::AABCharacter()
{
	...

	AttackRange = 200.0f;
	AttackRadius = 50.0f;
}

void AABCharacter::AttackCheck()
{
	FHitResult HitResult;
	FCollisionQueryParams Params(NAME_None, false, this);
	bool bResult = GetWorld()->SweepSingleByChannel(
		HitResult,
		GetActorLocation(),
		GetActorLocation() + GetActorForwardVector() * 200.0f,
		FQuat::Identity,
		ECollisionChannel::ECC_GameTraceChannel2,
		FCollisionShape::MakeSphere(50.0f),
		Params);

#if ENABLE_DRAW_DEBUG

	FVector TraceVec = GetActorForwardVector() * AttackRange;
	FVector Center = GetActorLocation() + TraceVec * 0.5f;
	float HalfHeight = AttackRange * 0.5f + AttackRadius;
	FQuat CapsuleRot = FRotationMatrix::MakeFromZ(TraceVec).ToQuat();
	FColor DrawColor = bResult ? FColor::Green : FColor::Red;
	float DebugLifeTime = 5.0f;

	DrawDebugCapsule(GetWorld(),
		Center,
		HalfHeight,
		AttackRadius,
		CapsuleRot,
		DrawColor,
		false,
		DebugLifeTime);

#endif

	if (bResult)
	{
		if (HitResult.Actor.IsValid())
		{
			ABLOG(Warning, TEXT("Hit Actor Name : %s"), *HitResult.Actor->GetName());
		}
	}
}
```

<br>

![녹화_2023_03_06_18_29_57_585_AdobeExpress](https://user-images.githubusercontent.com/93882395/223071248-56ba3dfe-2124-4932-9251-04b90256c7f2.gif) 

>   결과 화면
>
>   공격 판정이 인정되면 초록색, 판정되지 않으면 붉은색으로 표시된다. 

<br>

---

# **4. 대미지 프레임워크**

대미지 프레임워크를 활용하면 감지된 액터에 대미지를 전달할 수 있다. 액터 클래스의 `AActor`에 있는 `TakeDamage` 함수를 사용해 데미지를 전달해보자.

**`TakeDamage` 함수 인자**

*   `DamageAmount` : 전달할 대미지의 세기
*   `DamageEvent` : 대미지 종류
*   `EventInstigator` : 공격 명령을 내린 주체(폰x, 컨트롤러!)
*   `DamageCauser` : 대미지 전달을 위해 사용한 도구

<br>

대미지 기능 구현

1.   대미지 전달 로직 작성
2.   받은 대미지를 처리하는 로직 추가
     *   액터의 `TakeDamage` 함수 오버라이드
         *   부모 클래스이므로 `Super` 키워드를 사용해 부모 클래스 로직 우선 실행
3.   `ABAnimInstance` 에서 `IsDead` 속성 추가
4.   애님 인스턴스에 캐릭터 사망 여부 확인
5.   애님 그래프에서 `IsDead`가 참이 되면 죽는 애니메이션 동작을 재생하도록 노드 연결
     *   애니메이션을 반복하지 않도록 `Loop` 해제
6.   캐릭터에 사망 시 액터의 충돌 설정을 해제하도록 로직 작성

<br>

**ABCharacter.h**

```c++
class ARENABATTLE_API AABCharacter : public ACharacter
{
public:
	// 받은 대미치 처리용 로직 선언(오버라이드)
	virtual float TakeDamage(float DamageAmount, struct FDamageEvent const& DamageEvent, class AController* EventInstigator, AActor* DamageCauser) override;
	...
}
```

<br>

**ABCharacter.cpp**

```c++
// 피격 대미지 처리 함수
float AABCharacter::TakeDamage(float DamageAmount, FDamageEvent const& DamageEvent, AController* EventInstigator, AActor* DamageCauser)
{
	float FinalDamage = Super::TakeDamage(DamageAmount, DamageEvent, EventInstigator, DamageCauser);
	ABLOG(Warning, TEXT("Actor : %s took Damage : %f"), *GetName(), FinalDamage);
    
    // 사망 시 충돌 해제
    if (FinalDamage > 0.0f)
	{
		ABAnim->SetDeadAnim();
		SetActorEnableCollision(false);
	}
    
	return FinalDamage;
}

void AABCharacter::AttackCheck()
{
	...

#endif

	if (bResult)
	{
		if (HitResult.Actor.IsValid())
		{
			// 대미지 전달 로직
			ABLOG(Warning, TEXT("Hit Actor Name : %s"), *HitResult.Actor->GetName());

			FDamageEvent DamageEvent;
			HitResult.Actor->TakeDamage(50.0f, DamageEvent, GetController());
		}
	}
}
```

<br>

**ABAnimInstance.h**

```c++
class ARENABATTLE_API UABAnimInstance : public UAnimInstance
{
public:
	...
	
	void SetDeadAnim() { IsDead = true; }


private:
	...

	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = Pawn, Meta = (AllowPrivateAccess = true))
	bool IsDead;
}
```

<br>

**ABAnimInstance.cpp**

```c++
UABAnimInstance::UABAnimInstance()
{
	IsDead = false;
	...
}

void UABAnimInstance::NativeUpdateAnimation(float DeltaSeconds)
{
	Super::NativeUpdateAnimation(DeltaSeconds);
	auto Pawn = TryGetPawnOwner();
	if (!::IsValid(Pawn)) return;

	if (!IsDead)
	{
		CurrentPawnSpeed = Pawn->GetVelocity().Size();
		auto Character = Cast<ACharacter>(Pawn);
		if (Character)
		{
			IsInAir = Character->GetMovementComponent()->IsFalling();
		}
	}
}

void UABAnimInstance::PlayAttackMontage()
{
	ABCHECK(!IsDead);
	Montage_Play(AttackMontage, 1.0f);
}

void UABAnimInstance::JumpToAttackMontageSection(int32 NewSection)
{
	ABCHECK(!IsDead);
	ABCHECK(Montage_IsPlaying(AttackMontage));
	Montage_JumpToSection(GetAttackMontageSectionName(NewSection), AttackMontage);
}
```

<br>

<img src="https://user-images.githubusercontent.com/93882395/223078911-7dd93300-51ba-4966-9709-c313a4fdfb42.png" alt="image" style="zoom:80%;" /> 

>   블루프린트 수정 화면

<br>

![녹화_2023_03_06_19_10_27_810_AdobeExpress](https://user-images.githubusercontent.com/93882395/223081057-7c3b5532-a6fc-45b8-a7d1-96815d0a1dae.gif) 

>   결과 화면
>
>   캐릭터 공격 시 죽는 애니메이션을 재생하고 더이상 충돌 이벤트가 발생하지 않는다.
