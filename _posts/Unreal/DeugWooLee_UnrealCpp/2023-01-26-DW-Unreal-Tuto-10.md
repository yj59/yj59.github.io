---
title: "[이득우의 C++ 언리얼] #10 언리얼 소켓 시스템"

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
last_modified_at: 2023-01-26

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

# **1. 무기 제작**

## **1.1. 캐릭터 소켓 설정**

캐릭터가 무기를 다루려면 무기가 트랜스폼으로 배치되지 않고 메시에 착용해야 애니메이션에 맞게 무기가 움직인다.

캐릭터에 무기나 액세서리를 부착할 수 있도록 소켓 시스템을 제공한다. 캐릭터의 스켈레탈 메시를 열어 `hand_rSoket` 이름의 소켓을 확인할 수 있다.

소켓에 프리뷰 애셋을 추가하고 애니메이션을 실행해보며 소켓의 위치와 회전값을 조정한다. 소켓의 설정을 완료하면 캐릭터에 무기를 부착해보자.

1.   무기 애셋의 레퍼런스 복사 후 스켈레탈 메시 컴포넌트에 로딩
     *   무기 애셋은 전부 스켈레탈 메시이기 때문임
2.   `SetupAttachment` 함수에 소켓 이름을 인자로 넘김
     *   소켓 위치를 기준으로 트랜스폼 자동 설ㅈ어

<br>

 **ABCharacter.h**

```c++
UCLASS()
class ARENABATTLE_API AABCharacter : public ACharacter
{
public:
	UPROPERTY(VisibleAnywhere, Category = Weapon)
	USkeletalMeshComponent* Weapon;
	...
}
```

<br>

**ABCharacter.cpp**

```c++
AABCharacter::AABCharacter()
{
FName WeaponSocket(TEXT("hand_rSocket"));
	if (GetMesh()->DoesSocketExist(WeaponSocket))
	{
		Weapon = CreateDefaultSubobject<USkeletalMeshComponent>(TEXT("WEAPON"));
		static ConstructorHelpers::FObjectFinder<USkeletalMesh> SK_WEAPON(TEXT("/Game/InfinityBladeWeapons/Weapons/Blade/Swords/Blade_BlackKnight/SK_Blade_BlackKnight.SK_Blade_BlackKnight"));
		if (SK_WEAPON.Succeeded())
		{
			Weapon->SetSkeletalMesh(SK_WEAPON.Object);
		}
		Weapon->SetupAttachment(GetMesh(), WeaponSocket);
	}
}
```

<br>

