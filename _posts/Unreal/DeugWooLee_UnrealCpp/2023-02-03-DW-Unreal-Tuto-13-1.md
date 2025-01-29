---
title: "[이득우의 C++ 언리얼] #13-1 프로젝트 설정"

date: 2023-02-03 00:00:00 +0900
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

# **1. 프로젝트 설정**

## **1.1. 소스 파일 정리**

언리얼 소스 코드 구조

*   `Classes` : 언리얼 오브젝트 관련 헤더
*   `Public` : 외부에 공개하는 선언 파일
*   `Private` : 외부에 공개하지 않는 정의 파일

<br>

![image](https://user-images.githubusercontent.com/93882395/224456762-3d15734e-18a8-4845-aafd-cbc876c3866a.png) 

![image](https://user-images.githubusercontent.com/93882395/224456825-12a14584-6fbc-47c2-89ef-3dc09c324d1f.png) 

소스 코드 폴더에 Public과 Private 폴더를 만들어 .h 파일과 .cpp 파일을 옮겨보자. 솔루션을 재생성하면 프로젝트 구성이 바뀐 것을 확인할 수 있다.

*   `*.Target.cs` : 게임 빌드와 에디터 빌드 설정 지정

<br>

## **1.2. 모듈 추가**

주 게임 모듈 외에 다른 모듈을 게임 프로젝트에 추가하고 로직을 분리해 관리할 수 있다.

단, 추가 모듈은 자동으로 생성되지 않으므로 필요한 요소를 직접 추가해야 한다.

*   **모듈 폴더와 빌드 설정 파일**: 모듈 폴더와 모듈명으로 된 Build.cs 파일
*   **모듈의 정의 파일**: 모듈명으로 된 .cpp 파일

솔루션파일을 재생성한 후 VS에서 추가한 모듈을 빌드할 수 있도록 `ArenaBattle.Target.cs` 파일과 `ArenaBattleEditor.Target.cs` 파일 정보를 수정해야 한다. 모듈 설정까지 추가하고 나면 빌드 명령 시 새로운 모듈을 컴파일한다.

<br>

**모듈 추가 과정**

1.   모듈을 생성하기 위한 필수 파일 생성 후 Source 폴더에 복사
2.   솔루션 파일 재생성
3.   `ArenaBattle.Target.cs`와 `ArenaBattleEditor.Target.cs` 정보 수정 후 빌드
4.   새 DLL 파일에 모듈 정보 기입
5.   `ArenaBattle` 모듈의 의존성에 `ArenaBattleSetting` 기입
6.   `Object`를 부모로 하는 `ABCharacterSetting` 클래스 생성
     *   ArenaBattleSetting 모듈에 속한 오브젝트 필요
     *   오브젝트의 모듈을 `ArenaBattleSetting`으로 선택
7.   프로젝트 재생성 후 빌드

<br>

**ArenaBattle.Target.cs**

```csharp
public class ArenaBattleTarget : TargetRules
{
	public ArenaBattleTarget(TargetInfo Target) : base(Target)
	{
		...

		ExtraModuleNames.AddRange( new string[] { "ArenaBattle", "ArenaBattleSetting" } );
	}
}
```

<br>

**ArenaBattleEditor.Target.cs**

```csharp
public class ArenaBattleEditorTarget : TargetRules
{
	public ArenaBattleEditorTarget(TargetInfo Target) : base(Target)
	{
		...

		ExtraModuleNames.AddRange( new string[] { "ArenaBattle", "ArenaBattleSetting" } );
	}
}
```

<br>

**ArenaBattle.uproject**

```c++
{
	"FileVersion": 3,
	"EngineAssociation": "4.27",
	"Category": "",
	"Description": "",
	"Modules": [
		{
			"Name": "ArenaBattleSetting",
			"Type": "Runtime",
			// 다른 모듈모다 먼저 로딩되도록 함
			"LoadingPhase":  "PreDefault"
		},
		{
			"Name": "ArenaBattle",
			"Type": "Runtime",
			"LoadingPhase": "Default",
			"AdditionalDependencies": [
				"Engine",
				"UMG",
				"AIModule",
				// ArenaBattleSetting에 대한 의존성 부여
				"ArenaBattleSetting"
			]
		}
	]
}
```

<br>

![image](https://user-images.githubusercontent.com/93882395/224459769-dae18e89-9cce-4519-9c22-7be82a26c116.png) 

>   추가 모듈 로딩 결과

<br>

---

# **2. INI 설정**

## **2.1. 오브젝트 기본값 지정**

지금까지 애셋 정보를 생성자 코드로 지정했지만, 애셋이 변경되면 코드를 다시 만들고 컴파일해야 한다. 애셋의 효율적인 관리를 위해 새로 추가한 `ABCharacterSetting` 모듈에 앞으로 사용할 애셋의 목록을 보관하자. 애셋의 경로는 `FSoftObjectPath` 클래스를 활용해 저장한다.

언리얼 오브젝트의 기본값을 유연하게 관리하도록 INI 파일에서 기본 속성 값을 지정하는 기능을 제공한다. 애셋 경로 정보를 참조해 프로그램을 로딩할 수 있다. 기본값을 INI 파일에서 불러들이도록 설정하면 언리얼 오브젝트를 초기화할 때 해당 속성의 값을 INI 파일에서 읽어온다.

기본값을 지정해 생성하는 객체를 클래스 기본 객체라고 한다. 언리얼 엔진 초기화 시 모든 클래스 기본 객체가 메모리에 올라간다. 메모리에 올라간 클래스 기본 객체는 엔진 종료 직전까지 상주하므로 항상 객체를 활용할 수 있다.

<br>

**INI를 활용한 기본값 지정 과정**

1.   `UCLASS` 매크로에 `config` 키워드 추가
2.   불러들일 INI 파일명 지정
3.   `UPROPERTY` 속성에 `config` 키워드 선언
4.   `ArenaBattle.Build.cs` 파일에 참조할 모듈 목록 추가
     *   .cpp 파일에만 모듈을 사용하므로 `PrivateDependencyModule` 항목에 추가
5.   `ABCharacter`에서 `ABCharacterSetting` 의 애셋 목록을 얻어오는 코드 작성
     *   ArenaBattleSetting 모듈이 ArenaBattle 모듈보다 빨리 로딩되므로 `GetDefault` 함수를 사용해 애셋 정보 불러오기 가능
         *   `GetDefault` : 클래스 기본 객체 가져옴

<br>

**ABCharacterSetting.h**

```c++
#include "CoreMinimal.h"
#include "UObject/NoExportTypes.h"
#include "ABCharacterSetting.generated.h"

UCLASS(config=ArenaBattle)
class ARENABATTLESETTING_API UABCharacterSetting : public UObject
{
	GENERATED_BODY()
public:
	UABCharacterSetting();

	UPROPERTY(config)
	TArray<FSoftObjectPath> CharacterAssets;
};
```

<br>

**ABCharacterSetting.cpp**

```c++
UABCharacterSetting::UABCharacterSetting()
{

}
```

<br>

**ArenaBattle.Build.cs**

```c#
public class ArenaBattle : ModuleRules
{
	public ArenaBattle(ReadOnlyTargetRules Target) : base(Target)
	{
		...
		
		PrivateDependencyModuleNames.AddRange(new string[] { "ArenaBattleSetting" });
	}
}
```

<br>

**ABCharacter.cpp**

```c++
#include "ABCharacterSetting.h"

AABCharacter::AABCharacter()
{
    ...
        
	auto DefaultSetting = GetDefault<UABCharacterSetting>();
	if (DefaultSetting->CharacterAssets.Num() > 0)
	{
		for (auto CharacterAsset : DefaultSetting->CharacterAssets)
		{
			ABLOG(Warning, TEXT("Character Asset : %s"), *CharacterAsset.ToString());
		}
	}
}
```

<br>

![image](https://user-images.githubusercontent.com/93882395/224460906-d2702e4d-6e86-476a-b4bb-fa56ff1522dd.png) 

>   INI파일 설정값 로딩 결과

<br>

## 2.2. 캐릭터 애셋 로딩

애셋 경로를 통해 NPC가 생성될 때 랜덤하게 목록 중 하나를 골라 캐릭터 애셋을 로딩하는 기능을 만들 수 있다. `ABGameInstance` 클래스에 `FStreamableManager` 클래스를 멤버 변수로 선언해 애셋 로딩 명령을 내린다. 

*   `FStreamableManager` : 게임 진행 도중 비동기 방식으로 애셋 로딩
    *   프로젝트에서 하나만 활성화하는 것을 권장
    *   `AsyncLoad` : 비동기 방식 에셋 로딩 명령




**👀싱글톤 설정**<br>게임 인스턴스는 싱글톤처럼 동작해 `FStreamableManager` 클래스를 멤버 변수로 선언했지만 별도로 싱글톤으로 지정할 수도 있다.<br>`프로젝트 설정` -> `Default Classes` 섹션의 고급 섹션에서 설정 가능하다!
{: .notice--primary}

<br>

**ABGameInstance.h**

```
#include "Engine/StreamableManager.h"

class ARENABATTLE_API UABGameInstance : public UGameInstance
{
	GENERATED_BODY()

public:
	...

	FStreamableManager StreamableManager;
}
```

<br>

**ABCharacter.h**

```c++
class ARENABATTLE_API AABCharacter : public ACharacter
{
private:
	...

	void OnAssetLoadComplete();
private:
	FSoftObjectPath CharacterAssetToLoad = FSoftObjectPath(nullptr);
	TSharedPtr<struct FStreamableHandle> AssetStreamingHandle;
}
```

<br>

**ABCharacter.cpp**

```c++
#include "ABGameInstance.h"

void AABCharacter::BeginPlay()
{
	Super::BeginPlay();

	/*auto CharacterWidget = Cast<UABCharacterWidget>(HPBarWidget->GetUserWidgetObject());
	if (nullptr != CharacterWidget)
	{
		CharacterWidget->BindCharacterStat(CharacterStat);
	}*/
	if (!IsPawnControlled())
	{
		auto DefaultSetting = GetDefault<UABCharacterSetting>();
		int32 RandIndex = FMath::RandRange(0, DefaultSetting->CharacterAssets.Num() - 1);
		CharacterAssetToLoad = DefaultSetting->CharacterAssets[RandIndex];

		auto ABGameInstance = Cast<UABGameInstance>(GetGameInstance());
		if (nullptr != ABGameInstance)
		{
			AssetStreamingHandle = ABGameInstance->StreamableManager.RequestAsyncLoad(CharacterAssetToLoad, FStreamableDelegate::CreateUObject(this, &AABCharacter::OnAssetLoadCompleted));
		}
	}
}

void AABCharacter::OnAssetLoadCompleted()
{
	USkeletalMesh* AssetLoaded = Cast<USkeletalMesh>(AssetStreamingHandle->GetLoadedAsset());
	AssetStreamingHandle.Reset();
	if (nullptr != AssetLoaded)
	{
		GetMesh()->SetSkeletalMesh(AssetLoaded);
	}
}
```

<br>

![녹화_2023_03_11_13_01_27_409_AdobeExpress](https://user-images.githubusercontent.com/93882395/224463868-a78ea6d4-29fa-4c25-836c-685c748edf43.gif) 

>   결과 화면

<br>

---

