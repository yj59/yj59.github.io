---
title: "[이득우의 C++ 언리얼] #12-1 AI 컨트롤러1"

date: 2023-02-01 00:00:00 +0900
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

# **1. AIController**

## **1.1. AI 컨트롤러 생성**

캐릭터에 인공지능을 추가해 스스로 영역을 정찰하고 플레이어에게 상호작용을 하도록 만들 수 있다. AI 컨트롤러에 캐릭터를 빙의시켜 스스로 움직이는 기능을 구현해보자.

<br>

캐릭터 생성자 코드에서 AI를 구현하려면 아래처럼 코드를 작성하자.

**ABCharacter.cpp**

```c++
#include "ABAIController.h"

AABCharacter::AABCharacter()
{
    ...
        
    AIControllerClass = AABAIController::StaticClass();
    AutoPossessAI = EAutoPossessAI::PlacedInWorldOrSpawned;
}
```

*   `PlacedInWorldOrSpawned` : 새 `ABCharacter`마다 `ABAIController` 액터 생성

<br>

## **1.2. 네비게이션 메시**

`AIController`에 로직을 부여해 플레이어를 따라다니는 기능을 추가해보자.

1.   내비게이션 메시 배치
     *   NPC가 스스로 움직일 수 있는 환경 구축
     *   `모드` -> `볼륨` -> `NavMesh Bounds Volume` (10000x10000x500cm)
     *   뷰포트 선택 후 P(내비 메시 영역 생성)
2.   `ABAIController`에 빙의한 폰에게 목적지 알림
     *   `GetRandomPointInNavigableRadius` : 이동 가능한 목적지를 랜덤으로 가져옴
3.   AI 컨트롤러에 타이머 설치
     *   3초마다 폰에게 목적지로 이동하는 명령 내림
     *   `SimpleMoveToLocation` : 목적지로 폰 이동시킴

<br>

**ABAIController.h**

```c++
#include "ArenaBattle.h"
...

UCLASS()
class ARENABATTLE_API AABAIController : public AAIController
{
	GENERATED_BODY()

public:
	AABAIController();
	virtual void OnPossess(APawn* InPawn) override;
	virtual void OnUnPossess() override;
	
private:
	void OnRepeatTimer();

	FTimerHandle RepeatTimerHandle;
	float RepeatInterval;
};
```

<br>

**ABAIController.cpp**

```c++
#include "ABAIController.h"
#include "NavigationSystem.h"
#include "Blueprint/AIBlueprintHelperLibrary.h"

AABAIController::AABAIController()
{
	RepeatInterval = 3.0f;
}

void AABAIController::OnPossess(APawn* InPawn)
{
	Super::OnPossess(InPawn);
	GetWorld()->GetTimerManager().SetTimer(RepeatTimerHandle, this, &AABAIController::OnRepeatTimer, RepeatInterval, true);
}

void AABAIController::OnUnPossess()
{
	Super::OnUnPossess();
	GetWorld()->GetTimerManager().ClearTimer(RepeatTimerHandle);
}

void AABAIController::OnRepeatTimer()
{
	auto CurrentPawn = GetPawn();
	ABCHECK(nullptr != CurrentPawn);

	UNavigationSystemV1* NavSystem = UNavigationSystemV1::GetNavigationSystem(GetWorld());
	if (nullptr != NavSystem) return;

	FNavLocation NextLocation;
	if (NavSystem->GetRandomPointInNavigableRadius(FVector::ZeroVector, 500.0f, NextLocation))
	{
		UAIBlueprintHelperLibrary::SimpleMoveToLocation(this, NextLocation.Location);
		ABLOG(Warning, TEXT("Next Location : %s"), *NextLocation.Location.ToString());
	}
}
```

<br>

---

# **2. 비헤이비어 트리**

## **2.1. 비헤이비어 생성**

언리얼 엔진에서 비헤이비어 트리 모델과 편집 에디터를 제공해 NPC의 복잡한 행동 패턴을 체계적으로 설계할 수 있다.

**비헤이어트리 요구 애셋**

