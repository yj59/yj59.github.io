---
title: "[이득우의 C++ 언리얼] #12-2 AI 컨트롤러2"

date: 2023-02-02 00:00:00 +0900
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

# **3. 비헤이비어트리 확장**



## **3.1. NPC 추격 기능 구현**

1.   블랙보드에 `Object`타입 변수 생성(`Target`)

     *   NPC가 플레이어 발견 시 플레이어의 정보 저장
     *   기반 클래스로 `ABCharacter` 지정

2.   셀렉터 컴포짓을 활용해 로직 확장

     *   NPC가 추격과 정찰 중 하나를 선택해 행동하도록 명령
     *   추격에 우선권 부여
         *   `Target`을 향해 이동하도록 설계 확장

3.   서비스노드 클래스(부모클래스 `BTService`) 생성

     *   연결된 컴포짓 노드가 활성화되면 주기적으로 `TickNode` 함수 호출 (호출 주기 지정 가능 `Interval`)

         *(eg. 정찰 중 플레이어가 일정 반경 내에 있으면 감지해 추격)*

4.   `TickNode` 함수에 탐지 영역 구현

5.   비헤이비어 트리 에디터에서 컴포짓에 Detect 서비스 부착

6.   `ABAIController`에 키 등록

7.   `IsPlayerController` 함수 사용

     *   캐릭터가 감지되면 `Target`을 플레이어 캐릭터로 지정 

         (그렇지 않으면 `nullptr`)

     *   플레이어 캐릭터 감지 시 녹색 구체와 연결선 그림



<br>

**BTService_Detect.h**

```c++
#include "ArenaBattle.h"
#include "BehaviorTree/BTService.h"
#include "BTService_Detect.generated.h"

UCLASS()
class ARENABATTLE_API UBTService_Detect : public UBTService
{
	GENERATED_BODY()
	
public:
	UBTService_Detect();

protected:
	virtual void TickNode(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory, float DeltaSeconds) override;
};
```

<br>

**BTService_Detect.cpp**

```c++
#include "BTService_Detect.h"
#include "ABAIController.h"
#include "BehaviorTree/BlackboardComponent.h"
#include "DrawDebugHelpers.h"

UBTService_Detect::UBTService_Detect()
{
	NodeName = TEXT("Detect");
	Interval = 1.0f;
}

void UBTService_Detect::TickNode(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory, float DeltaSeconds)
{
	Super::TickNode(OwnerComp, NodeMemory, DeltaSeconds);

	APawn* ControllingPawn = OwnerComp.GetAIOwner()->GetPawn();
	if (nullptr == ControllingPawn) return;

	UWorld* World = ControllingPawn->GetWorld();
	FVector Center = ControllingPawn->GetActorLocation();
	float DetectRadius = 600.0f;

	if (nullptr == World) return;
	TArray<FOverlapResult> OverlapResults;
	FCollisionQueryParams CollisionQueryParam(NAME_None, false, ControllingPawn);
	bool bResult = World->OverlapMultiByChannel(
		OverlapResults,
		Center,
		FQuat::Identity,
		ECollisionChannel::ECC_GameTraceChannel2,
		FCollisionShape::MakeSphere(DetectRadius),
		CollisionQueryParam
	);

	DrawDebugSphere(World, Center, DetectRadius, 16, FColor::Red, false, 0.2f);
}
```

<br>

**ABAIController.h**

```c++
class ARENABATTLE_API AABAIController : public AAIController
{
public:
	static const FName Target;
    ...
}
```

<br>

**ABAIController.cpp**

```
const FName AABAIController::Target(TEXT("Target"));
```

<br>

