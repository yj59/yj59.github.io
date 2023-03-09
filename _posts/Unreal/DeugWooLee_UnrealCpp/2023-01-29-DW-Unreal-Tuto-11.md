---
title: "[이득우의 C++ 언리얼] #11 게임 데이터와 UI 위젯"

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
last_modified_at: 2023-01-29

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

# **1. 액셀 데이터 활용**

액셀에 저장된 데이터를 언리얼으로 불러와보자. 스탯 데이터와 같이 게임의 기반을 이루는 데이터들은 게임이 초기화될 때 불러들이는 것이 일반적이다. 스탯 데이터 등 게임을 관리하기 위해 언리얼에서 제공하는 게임 인스턴스 클래스를 활용한다.

게임이 초기화되면 언리얼에서 `Init` 함수를 호출하므로 `ABGameInstance`클래스에 `Init`함수를 오버라이드해 데이터를 불러오자.

<br>

**ABGameInstance.h**

```c++
#include "ArenaBattle.h"
#include "Engine/DataTable.h"

USTRUCT(BlueprintType)
struct FABCharacterData : public FTableRowBase
{
	GENERATED_BODY()

public:
	FABCharacterData() : Level(1), MaxHP(100.0f), Attack(10.0f), DropExp(10), NextExp(30) {}
	
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Data")
	int32 Level;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Data")
	float MaxHP;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Data")
	float Attack;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Data")
	float DropExp;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Data")
	int32 NextExp;
};

class ARENABATTLE_API UABGameInstance : public UGameInstance
{
	GENERATED_BODY()
	
public:
	UABGameInstance();
	virtual void Init() override;
};
```

<br>

**ABGameInstance.cpp**

```c++
UABGameInstance::UABGameInstance()
{

}

void UABGameInstance::Init()
{
	Super::Init();
	ABLOG_S(Warning);
}
```

<br>

