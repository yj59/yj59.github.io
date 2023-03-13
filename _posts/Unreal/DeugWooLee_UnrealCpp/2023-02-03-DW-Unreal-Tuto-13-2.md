---
title: "[이득우의 C++ 언리얼] #13-2 무한 맵 생성"

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
last_modified_at: 2023-02-03

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

# **1. 섹션 액터 제작**

`Actor`를 부모 클래스로 하는 `ABSection` 클래스를 만들어 무한 맵 스테이지를 제작할 수 있다.

**섹션 액터가 할 일**

*   섹션의 배경과 네 방향으로 캐릭터 입장을 통제하는 문 제공
*   플레이어가 섹션에 진입하면 모든 문 닫음
*   문을 닫고 일정 시간 후에 섹션 중앙에서 NPC 생성
*   문을 닫고 일정 시간 후에 아이템 상자를 섹션 내 랜덤한 위치에 생성
*   생성한 NPC가 죽으면 모든 문 개방
*   통과한 문으로 이어지는 새로운 섹션 생성

<br>

액터에 스태틱메시 컴포넌트를 선언한 후 루트로 설정하고 현재 배경 애셋을 지정한다. 예제로 제공된 애셋은 방향별로 출입문과 섹션을 이어붙일 수 있도록 소켓이 부착되어 있다. 

배경의 각 출입구에 철문을 배치시켜보자. 철문마다 스태틱메시 컴포넌트를 제작해 소켓에 부착한다. 코드에 소켓 목록을 제작하고 철문을 각각 부착할 수 있다. 철문은 모두 동일한 기능을 가지므로 `TArray`로 관리한다.

**ABSection.h**

```c++
class ARENABATTLE_API AABSection : public AActor
{
...

private:
	// 철문 관리용 TArray
	UPROPERTY(VisibleAnywhere, Category = Mesh, Meta = (AllowPrivateAccess = true))
	TArray<UStaticMeshComponent*> GateMeshes;

	// 루트로 설정할 배경 메시의 스태틱 컴포넌트
	UPROPERTY(VisibleAnywhere, Category = Mesh, Meta = (AllowPrivateAccess = true))
	UStaticMeshComponent* Mesh;
};
```

<br>

**ABSection.cpp**

```c++
AABSection::AABSection()
{
	PrimaryActorTick.bCanEverTick = false;

	// 루트 컴포넌트를 Mesh로 지정
	Mesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("MESH"));
	RootComponent = Mesh;

	// 배경 애셋 경로 참조
	FString AssetPath = TEXT("/Game/Book/StaticMesh/SM_SQUARE.SM_SQUARE");
	// ConstructorHelpers: 애셋의 내용물을 가져올 때 사용
	static ConstructorHelpers::FObjectFinder<UStaticMesh> SM_SQUARE(*AssetPath);
	if (SM_SQUARE.Succeeded())
	{
		Mesh->SetStaticMesh(SM_SQUARE.Object);
	}
	else
	{
		ABLOG(Error, TEXT("Failed to load staticmesh asset. : %s"), *AssetPath);
	}

	// 철문 애셋 경로 참조
	FString GateAssetPath = TEXT("/Game/Book/StaticMesh/SM_GATE.SM_GATE");
	static ConstructorHelpers::FObjectFinder<UStaticMesh> SM_GATE(*GateAssetPath);
	if (!SM_GATE.Succeeded())
	{
		ABLOG(Error, TEXT("Failed to load staticmesh asset : %s"), *GateAssetPath);
	}

	// 철문에 부착할 소켓 목록 제작
	static FName GateSokets[] = { {TEXT("+XGate")}, {TEXT("-XGate")}, {TEXT("+YGate")}, {TEXT("-YGate")} };
	for (FName GateSocket : GateSokets)
	{
		// 배경 애셋에 소켓이 없으면 오류 로그 출력
		ABCHECK(Mesh->DoesSocketExist(GateSocket));
		UStaticMeshComponent* NewGate = CreateDefaultSubobject<UStaticMeshComponent>(*GateSocket.ToString());
		NewGate->SetStaticMesh(SM_GATE.Object);
		// NewGate의 부착 지점 설정
		NewGate->SetupAttachment(RootComponent, GateSocket);
		NewGate->SetRelativeLocation(FVector(0.0f, -80.5f, 0.0f));
		GateMeshes.Add(NewGate);
	}
}
```

<br>