![image](https://user-images.githubusercontent.com/93882395/223317150-a0c6389f-ab37-4535-9a67-e07d3c2a93e5.png) 

>   컴파일을 마치면 플레이어에 무기가 부착된다.

<br>

## 1.2. 무기 액터 제작

필요에 따라 무기를 바꿀 수 있도록 무기를 액터로 분리하자. 무기 액터의 클래스명은 `ABWeapon`으로 한다. 해당 예제에서는 캐릭터에게 충돌 역할을 부여했으므로 무기 액터는 충돌 설정을 `NoCollision`으로 지정한다.

무기 액터를 제작했으면 캐릭터가 액터를 손에 잡도록 한다.

기존 캐릭터의 무기 부착 코드를 제거한 후 `BeginPlay` 함수에서 무기 액터를 생성하고 부착시키자.

<br>

**ABWeapon.h**

```c++
UCLASS()
class ARENABATTLE_API AABWeapon : public AActor
{
	...

public:	
	UPROPERTY(VisibleAnywhere, Category = Weapon)
	USkeletalMeshComponent* Weapon;
};
```

<br>

**ABWeapon.cpp**

```c++
AABWeapon::AABWeapon()
{
 	...

	Weapon = CreateDefaultSubobject<USkeletalMeshComponent>(TEXT("WEAPON"));
	RootComponent = Weapon;

	static ConstructorHelpers::FObjectFinder<USkeletalMesh> SK_WEAPON(TEXT("/Game/InfinityBladeWeapons/Weapons/Blade/Swords/Blade_BlackKnight/SK_Blade_BlackKnight.SK_Blade_BlackKnight"));
	if (SK_WEAPON.Succeeded())
	{
		Weapon->SetSkeletalMesh(SK_WEAPON.Object);
	}
	Weapon->SetCollisionProfileName(TEXT("NoCollision"));
}
```

<br>

**ABCharacter.cpp**

```c++
#include "ABWeapon.h"

// 1.1. 무기 생성/장착 코드 삭제

void AABCharacter::BeginPlay()
{
	Super::BeginPlay();

	FName WeaponSocket(TEXT("hand_rSocket"));
	auto CurWeapon = GetWorld()->SpawnActor<AABWeapon>(FVector::ZeroVector, FRotator::ZeroRotator);
	if (nullptr != CurWeapon)
	{
		CurWeapon->AttachToComponent(GetMesh(), FAttachmentTransformRules::SnapToTargetNotIncludingScale, WeaponSocket);
	}
}
```

*   `SpawnActor(생성할 액터의 클래스, 앞으로 생성할 위치 및 회전)` 

    *   월드에 새로운 액터 생성

    *   월드 명령어이므로 `GetWorld`로 월드 포인터를 가져와 해당 함수 실행

        

<br>

---

# **2. 아이템 습득 기능**

## **2.1. 아이템 상자 제작**

아이템 박스 크기를 참고해 아이템박스 액터에 콜리전 컴포넌트의 크기와 스태틱메시 컴포넌트의 애셋을 지정한다. 

<br>

**ABItemBox.h**

```c++
class ARENABATTLE_API AABItemBox : public AActor
{
...
	
public:
	UPROPERTY(VisibleAnywhere, Category = Box)
	UBoxComponent Trigger;

	UPROPERTY(VisibleAnywhere, Category = Box)
	UBoxCompoenet* Box;
};

```

<br>

**ABItemBox.cpp**

```c++
AABItemBox::AABItemBox()
{
    PrimaryActorTick.bCanEverTick = false;

    Trigger = CreateDefaultSubobject<UBoxComponent>(TEXT("TRIGGER"));
    Box = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("BOX"));

    RootComponent = Trigger;
    Box->SetupAttachment(RootComponent);

    Trigger->SetBoxExtent(FVector(40.0f, 42.0f, 30.0f));
    static ConstructorHelpers::FObjectFinder<UStaticMesh> SM_BOX(TEXT("/Game/InfinityBladeGrassLands/Environments/Breakables/StaticMesh/Box/SM_Env_Breakables_Box1.SM_Env_Breakables_Box1"));
    
    if (SM_BOX.Succeeded())
    {
        Box->SetStaticMesh(SM_BOX.Object);
    }

    Box->SetRelativeLocation(FVector(0.0f, -3.5f, -30.0f));
}
```

*   `SetBoxExtent` : 박스 콜리전 컴포넌트의 Extend 값으로 전체 박스 영역 크기의 절반 입력

<br>

박스 설치와 초기 설정을 완료했으면 폰이 아이템을 획득하도록 아이템 상자에 오브젝트 채널을 추가해보자. 프로젝트 설정에서 ItemBox의 콜리전과 프리셋을 설정한다. 이때, 캐릭터에게 아이템 이벤트를 실행시키기 위해 ItemBox의 콜리전과 프리셋은 모두 무시로 하되, `ABCharacter`에 대한 충돌 속성만 겹침으로 변경해야 한다.

<br>

새로운 프리셋을 박스 컴포넌트에 설정하고, 박스 컴포넌트에서 캐릭터를 감지할 때 관련 행동을 구현한다.

박스 컴포넌트는 `Overlap` 이벤트를 처리할 수 있도록 `OnComponentBeginOverlap` 델리게이트가 선언되어 있다!

<br>

**ABItemBox.h**

```c++
UCLASS()
class ARENABATTLE_API AABItemBox : public AActor
{

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

	virtual void PostInitializeComponents() override;

private:
	UFUNCTION()
		void OnCharacterOverlap(UPrimitiveComponent* OverlappedComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult);
		...
}
```

<br>

**ABItemBox.cpp**

````c++
AABItemBox::AABItemBox()
{
	...
	
	Trigger->SetCollisionProfileName(TEXT("ItemBox"));
    Box->SetCollisionProfileName(TEXT("NoCollision"));
}

void AABItemBox::PostInitializeComponents()
{
    Super::PostInitializeComponents();
    Trigger->OnComponentBeginOverlap.AddDynamic(this, &AABItemBox::OnCharacterOverlap);
}

void AABItemBox::OnCharacterOverlap(UPrimitiveComponent* OverlappedComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
    ABLOG_S(Warning);
}
````

<br>

![image](https://user-images.githubusercontent.com/93882395/223409919-4ae2079e-a801-4e91-b607-70861376648b.png) 

>   결과 화면
>
>   캐릭터가 아이템 박스를 통과할 때마다 오버랩 델리게이트와 바인딩된 함수가 실행된다.

<br>

## **2.2. 아이템 습득**

아이템 상자를 통과하면 플레이어에게 아이템을 쥐어주는 기능을 구현해보자.

1.    아이템 상자에 클래스 정보를 저장할 속성 추가
     *   `TSubclassof` : 특정 클래스와 상속받은 클래스들로 목록 한정
     *   구현부 생성자 코드에 무기아이템 속성에 대한 기본 클래스값 지정
2.   플레이어가 아이템 상자 영역에 들어왔을 때 아이템 생성
     *   캐릭터에 무기를 장착시키는 멤버 함수 선언
         *   현재 캐릭터에 무기가 없으면 `hand_rSocket`에 무기 장착
         *   무기 액터의 소유자를 캐릭터로 변경
3.   `BeginPlay`에서 시작할 때 무지 액터를 장착시킨 로직 삭제
4.   상자에 `Overlap` 이벤트가 발생하면 아이템 상자에 설정된 클래스 정보로부터 무기 생성

<br>

**ABItemBox.h**

```c++
class ARENABATTLE_API AABItemBox : public AActor
{
public:
	UPROPERTY(EditInstanceOnly, Category = Box)
	TSubclassOf<class AABWeapon> WeaponItemClass;
	
	...
}
```

<br>

**ABItemBox.cpp**

```c++
#include "ABWeapon.h"
#include "ABCharacter.h"

AABItemBox::AABItemBox()
{
    WeaponItemClass = AABWeapon::StaticClass();
    
    ...
}

void AABItemBox::OnCharacterOverlap(UPrimitiveComponent* OverlappedComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
    ABLOG_S(Warning);

    auto ABCharacter = Cast<AABCharacter>(OtherActor);
    ABCHECK(nullptr != ABCharacter);
    if (nullptr != ABCharacter && nullptr != WeaponItemClass)
    {
        if (ABCharacter->CanSetWeapon())
        {
            auto NEwWeapon = GetWorld()->SpawnActor<AABWeapon>(WeaponItemClass, FVector::ZeroVector, FRotator::ZeroRotator);
            ABCharacter->SetWeapon(NEwWeapon);
        }
        else
        {
            ABLOG(Warning, TEXT("%s can't equip weapon currently."), *ABCharacter->GetName());
        }
    }
}
```

<br>

**ABCharacter.h**

```c++
class ARENABATTLE_API AABCharacter : public ACharacter
{
public:
	bool CanSetWeapon();
	void SetWeapon(class AABWeapon* NewWeapon);
    UPROPERTY(VisibleAnywhere, Category = Weapon)
	class AABWeapon* CurrentWeapon;
	
	...
}
```

<br>

**ABCharacter.cpp**

```c++
bool AABCharacter::CanSetWeapon()
{
	return (nullptr == CurrentWeapon);
}

void AABCharacter::SetWeapon(AABWeapon* NewWeapon)
{
	ABCHECK(nullptr != NewWeapon && nullptr == CurrentWeapon);
	FName WeaponSocket(TEXT("hand_rSocket"));
	if (nullptr != NewWeapon)
	{
		NewWeapon->AttachToComponent(GetMesh(), FAttachmentTransformRules::SnapToTargetNotIncludingScale, WeaponSocket);
		NewWeapon->SetOwner(this);
		CurrentWeapon = NewWeapon;
	}
}
```

<br>

![녹화_2023_03_07_20_55_29_917_AdobeExpress](https://user-images.githubusercontent.com/93882395/223415548-bcf0c99e-150f-4af4-b329-0eef504091f2.gif) 

>   결과 화면
>
>   아이템 박스에 닿으면 무기를 장착하고, 이미 무기를 장착한 상태라면 대응 로그가 출력된다.

<br>

---

## **2.3. 이펙트 구현**

캐릭터가 아이템을 습득하면 상자가 이펙트를 재생하고 사라지는 기능을 구현한다.

1.   상자 액터에 파티클 컴포넌트 추가

2.   이펙트 애셋 레퍼런스를 파티클 컴포넌트 템플릿으로 지정

3.   멤버 함수를 추가하고 파티클 컴포넌트 시스템에서 제공하는 `OnSystemFinishied` 델리게이트에 추가

     *   이펙트 생성이 종료되면 아이템 상자 제거

     *   `OnSystemFinishied` 델리게이트는 다이나믹 형식이므로 바인딩할 대상 멤버 함수에 UFUNCTION 매크로 선언

4.   캐릭터가 아이템을 두 번 습득하지 못하도록 함

     *   이펙트 재생 도중에는 액터 충돌 기능 제거
     *   액터가 제거될 때까지 박스 스태틱메시 숨김

<br>

**ABItemBox.h**

```c++
class ARENABATTLE_API AABItemBox : public AActor
{
public:
	...
	
	UPROPERTY(EditInstanceOnly, Category = Box)
	UParticleSystemComponent* Effect;
    
private:
	UFUNCTION()
	void OnEffectFinished(class UParticleSystemComponent* PSystem);
};
```

<br>

**ABItemBox.cpp**

```c++
AABItemBox::AABItemBox()
{
 	...
 	
    Effect = CreateDefaultSubobject<UParticleSystemComponent>(TEXT("EFFECT"));
    
    Effect->SetupAttachment(RootComponent);
    
    static ConstructorHelpers::FObjectFinder<UParticleSystem> P_CHESTOPEN(TEXT("/Game/InfinityBladeGrassLands/Effects/FX_Treasure/Chest/P_TreasureChest_Open_Mesh.P_TreasureChest_Open_Mesh"));
    if (P_CHESTOPEN.Succeeded())
    {
        Effect->SetTemplate(P_CHESTOPEN.Object);
        Effect->bAutoActivate = false;
    }
    
    ...
}

void AABItemBox::OnCharacterOverlap(UPrimitiveComponent* OverlappedComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
    ABLOG_S(Warning);

    auto ABCharacter = Cast<AABCharacter>(OtherActor);
    ABCHECK(nullptr != ABCharacter);
    if (nullptr != ABCharacter && nullptr != WeaponItemClass)
    {
        if (ABCharacter->CanSetWeapon())
        {
            auto NEwWeapon = GetWorld()->SpawnActor<AABWeapon>(WeaponItemClass, FVector::ZeroVector, FRotator::ZeroRotator);
            ABCharacter->SetWeapon(NEwWeapon);

            // 이펙트 도중 충돌제거/박스 스태틱메시 숨김
            // 이펙트 재생 종료시 아이템 제거 함수 호출
            Effect->Activate(true);
            Box->SetHiddenInGame(true, true);
            SetActorEnableCollision(false);
            Effect->OnSystemFinished.AddDynamic(this, &AABItemBox::OnEffectFinished);
        }
        else
        {
            ABLOG(Warning, TEXT("%s can't equip weapon currently."), *ABCharacter->GetName());
        }
    }
}

void AABItemBox::OnEffectFinished(UParticleSystemComponent* PSystem)
{
    Destroy();
}
```

<br>

부모 클래스를 `ABWeapon`으로 하는 새 블루프린트 클래스를 생성한다. 부모 클래스 선택시 모든 클래스 탭을 눌러 사용자 지정 클래스를 선택할 수 있다.

`ABWeapon`이 부모 클래스이므로 사전에 지정해두었던 무기가 기본 블루프린트 형식으로 출력된다. 블루프린트를 열고 스켈레탈 메시에서 도끼로 무기를 변경해보자.

언리얼 에디터에서 상자의 `WeaponClass` 설정을 방금 설정한 블루프린트 이름으로 바꾸어준다.

![image](https://user-images.githubusercontent.com/93882395/223420291-a78206d9-781a-4d3f-bbca-66a7bea0faf9.png) 

<br>

![녹화_2023_03_07_21_20_33_814_AdobeExpress](https://user-images.githubusercontent.com/93882395/223420769-a1bf002e-5b9a-4323-b09a-cb0001d44e7d.gif) 

>   결과 화면
>
>   캐릭터가 아이템 상자를 습득하면 도끼가 부착된다.