![image](https://user-images.githubusercontent.com/93882395/223600167-312217a5-445f-4a25-b2ac-e937a3406885.png) 

>   csv 임포트시 연동할 데이터 테이블 타입을 선택할 수 있다.

<br>

![image](https://user-images.githubusercontent.com/93882395/223609721-2dc25267-fb8f-44b2-bdfa-003709d28b3d.png) 

>   로그 출력 확인
>
>   *+)*
>
>   *꼭 프로젝트 세팅에서 게임 인스턴스 항목을 새로 만들어준 클래스로 변경해주자!! 한 시간을 해멨다ㅠㅡㅠ*

<br>

# **2. 액터 컴포넌트 제작**

## **2.1. 액터 컴포넌트 클래스**

새로운 액터 컴포넌트를 만들어 캐릭터 스탯 관리를 일임하자. 캐릭터에 부착할 수 있도록 클래스가 컴파일되면 `ABCharacter`의 멤버 변수로 선언한다.

<br>

`InitializeComponent` 함수를 활용해 컴포넌트의 초기화 로직을 구현해보자. `SetNewLevel` 함수를 호출시켜 모든 스탯을 관리하도록 한다.

*   `InitializeComponent` 선언
    *   `PostInitializeComponents` 함수 호출 직전에 호출되는 함수
    *   호출조건: `bWantInitializeComponent` 값을 `true`로 설정
*   스탯 데이터 관리 변수를 `private`로 한정해 선언
    *   `SetNewLevel`함수에서만 스탯 관리
*   게임 인스턴스에서 레벨을 가져와 스탯 초기화
*   레벨이 변경되면 해당 스탯 변경

<br>

**ABCharacter.h**

```c++
UCLASS()
class ARENABATTLE_API AABCharacter : public ACharacter
{

	... 
	
public:
	UPROPERTY(VisibleAnywhere, Category = Stat)
	class UABCharacterStatComponent* CharacterStat;
};
```

<br>

**ABCharacter.cpp**

```c++
#include "ABCharacterStatComponent.h"

AABCharacter::AABCharacter()
{
	...
	
	CharacterStat = CreateDefaultSubobject<UABCharacterStatComponent>(TEXT("CHARACTERSTAT"));
}	
```

<br>

**ABCharacterStatComponent.h**

```c++
#include "ArenaBattle.h"

class ARENABATTLE_API AABCharacterStatComponent : public AActor
{
	...
	
protected:
	virtual void InitializeComponent() override;
    
public:
	void SetNewLevel(int32 NewLevel);

private:
	struct FABCharacterData* CurrentStatData = nullptr;

	UPROPERTY(EditInstanceOnly, Category = Stat, Meta = (AllowPrivateAccess = true))
	int32 Level;

	UPROPERTY(Transient, VisibleInstanceOnly, Category = Stat, Meta = (AllowPrivateAccess = true))
	float CurrentHP;
};
```

<br>

**ABCharacterStatComponent.cpp**

```c++
#include "ABGameInstance.h"

UABCharacterStatComponent::UABCharacterStatComponent()
{
	PrimaryComponentTick.bCanEverTick = false;
	bWantsInitializeComponent = true;
    
    Level = 1;
}

void UABCharacterStatComponent::InitializeComponent()
{
	Super::InitializeComponent();

    SetNewLevel(Level);
}

void UABCharacterStatComponent::SetNewLevel(int32 NewLevel)
{
	auto ABGameInstance = Cast<UABGameInstance>(UGameplayStatics::GetGameInstance(GetWorld()));

	ABCHECK(nullptr != ABGameInstance);
	CurrentStatData = ABGameInstance->GetABCharacterData(NewLevel);
	if (nullptr != CurrentStatData)
	{
		Level = NewLevel;
		CurrentHP = CurrentStatData->MaxHP;
	}
	else
	{
		ABLOG(Error, TEXT("Level (%d) data doesn't exist"), NewLevel);
	}
}
```



<br>

언리얼에서는 오브젝트의 UPROPERTY 속성을 저장하고 롲딩할 수 있다! 단, `CurrentHP` 값처럼 게임을 초기화할 때마다 변경되는 속성은 `Transient` 키워드를 추가해 직렬화에서 제외해주자.

<br>

![image](https://user-images.githubusercontent.com/93882395/223616780-2e286605-62d5-4319-a06f-d320663fe3a4.png) 

>    캐릭터의 레벨을 바꾸고 플레이 버튼을 누르면 레벨에 맞게 HP 상태가 초기화된다.

<br>



---

## **2.2. 캐릭터 대미지 적용**

캐릭터가 대미지를 받으면 받은 대미지만큼 `CurrentHP`에서 차감하고 0보다 작으면 죽는 애니메이션을 재생하도록 만들어보자. `TakeDamage` 함수에서 직접 처리하던 대미지 기능을 액터 컴포넌트를 호출해 대신 처리하도록 한다.

<br>

1.   `ABCharacterStatComponent`에 `SetDamage` 함수 호출
2.   액터 컴포넌트에서 `CurrentHP`가 소진되면 캐릭터에게 죽음 이벤트 알림
     *   액터 컴포넌트에 델리게이트를 선언하고 캐릭터에서 바인딩 (의존성 방지!!)

<br>

**ABCharacterStatComponent.h**

```c++
DECLARE_MULTICAST_DELEGATE(FOnHPIsZeroDelegate);

class ARENABATTLE_API UABCharacterStatComponent : public UActorComponent
{
	...

public:
	void SetNewLevel(int32 NewLevel);

	void SetDamage(float NewDamage);
	float GetAttack();

	FOnHPIsZeroDelegate OnHPIsZero;
};
```

<br>

**ABCharacterStatComponent.cpp**

```c++
void UABCharacterStatComponent::SetDamage(float NewDamage)
{
	ABCHECK(nullptr != CurrentStatData);
	CurrentHP = FMath::Clamp<float>(CurrentHP - NewDamage, 0.0f, CurrentStatData->MaxHP);
	if (CurrentHP <= 0.0f)
	{
		OnHPIsZero.Broadcast();
	}
}

float UABCharacterStatComponent::GetAttack()
{
	ABCHECK(nullptr != CurrentStatData, 0.0f);
	return CurrentStatData->Attack;
}
```

<br>

**ABCharacter.cpp**

```c++
void AABCharacter::PostInitializeComponents()
{
	...
	
	CharacterStat->OnHPIsZero.AddLambda([this]()->void {
		ABLOG(Warning, TEXT("OnHPIsZero"));
		ABAnim->SetDeadAnim();
		SetActorEnableCollision(false);
	});
}

float AABCharacter::TakeDamage(float DamageAmount, FDamageEvent const& DamageEvent, AController* EventInstigator, AActor* DamageCauser)
{
	float FinalDamage = Super::TakeDamage(DamageAmount, DamageEvent, EventInstigator, DamageCauser);
	ABLOG(Warning, TEXT("Actor : %s took Damage : %f"), *GetName(), FinalDamage);

	/*if (FinalDamage > 0.0f)
	{
		ABAnim->SetDeadAnim();
		SetActorEnableCollision(false);
	}*/
	CharacterStat->SetDamage(FinalDamage);
}

float AABCharacter::TakeDamage(float DamageAmount, FDamageEvent const& DamageEvent, AController* EventInstigator, AActor* DamageCauser)
{
	float FinalDamage = Super::TakeDamage(DamageAmount, DamageEvent, EventInstigator, DamageCauser);
	ABLOG(Warning, TEXT("Actor : %s took Damage : %f"), *GetName(), FinalDamage);

	/*if (FinalDamage > 0.0f)
	{
		ABAnim->SetDeadAnim();
		SetActorEnableCollision(false);
	}*/
	CharacterStat->SetDamage(FinalDamage);

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
			ABLOG(Warning, TEXT("Hit Actor Name : %s"), *HitResult.Actor->GetName());

			// 대미지 전달 로직
			FDamageEvent DamageEvent;

			// HitResult.Actor->TakeDamage(50.0f, DamageEvent, GetController(), this);
			HitResult.Actor->TakeDamage(CharacterStat->GetAttack(), DamageEvent, GetController(), this);
		}
	}
}
```

<br>

![image](https://user-images.githubusercontent.com/93882395/223621570-995df59f-366f-4515-8afd-39c1b0f5b04f.png) 

![image-20230308134358101](C:\Users\yjoso\AppData\Roaming\Typora\typora-user-images\image-20230308134358101.png) 

>   결과 화면
>
>   캐릭터의 체력이 0이 되면 `OnHPIsZero` 함수를 호출하고 죽는 애니메이션을 재생한다.

<br>

---

# **3. 캐릭터 위젯 UI 제작**

## **3.1. 위젯 생성**

위젯 블루프린트를 생성하고 UI 애셋을 제작해보자. `UWidgetComponent` 클래스를 활용해 액터에 위젯을 부착할 수 있다.

단, `UWidgetComponent` 변수를 선언하고 컴파일을 진행하면 에러가 발생한다. 언리얼 에디터가 자동으로 개발 환경을 생성하는데, UI와 관련된 모듈은 지정되어 있지 않기 때문이다. 우리가 사용하는 언리얼 엔진 모듈인 `ArenaBattle.Build.cs `파일에서 위젯 컴포넌트 기능 `UMG`를 추가해주자.

<br>

**위젯 제작 과정**

1.   위젯 블루프린트 추가
     *   위젯 종류/크기 조절
2.   `UMG` 모듈 추가
3.   `UWidgetComponent` 변수 선언
4.   위젯 컴포넌트 생성 코드 작성
     *   컴포넌트가 캐릭터 머리 위로 오도록 위치 조정
     *   애셋의 클래스 정보를 위젯 컴포넌트의 `WidgetClass`로 등록
     *   UI 위젯을 `Screen` 모드로 지정 (항상 위젯이 스크린 응시)
     *   크기는 블루프린트 설정과 동일하게 설정

<br>

![image](https://user-images.githubusercontent.com/93882395/223631193-cf95f4cb-948e-4e87-be98-0c416ee172f7.png) 

>   프로그레스바 컨트롤 설정 윈도우

<br>



**ArenaBattle.Build.cs**

```c#
public class ArenaBattle : ModuleRules
{
	public ArenaBattle(ReadOnlyTargetRules Target) : base(Target)
	{
		...
		PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore", "UMG" });
	}
}
```

추가된 언리얼 모듈은 언리얼 빌드 툴이 컴파일 설정을 바꾸어준다. 추가된 `UMG` 모듈은 사용자가 헤더 파일을 별도로 설정하지 않고 바로 활용할 수 있도록 기본 경로로 지정된다.

<br>

**ABCharacter.h**

```c++
UCLASS()
class ARENABATTLE_API AABCharacter : public ACharacter
{
public:
	UPROPERTY(VisibleAnywhere, Category = UI)
	class UWidgetComponent* HPBarWidget;
    ...
}
```

<br>

**ABCharacter.cpp**

```c++
#include "Components/WidgetComponent.h"

AABCharacter::AABCharacter()
{
    ...
        
    HPBarWidget = CreateDefaultSubobject<UWidgetComponent>(TEXT("HPBARWIDGET"));
    
    HPBarWidget->SetupAttachment(GetMesh());
    
	HPBarWidget->SetRelativeLocation(FVector(0.0f, 0.0f, 180.0f));
	HPBarWidget->SetWidgetSpace(EWidgetSpace::Screen);

	static ConstructorHelpers::FClassFinder<UUserWidget> UI_HUD(TEXT("Game/Book/UI/UI_HPBar"));
	if (UI_HUD.Succeeded())
	{
		HPBarWidget->SetWidgetClass(UI_HUD.Class);
		HPBarWidget->SetDrawSize(FVector2D(150.0f, 50.0f));
	}
}    
```

<br>

![image](https://user-images.githubusercontent.com/93882395/223634967-cec72a24-6842-4100-b515-158ff375485f.png) 

>   결과 화면

<br>

## **3.2. UI와 데이터 연동**

캐릭터의 스텟이 변경되면 UI에 전달해 프로그레스바가 변경되는 기능을 구현해보자. 위젯 블루프린트는 `UserWidget` 클래스를 기반으로 사용된다. 

`UserWidget` 클래스를 만들고 `ABCharacterStatComponent`와 연동해 스탯이 변화할 때마다 프로그레스바의 내용을 업데이트할 수 있다. 상호 의존성을 가지지 않도록 델리게이트를 선언하자.

<br>

**ABCharacterStatComponent.h**

```c++
DECLARE_MULTICAST_DELEGATE(FOnHPChangedDelegate);

UCLASS( ClassGroup=(Custom), meta=(BlueprintSpawnableComponent) )
class ARENABATTLE_API UABCharacterStatComponent : public UActorComponent
{
...
    
public:
    void SetHP(float NewHP);
    float GetHPRatio();
    FOnHPChangedDelegate OnHPChanged;
};
```

<br>

**ABCharacterStatComponent.cpp**

```c++
void UABCharacterStatComponent::SetNewLevel(int32 NewLevel)
{
	...
        
	if (nullptr != CurrentStatData)
	{
		Level = NewLevel;
		CurrentHP = CurrentStatData->MaxHP;
		SetHP(CurrentStatData->MaxHP);
	}
	
    ...
}
void UABCharacterStatComponent::SetDamage(float NewDamage)
{
	ABCHECK(nullptr != CurrentStatData);
	SetHP(FMath::Clamp<float>(CurrentHP - NewDamage, 0.0f, CurrentStatData->MaxHP));
}

void UABCharacterStatComponent::SetHP(float NewHP)
{
	CurrentHP = NewHP;
	OnHPChanged.Broadcast();
	if (CurrentHP < KINDA_SMALL_NUMBER)
	{
		CurrentHP = 0.0f;
		OnHPIsZero.Broadcast();
	}
}

float UABCharacterStatComponent::GetHPRatio()
{
	ABCHECK(nullptr != CurrentStatData, 0.0f);

	return (CurrentStatData->MaxHP < KINDA_SMALL_NUMBER) ? 0.0f : (CurrentHP / CurrentStatData->MaxHP);
}
```

<br>

UI를 캐릭터 컴포넌트에 연결에 HP가 변할 때마다 프로그레스바를 업데이트하도록 한다. 

*   약 포인터를 사용해 컴포넌트 참조 (`TWeakObjectPtr`)
    *   해당 예제는 약 포인터의 사용이 필요하지 않지만, UI와 캐릭터가 서로 다른 액터라면 약 포인터를 사용하는 것이 바람직함
*   UI 초기화 시점 확인
    *   `BeginPlay`함수에서 `NativeConstruct` 함수 호출
        *   `PostInitializeComponents` 함수에서 발생한 명령어 반영 x (이전 호출)
    *   `NativeConstruct` 함수에서 위젯 내용을 업데이트해야 함

<br>

**ABCharacterWidget.h**

```c++
#include "ArenaBattle.h"

UCLASS()
class ARENABATTLE_API UABCharacterWidget : public UUserWidget
{
	GENERATED_BODY()
	
public:
	void BindCharacterStat(class UABCharacterStatComponent* NewCharacterStat);

protected:
	virtual void NativeConstruct() override;
	void UpdateHPWidget();

private:
	TWeakObjectPtr<class UABCharacterStatComponent> CurrentCharacterStat;

	UPROPERTY()
	class UProgressBar* HPProgressBar;
};
```

<br>

**ABCharacterWidget.cpp**

```c++
#include "ABCharacterStatComponent.h"
#include "Component/ProgressBar.h"

void UABCharacterWidget::BindCharacterStat(UABCharacterStatComponent* NewCharacterStat)
{
	ABCHECK(nullptr != NewCharacterStat);

	CurrentCharacterStat = NewCharacterStat;
	NewCharacterStat->OnHPChanged.AddUObject(this, &UABCharacterWidget::UpdateHPWidget);

}

void UABCharacterWidget::NativeConstruct()
{
	Super::NativeConstruct();
	HPProgressBar = Cast<UProgressBar>(GetWidgetFromName(TEXT("PB_HPBar")));
	ABCHECK(nullptr != HPProgressBar);
	UpdateHPWidget();
}

void UABCharacterWidget::UpdateHPWidget()
{
	if (CurrentCharacterStat.IsValid())
	{
        ABLOG(Warning, TEXT("HPRatio : %f"), CurrentCharacterStat->GetHPRatio());
		if (nullptr != HPProgressBar)
		{
			HPProgressBar->SetPercent(CurrentCharacterStat->GetHPRatio());
		}
	}
}
```

<br>

**ABCharacter.cpp**

```c++
#include "ABCharacterWidget.h"

void AABCharacter::BeginPlay()
{
	...

	auto CharacterWidget = Cast<UABCharacterWidget>(HPBarWidget->GetUserWidgetObject());
	if (nullptr != CharacterWidget)
	{
		CharacterWidget->BindCharacterStat(CharacterStat);
	}
}
```

<br>

만들어둔 위젯 블루프린트가 `ABCharacterWidget` 클래스를 상속받도록 설정한다.

*   `그래프` -> `클래스 세팅` -> `부모클래스: ABCharacterWidget`

<br>

![녹화_2023_03_09_15_02_14_287_AdobeExpress](https://user-images.githubusercontent.com/93882395/223934486-f4aa5bb4-c5c3-4aa9-ac44-44fa491fa37a.gif) 

>   결과 화면

<br>

---

# *

바보 같은 실수 모음집 ㅎㅎ...

*   게임 인스턴스를 다 만들어두고 프로젝트 설정을 따로 해주지 않아 게임에서 인스턴스 클래스를 읽어들이지 않았다. 코드 작성 전에 바로바로 적용해주자!
*   캐릭터 스텟 클래스를 액터로 만들고 코드를 작성해 나중에 전부 뜯어내야 했다. 액터와 액터 컴포넌트를 잘 구분하고 생성할 것! 특히 클래스 접두사가 뭔가 이상하다 싶으면 빠르게 재확인하기
*   UE4.21 이상 버전부터 위젯의 초기화 시점이 `BeginPlay`로 변경되었다. `PostInitializeComponents`에서 위젯을 구현하면 올바르게 동작하지 않는다.

<br>

슬슬 다시 헷갈리기 시작해 정리

*   [언리얼 클래스 명명 규칙](https://docs.unrealengine.com/5.1/ko/epic-cplusplus-coding-standard-for-unreal-engine/)

    