![image](https://user-images.githubusercontent.com/93882395/224471940-d576ce20-a303-455d-beb7-593aecadf63c.png) 

>   철문 부착 화면

<br>

# **2. 트리거 영역 추가**

플레이어의 입장을 감지하고 출구를 선택할 때 사용할 콜리전 프리셋을 생성한다.

![image](https://user-images.githubusercontent.com/93882395/224472122-ac69fc3b-f2a6-4122-bdb6-b6b399e89d59.png) 

>   캐릭터만 감지하도록 트리거 프리셋을 설정한다.

<br>

**ABSection.h**

```c++
class ARENABATTLE_API AABSection : public AActor
{
    ...
        
private:
	...

	// 게이트별 트리거 정보를 담을 Box 컴포넌트 리스트
	UPROPERTY(VisibleAnywhere, Category = Trigger, Meta = (AllowPrivateAccess = true))
	TArray<UBoxComponent*> GateTriggers;

	// 섹션 중앙에 부착할 Box 컴포넌트
	UPROPERTY(VisibleAnywhere, Category = Trigger, Meta = (AllowPrivateAccess = true))
	UBoxComponent* Trigger;
};
```

<br>

**ABSection.cpp**

```c++
AABSection::AABSection()
{
 	...

	// 트리거 이름의 박스 컴포넌트 생성(섹션 중앙)
	Trigger = CreateDefaultSubobject<UBoxComponent>(TEXT("TRIGGER"));
	Trigger->SetBoxExtent(FVector(775.0f, 775.0f, 300.0f));
	Trigger->SetupAttachment(RootComponent);
	Trigger->SetRelativeLocation(FVector(0.0f, 0.0f, 250.0f));
	// 컴포넌트의 콜리전 프리셋 설정
	Trigger->SetCollisionProfileName(TEXT("ABTrigger"));

	// 철문에 부착할 소켓 목록 제작
	static FName GateSokets[] = { {TEXT("+XGate")}, {TEXT("-XGate")}, {TEXT("+YGate")}, {TEXT("-YGate")} };
	for (FName GateSocket : GateSokets)
	{
		...

		// 철문에 부착할 트리거 컴포넌트 리스트 생성
		UBoxComponent* NewGateTrigger = CreateDefaultSubobject<UBoxComponent>(*GateSocket.ToString().Append(TEXT("Trigger")));
		NewGateTrigger->SetBoxExtent(FVector(100.0f, 100.0f, 300.f));
		NewGateTrigger->SetupAttachment(RootComponent, GateSocket);
		NewGateTrigger->SetRelativeLocation(FVector(70.0f, 0.0f, 250.0f));
		NewGateTrigger->SetCollisionProfileName(TEXT("ABTrigger"));
		GateTriggers.Add(NewGateTrigger);
	}
}
```

<br>

![image](https://user-images.githubusercontent.com/93882395/224473314-d0abb150-fcc4-4187-a1cf-2f05e29054e0.png) 

>   액터에 트리거 영역이 추가된 화면

<br>

# **3. 액터 로직 설계**

## 3.1. 스테이트 머신 생성

액터의 로직을 스테이트 머신으로 설계하자.

*   **준비 스테이트**
    *   액터의 시작 스테이트
    *   중앙 트리거가 플레이어 진입을 감지하면 전투 스테이트로 전환
*   **전투 스테이트**
    *   문을 닫고 일정 시간 뒤 NPC 소환
    *   랜덤한 위치에 아이템 상자 생성
    *   NPC가 죽으면 완료 스테이트로 이동
*   **완료 스테이트**
    *   닫힌 철문 개방
    *   각 문에 배치한 트리거 게이트가 플레이어를 감지하면 해당 방향에 새 섹션 소환

<br>

**ABSection.h**

```c++
UCLASS()
class ARENABATTLE_API AABSection : public AActor
{
...

public:
	AABSection();

	// 제작 과정과 시작 스테이트(COMPLETE) 연동
	virtual void OnConstruction(const FTransform& Transform) override;
    
private:
	// 스테이트 리스트 열거
	enum class ESectionState : uint8
	{
		READY = 0,
		BATTLE,
		COMPLETE
	};

	// 스테이트 설정
	void SetState(ESectionState NewState);
	ESectionState CurrentState = ESectionState::READY;

	// 게이트 개방
	void OperateGates(bool bOpen = true);
	
private:
	// 전투 여부 확인
	UPROPERTY(VisibleAnywhere, Category = State, Meta = (AllowPrivateAccess = true))
	bool bNoBattle;
};
```

<br>

**ABSection.cpp**

```c++
AABSection::AABSection()
{
	...
	
	// 플레이 첫 스테이트는 전투 없이 시작하도록 false로 설정
	bNoBattle = false;
}

void AABSection::BeginPlay()
{
	Super::BeginPlay();
	
	SetState(bNoBattle ? ESectionState::COMPLETE : ESectionState::READY);
}

void AABSection::OnConstruction(const FTransform& Transform)
{
	Super::OnConstruction(Transform);
	SetState(bNoBattle ? ESectionState::COMPLETE : ESectionState::READY);
}

void AABSection::SetState(ESectionState NewState)
{
	switch (NewState)
	{
	case ESectionState::READY:
	{
		// 섹션 중앙에서 플레이어를 감지하기 위해 콜리전 프리셋 설정
		Trigger->SetCollisionProfileName(TEXT("ABTrigger"));
		for (UBoxComponent* GateTrigger : GateTriggers)
		{
			GateTrigger->SetCollisionProfileName(TEXT("NoCollision"));
		}
		OperateGates(true);
		break;
	}
	case ESectionState::BATTLE:
	{
		Trigger->SetCollisionProfileName(TEXT("NoCollision"));
		for (UBoxComponent* GateTrigger : GateTriggers)
		{
			GateTrigger->SetCollisionProfileName(TEXT("NoCollision"));
		}
		OperateGates(false);
		break;
	}
	case ESectionState::COMPLETE:
	{
		Trigger->SetCollisionProfileName(TEXT("NoCollision"));
		for (UBoxComponent* GateTrigger : GateTriggers)
		{
			GateTrigger->SetCollisionProfileName(TEXT("NoCollision"));
		}
		OperateGates(true);
		break;
	}
	}
	CurrentState = NewState;
}

void AABSection::OperateGates(bool bOpen)
{
	for (UStaticMeshComponent* Gate : GateMeshes)
	{
		Gate->SetRelativeRotation(bOpen ? FRotator(0.0f, -90.0f, 0.0f) : FRotator::ZeroRotator);
	}
}
```

*   `OnConstruction` : 에디터 작업에서 선택한 액터의 속성이나 트랜스폼 정보가 변경될 때 미리 결과 확인

<br>

![image](https://user-images.githubusercontent.com/93882395/224488947-e056ecf4-a000-49a8-9f9a-24d38986ea55.png) 

>   완료 스테이트 설정이 적용된 에디터 뷰포트

<br>

## **3.2.** 섹션 액터 생성

**섹션 액터 조건**

*   `READY` 스테이트에서부터 시작
    *   가운데 트리거 영역을 활성화하고 플레이어의 진입 감지
    *   플레이어 감지 시 `BATTLE` 스테이트로 전환하고 문 닫음
*   철문 방향에 섹션 액터가 생성되어 있는지 확인
    *   이미 액터가 있다면 생성 건너뜀

<br>

박스 컴포넌트의 감지 기능을 사용해 위 내용을 구현할 수 있다.

1.   `OnComponentBeginOverlap` 델리게이트에 바인딩할 함수 생성
     *   다이내믹 델리게이트이므로 `UFUNCTION` 선언
2.   모든 문의 박스 컴포넌트 델리게이트에 하나의 멤버 함수 연결
     *   각 문의 기능이 모두 동일하기 때문
3.   박스 컴포넌트를 구분할 수 있도록 소켓 이름으로 태그 설정
4.   태그 방향으로 다음 섹션 액터 생성

<br>

**ABSection.h**

```c++
class ARENABATTLE_API AABSection : public AActor
{
...

private:
	...

	UFUNCTION()
	void OnTriggerBeginOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult);

	UFUNCTION()
	void OnGateTriggerBeginOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult);
};
```

<br>

**ABSection.cpp**

```c++
AABSection::AABSection()
{
 	...

	// 박스 컴포넌트와 델리게이트 바인딩
	Trigger->OnComponentBeginOverlap.AddDynamic(this, &AABSection::OnTriggerBeginOverlap);
    
	static FName GateSokets[] = { {TEXT("+XGate")}, {TEXT("-XGate")}, {TEXT("+YGate")}, {TEXT("-YGate")} };
	for (FName GateSocket : GateSokets)
	{
		...

		// 컴포넌트 델리게이트와 게이트 함수 연결
		NewGateTrigger->OnComponentBeginOverlap.AddDynamic(this, &AABSection::OnGateTriggerBeginOverlap);
		// 게이트 소켓명으로 컴포넌트 태그 추가
		NewGateTrigger->ComponentTags.Add(GateSocket);
	}

	// 플레이 첫 스테이트는 전투 없이 시작하도록 false로 설정
	bNoBattle = false;
}

// 박스 컴포넌트 델리게이트와 연결된 함수
void AABSection::OnTriggerBeginOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
	// 플레이어 감지 시 스테이트가 READY 상태라면 BATTLE으로 변경
	if (CurrentState == ESectionState::READY)
	{
		SetState(ESectionState::BATTLE);
	}
}

// 게이트에 부착된 컴포넌트 델리게이트와 연결된 함수
void AABSection::OnGateTriggerBeginOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
	ABCHECK(OverlappedComponent->ComponentTags.Num() == 1);

	// 컴포넌트 태그명과 소켓 이름 반환
	FName ComponentTag = OverlappedComponent->ComponentTags[0];
	FName SocketName = FName(*ComponentTag.ToString().Left(2));
	if (!Mesh->DoesSocketExist(SocketName)) return;

	// 소켓 위치를 섹션 액터를 생성할 위치로 결정
	FVector NewLocation = Mesh->GetSocketLocation(SocketName);

	TArray<FOverlapResult> OverlapResults;
	FCollisionQueryParams CollisionQueryParam(NAME_None, false, this);
	FCollisionObjectQueryParams ObjectQueryParam(FCollisionObjectQueryParams::InitType::AllObjects);

	bool bResult = GetWorld()->OverlapMultiByObjectType(
		OverlapResults,
		NewLocation,
		FQuat::Identity,
		ObjectQueryParam,
		FCollisionShape::MakeSphere(775.0f),
		CollisionQueryParam
	);

	if (!bResult)
	{
		auto NewSection = GetWorld()->SpawnActor<AABSection>(NewLocation, FRotator::ZeroRotator);
	}
	else
	{
		ABLOG(Warning, TEXT("New Section area is not empty"));
	}
}
```

<br>