![image](https://user-images.githubusercontent.com/93882395/224227733-e3b30089-c57c-4a06-92ce-88af5a630667.png) 

>   비헤이비어트리 서비스 추가

<br>

![녹화_2023_03_10_15_47_08_902_AdobeExpress (1)](https://user-images.githubusercontent.com/93882395/224244586-a9c66e5a-c62a-4590-b1b9-750f083a4ff2.gif) 

>   결과 화면

<br>

## **3.2. NPC 추가 설정**

*   NPC 회전을 자연스럽게 수정
    *   `ControlMode`를 추가해 이동 방향에 따라 회전하도록 무브먼트 설정 변경
*   NPC의 최대 이동 속도를 플레이어보다 낮게 설정
*   데코레이터 노드 삽입
    *   블랙보드 값을 기반으로 특정 컴포짓의 실행 여부 결정
    *   `Target`키값 유무로 추격과 정찰 구분
*   데코레이터 속성 지정
    *   노티파이 옵저버 값을 `On Value Change`로 변경
        *   키 값의 변경을 감지하면 현재 컴포짓 노드 실해 ㅇ취소
    *   관찰자 중단 값 설정(`Self`)
    *   블랙보드 키값 설정(추격->`Is Set`, 정찰->`Is Not Set`)

<br>

**ABCharacter.h**

```c++
class ARENABATTLE_API AABCharacter : public ACharacter
{
protected:
	enum class EControlMode
	{
		GTA,
		DIABLO,
		NPC
	};
    
	...
        
public:
    virtual void PossessedBy(AController* NewController) override;
}
```

<br>

**ABCharacter.cpp**

```c++
void AABCharacter::SetControlMode(EControlMode NewControlMode)
{
	...
	
	switch (CurrentControlMode)
	{
	
	...
	
	case EControlMode::NPC:
		bUseControllerRotationYaw = false;
		GetCharacterMovement()->bUseControllerDesiredRotation = false;
		GetCharacterMovement()->bOrientRotationToMovement = true;
		GetCharacterMovement()->RotationRate = FRotator(0.0f, 480.f, 0.0f);
		break;
	}
}

void AABCharacter::PossessedBy(AController* NewController)
{
	Super::PossessedBy(NewController);

	if (IsPlayerControlled())
	{
		SetControlMode(EControlMode::DIABLO);
		GetCharacterMovement()->MaxWalkSpeed = 600.0f;
	}
	else
	{
		SetControlMode(EControlMode::NPC);
		GetCharacterMovement()->MaxWalkSpeed = 300.0f;
	}
}
```

<br>

![image](https://user-images.githubusercontent.com/93882395/224248322-7bddd472-3c7c-4664-8ac0-cc71d053b7a1.png) 

>    비헤이비어 트리 데코레이터 추가

<br>

---

# **4. NPC 공격 기능**

## 4.1. 데코레이터 로직

플레이어의 거리에 따라 추격과 공격으로 분기하므로 비헤이비어 트리의 왼쪽 로직을 두 갈래로 분기시킨다.

블랙보드 값을 참조하지 않고 플레이어가 범위 이내에 있는지 판단하도록 데코레이터 클래스를 생성한다. 데코레이터 클래스는 `BTDecorator`를 부모로 한다.

<br>

**BTDecorator_IsInAttackRange.h**

```c++
class ARENABATTLE_API UBTDecorator_IsInAttackRange : public UBTDecorator
{
	GENERATED_BODY()

public:
	UBTDecorator_IsInAttackRange();

protected:
	virtual bool CalculateRawConditionValue(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory) const override;
};
```

*   [`CalculateRawConditionValue`](https://docs.unrealengine.com/4.27/en-US/API/Runtime/AIModule/BehaviorTree/UBTDecorator/CalculateRawConditionValue/) : 데코레이터 조건 값 계산
    *   데코레이터는 해당 함수를 통해 원하는 조건이 달성되었는지 파악

<br>

**BTDecorator_IsInAttackRange.cpp**

```c++
#include "BTDecorator_IsInAttackRange.h"
#include "ABAIController.h"
#include "ABCharacter.h"
#include "BehaviorTree/BlackboardComponent.h"

UBTDecorator_IsInAttackRange::UBTDecorator_IsInAttackRange()
{
	NodeName = TEXT("CanAttack");
}

bool UBTDecorator_IsInAttackRange::CalculateRawConditionValue(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory) const
{
	bool bResult = Super::CalculateRawConditionValue(OwnerComp, NodeMemory);

	auto ControllingPawn = OwnerComp.GetAIOwner()->GetPawn();
	if (nullptr == ControllingPawn) return false;

	auto Target = Cast<AABCharacter>(OwnerComp.GetBlackboardComponent()->GetValueAsObject(AABAIController::TargetKey));
	if (nullptr == Target) return false;

	bResult = (Target->GetDistanceTo(ControllingPawn) <= 200.0f);
	return bResult;
}
```

<br>

![image](https://user-images.githubusercontent.com/93882395/224254258-a28990fd-d49a-4c1d-84fb-685272298641.png) 

>   데코레이터 로직 작성 후 추가

<br>

## 4.2. 태스크 로직

### **4.2.1. UBTTask_Attack**

`BTTaskNode`를 부모로 하는 `BTTask_Attack` 클래스를 생성해 공격 로직을 작성해보자.

1.   `FinishiLatentTask` 함수 선언
     *   `ExecuteTask`의 결과값을 `InProgress`로 반환한 후 공격이 끝나면 태스크 종료 알림 제공
     *   함수가 호출되지 않을 경우 비헤이비어 시스템이 현재 태스크 계속 잔존
2.   `Tick` 기능 활성화
     *   `FinishLatentTask` 호출
     *   `Tick`에서 조건을 파악한 후 태스크 종료 명령 내림
3.   `ABController` 클래스의 `Attack` 함수 접근 권한을 `public`으로 변경
     *   AI컨트롤러에서도 공격 명령을 내리기 위함
4.   공격 종료 알림을 받도록 델리게이트 선언
     *   공격 종료 시 호출할 로직 구현 (캐릭터)
     *   태스크에서 람다 함수를 델리게이트에 등록
     *   `Tick` 함수 로직에서 델리게이트 반환값을 받으면 `FinishiLatentTask` 함수를 호출하고 태스크 종료
5.   비헤이비어 트리에 `Attack` 태스크 삽입

<br>

**UBTTask_Attack.h**

```c++
class ARENABATTLE_API UBTTask_Attack : public UBTTaskNode
{
	GENERATED_BODY()

public:
	UBTTask_Attack();

	virtual EBTNodeResult::Type ExecuteTask(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory) override;	

protected:
	virtual void TickTask(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory, float DeltaSeconds) override;
    
private:
	bool IsAttacking = false;

};
```

<br>

**UBTTask_Attack.cpp**

```c++
#include "BTTask_Attack.h"
#include "ABAIController.h"
#include "ABCharacter.h"

UBTTask_Attack::UBTTask_Attack()
{
	bNotifyTick = true;
}

EBTNodeResult::Type UBTTask_Attack::ExecuteTask(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory)
{
	EBTNodeResult::Type Result = Super::ExecuteTask(OwnerComp, NodeMemory);

	auto ABCharacter = Cast<AABCharacter>(OwnerComp.GetAIOwner()->GetPawn());
	if (nullptr != ABCharacter) return EBTNodeResult::Failed;

	ABCharacter->Attack();
	IsAttacking = true;
	ABCharacter->OnAttackEnd.AddLambda([this]()->void {
		IsAttacking = false;
	});

	return EBTNodeResult::InProgress;
}

void UBTTask_Attack::TickTask(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory, float DeltaSeconds)
{
	Super::TickTask(OwnerComp, NodeMemory, DeltaSeconds);
	FinishLatentTask(OwnerComp, EBTNodeResult::Succeeded());

	if (!IsAttacking)
	{
		FinishLatentTask(OwnerComp, EBTNodeResult::Succeeded);
	}
}
```

<br>

**ABCharacter.h**

```c++
DECLARE_MULTICAST_DELEGATE(FOnAttackEndDelegate);

UCLASS()
class ARENABATTLE_API AABCharacter : public ACharacter
{
public:
	void Attack();
	FOnAttackEndDelegate OnAttackEnd;
    ...
}
```

<br>

**ABCharacter.cpp**

```c++
void AABCharacter::OnAttackMontageEnded(UAnimMontage* Montage, bool bInterrupted)
{
	...

	OnAttackEnd.Broadcast();
}
```

<br>

![녹화_2023_03_10_20_30_52_546_AdobeExpress (1)](https://user-images.githubusercontent.com/93882395/224306001-a35b1860-00d2-42b3-9bc6-5b8e5be47e2d.gif) 

>    결과 화면

<br>

### **4.2.2. BTTask_TurnToTarget**

조금 더 자연스러운 동작을 위해 NPC가 공격과 동시에 플레이어를 향해 회전하는 기능을 추가한다.

1.   블랙보드의 `Target`으로 회전하는 태스크 추가

2.   `BTTaskNode`를 부모 클래스로 하는 태스크 생성(`BTTask_TurnToTarget`)

3.   플레이어 폰을 향해 일정한 속도로 회전하는 기능 구현

     *   `FMath::RInterpTo` 함수 사용

4.   공격 로직의 시퀀스 컴포짓을 심플 패러럴 컴포짓으로 변경

     *   공격과 회전 태스크를 동시에 실행시키기 위함
     *   공격을 메인 태스크 , 회전을 보조 태스크로 지정

     

<br>

**BTTask_TurnToTarget.h**

```c++
class ARENABATTLE_API UBTTask_TurnToTarget : public UBTTaskNode
{
	GENERATED_BODY()
	
public:
	UBTTask_TurnToTarget();
	virtual EBTNodeResult::Type ExecuteTask(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory) override;
};
```

<br>

**BTTask_TurnToTarget.cpp**

```c++
#include "BTTask_TurnToTarget.h"
#include "ABAIController.h"
#include "ABCharacter.h"
#include "BehaviorTree/BlackboardComponent.h"

UBTTask_TurnToTarget::UBTTask_TurnToTarget()
{
	NodeName = TEXT("Turn");
}

EBTNodeResult::Type UBTTask_TurnToTarget::ExecuteTask(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory)
{
	EBTNodeResult::Type Result = Super::ExecuteTask(OwnerComp, NodeMemory);

	auto ABCharacter = Cast<AABCharacter>(OwnerComp.GetAIOwner()->GetPawn());
	if (nullptr == ABCharacter) return EBTNodeResult::Failed;

	auto Target = Cast<AABCharacter>(OwnerComp.GetBlackboardComponent()->GetValueAsObject(AABAIController::TargetKey));
	if (nullptr == Target) return EBTNodeResult::Failed;

	FVector LookVector = Target->GetActorLocation() - ABCharacter->GetActorLocation();
	LookVector.Z = 0.0f;

	FRotator TargetRot = FRotationMatrix::MakeFromX(LookVector).Rotator();
	ABCharacter->SetActorRotation(FMath::RInterpTo(ABCharacter->GetActorRotation(), TargetRot, GetWorld()->GetDeltaSeconds(), 2.0f));

	return EBTNodeResult::Succeeded;
}
```

<br>

![image](https://user-images.githubusercontent.com/93882395/224310386-11576d80-348f-484a-a7db-23a0a68ccfe5.png)

>   비헤이비어 트리 결과
