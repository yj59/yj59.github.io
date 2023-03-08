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

# 1. 액셀 데이터 활용

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

# 2. 액터 컴포넌트 제작

## 2.1. 액터 컴포넌트 클래스

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

## 2.2. 캐릭터 대미지 적용

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

# 3. 캐릭터 위젯 UI 제작









---

# *

바보 같은 실수 모음집 ㅎㅎ...

*   게임 인스턴스를 다 만들어두고 프로젝트 설정을 따로 해주지 않아 게임에서 인스턴스 클래스를 읽어들이지 않았다. 코드 작성 전에 바로바로 적용해주자!
*   캐릭터 스텟 클래스를 액터로 만들고 코드를 작성해 나중에 전부 뜯어내야 했다. 액터와 액터 컴포넌트를 잘 구분하고 생성할 것! 특히 클래스 접두사가 뭔가 이상하다 싶으면 빠르게 재확인하기

<br>

슬슬 다시 헷갈리기 시작해 정리

*   [언리얼 클래스 명명 규칙](https://docs.unrealengine.com/5.1/ko/epic-cplusplus-coding-standard-for-unreal-engine/)

    