*   비헤이비어 트리: 비헤이비어 트리 정보 저장(시각화)
*   블랙보드 애셋: 인공지능의 판단에 사용하는 데이터 집합

<br>

비헤이비어 트리 에디터에서 태스크를 통해 동작 명령을 내릴 수 있다. 단, 태스크는 독립적으로 실행될 수 없어 컴포짓 노드를 거친다.

[**컴포짓 노드**](https://docs.unrealengine.com/4.27/ko/InteractiveExperiences/ArtificialIntelligence/BehaviorTrees/BehaviorTreeNodeReference/BehaviorTreeNodeReferenceComposites/): 왼쪽으로 오른쪽 순서로 태스크 실행

*   셀렉터: 태스크 중 하나가 **성공**하면 실행 중단
*   시퀀스: 태스크 중 하나가 **실패**하면 실행 중단

<br>

![image](https://user-images.githubusercontent.com/93882395/224038094-8660affc-d8da-4a39-beb6-4dd2ed3638fb.png) 

>   기본적인 노드 연결 형태

<br>

비헤이비어 트리와 블랙보드를 코드에서 사용하기 위해 AIModule 모듈을 추가해야 한다. `ABAIController` 클래스의 모든 코드를 비헤이비어 트리 구동에 맞게 다시 작성하자.

<br>

**ArenaBattle.Build.cs**

```c#
PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore", "UMG", "NavigationSystem", "AIModule" });
```

<br>

**ABAIController.h**

```c++
UCLASS()
class ARENABATTLE_API AABAIController : public AAIController
{
	GENERATED_BODY()

public:
	AABAIController();
	virtual void OnPossess(APawn* InPawn) override;
	
private:
	UPROPERTY()
	class UBehaviorTree* BTAsset;

	UPROPERTY()
	class UBlackboardData* BBAsset;
};
```

<br>

**ABAIController.cpp**

```c++
#include "ABAIController.h"
#include "BehaviorTree/BehaviorTree.h"
#include "BehaviorTree/BlackboardData.h"

AABAIController::AABAIController()
{
	static ConstructorHelpers::FObjectFinder<UBlackboardData> BBObject(TEXT("/Game/Book/AI/BB_ABCharacter.BB_ABCharacter"));
	if (BBObject.Succeeded())
	{
		BBAsset = BBObject.Object;
	}

	static ConstructorHelpers::FObjectFinder<UBehaviorTree> BTObject(TEXT("/Game/Book/AI/BT_ABCharacter.BT_ABCharacter"));
	if (BTObject.Succeeded())
	{
		BTAsset = BTObject.Object;
	}
}

void AABAIController::OnPossess(APawn* InPawn)
{
	Super::OnPossess(InPawn);

	if (UseBlackboard(BBAsset, Blackboard))
	{
		if (!RunBehaviorTree(BTAsset))
		{
			ABLOG(Error, TEXT("AIController coudn't run behavior tree!"));
		}
	}
}
```

`ABAIController` 가동 시 비헤이비어 트리 애셋과 같은 폴더에 위치한 블랙보드 애셋, 비헤이비어 트리가 함께 동작한다!

<br>

## **2.2. 비헤이비어 트리 활용**

### **2.2.1. 캐릭터 초기값 지정**

NPC의 순찰 기능을 구현하기 위해 블랙보드에 데이터가 두 개 필요하다. 블랙보드에 정보를 보관할 `vector` 타입 키를 두 개 만들어주자.

1.   NPC 생성 위치 값(`HomePos`)
2.   앞으로 NPC가 순찰할 위치 정보(`PatrolPos`)

<br>

**ABAIController.h**

```c++
class ARENABATTLE_API AABAIController : public AAIController
{
...

public:
	static const FName HomePosKey;
	static const FName PatrolPosKey;
}
```

<br>

**ABAIController.cpp**

```c++
#include "BehaviorTree/BlackboardComponent.h"

const FName AABAIController::HomePosKey(TEXT("HomePos"));
const FName AABAIController::PatrolPosKey(TEXT("PatrolPos"));

void AABAIController::OnPossess(APawn* InPawn)
{
	Super::OnPossess(InPawn);

	if (UseBlackboard(BBAsset, Blackboard))
	{
		// 액터의 위치정보를 HomePosKey에 저장
		Blackboard->SetValueAsVector(HomePosKey, InPawn->GetActorLocation());
		if (!RunBehaviorTree(BTAsset))
		{
			ABLOG(Error, TEXT("AIController coudn't run behavior tree!"));
		}
	}
}
```

<br>

![image](https://user-images.githubusercontent.com/93882395/224052581-5af20e68-3712-482b-8a6e-5d33648c4247.png) 

>   게임을 플레이하면 캐릭터의 `HomePos` 키 값이 블랙보드에 전달된다.

<br>

### **2.2.2. 캐릭터 유동값 저장**

NPC가 이동할 위치인 `PatrolPos` 데이터를 생성해보자. NPC 순찰마다 데이터가 변경되므로 태스크를 제작해 비헤이비어 트리에서 블랙보드에 값을 쓰도록 설계하는 것이 좋다.

태스크 로직을 작성하기 위해 `BTTaskNode`를 부모 클래스로 하는 C++ 클래스를 생성한다.

<br>

**ArenaBattle.Build.cs**

```c++
PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore", "UMG", "NavigationSystem", "AIModule", "GamePlayTasks" });
```

*   `GamePlayTasks` : 비헤이비어 트리 태스크 관련 모듈 참조

<br>

비헤이비어 트리는 태스크를 실행할 때 태스크 클래스의 `ExecuteTask` 멤버 함수를 실행한다.

**`ExecuteTask` 함수 반환값**

*   `Aborted` : 태스크 실행 중 중단(실패)
*   `Failed` : 태스크 수행 실패
*   `Succeeded` : 태스크 수행 성공
*   `InProgress` : 태스크 계속 수행(실행 결과 향후 알림)

<br>

**BTTask_FindPatrolPos.h**

```c++
UCLASS()
class ARENABATTLE_API UBTTask_FindPatrolPos : public UBTTaskNode
{
	GENERATED_BODY()
	
public:
	UBTTask_FindPatrolPos();

	virtual EBTNodeResult::Type ExecuteTask(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory) override;
};
```

<br>

**BTTask_FindPatrolPos.cpp**

```c++
#include "BTTask_FindPatrolPos.h"
#include "ABAIController.h"
#include "BehaviorTree/BlackboardComponent.h"
#include "NavigationSystem.h"

UBTTask_FindPatrolPos::UBTTask_FindPatrolPos()
{
	NodeName = TEXT("FindPatrolPos");
}

EBTNodeResult::Type UBTTask_FindPatrolPos::ExecuteTask(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory)
{
	EBTNodeResult::Type Result = Super::ExecuteTask(OwnerComp, NodeMemory);

	auto ControllingPawn = OwnerComp.GetAIOwner()->GetPawn();
	if (nullptr == ControllingPawn) return EBTNodeResult::Failed;

	UNavigationSystemV1* NavSystem = UNavigationSystemV1::GetNavigationSystem(ControllingPawn->GetWorld());
	if (nullptr == NavSystem) return EBTNodeResult::Failed;

	FVector Origin = OwnerComp.GetBlackboardComponent()->GetValueAsVector(AABAIController::HomePosKey);
	FNavLocation NextPatrol;

	if (NavSystem->GetRandomPointInNavigableRadius(FVector::ZeroVector, 500.0f, NextPatrol))
	{
		OwnerComp.GetBlackboardComponent()->SetValueAsVector(AABAIController::PatrolPosKey, NextPatrol.Location);
		return EBTNodeResult::Succeeded;
	}

	return EBTNodeResult::Failed;
}
```

<br>

---

# *



![image](https://user-images.githubusercontent.com/93882395/224065647-3f14667b-aa02-4927-949f-5d00330d31b3.png)

코드를 잘못 작성하고 컴파일하면 비헤이비어 트리가 실행되지 않는다. *(스텝 백: -1)*

해당 경우는 실패 로그 출력 조건을 잘못 주어 오류가 발생했다! 

>   ```
>// if(nullptr == ControllingPawn)
>   if(nullptr != ControllingPawn)
>```
>   

