---
title: "[이득우의 C++ 언리얼] #4 게임플레이 프레임워크"

date: 2023-01-06 00:00:00 +0900
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

# 1. 게임 모드

* 게임플레이 프레임워크: 원활한 게임 플레이를 위해 설계 및 관리 시스템 제공
  * 게임 장르, 멀티 플레이 등 다양한 구조 내재
  * 대표적인 프레임워크: 게임 모드, 폰
    * `게임 모드 `: 게임의 규칙 관리
    * `폰 `: 플레이어가 조종하는 액터

<br>

**게임 모드 설정**

1. `파일` $\rightarrow$ `새로운 c++ 클래스...`

   * 새 게임 모드(`ABGameMode`)와 폰(`ABPawn`) 생성

2. 게임 모드 적용

   * `세팅` $\rightarrow$`월드 세팅` $\rightarrow$ 게임모드 오버라이드 변경

     <img src="https://user-images.githubusercontent.com/93882395/221470536-71c3dcb3-b814-4c19-988d-62a13487c683.png" alt="image" style="zoom: 80%;" /> 

     > Step2 레벨에 등장하는 폰이 `DefaultPawn`으로 변경됨

<br>

**GameMode가 ABPawn을 참조하도록 수정**

1. `ABGameMode.h` $\rightarrow$ `CoreMinimal.h` 변경
   * 로그 매크로/`EngineMinimal.h` 참조하도록 `ArenaBattle.h` 선언
   
2. 게임 모드의 `DefaultPawn` 속성으로 `ABPawn` 지정
   
   * 폰, 플레이어 컨트롤러 등 클래스를 만들 경우 클래스 생성 후 게임 모드의 모듈에 선언 필요
   
   * 액터 생성x, `ABPawn`의 클래스 정보 지정
   
     => 미리 폰을 만들어두는 것보다 플레이어가 입장할 때마다 클래스 정보를 기반으로 폰을 생성하는 것이 합리적임!
   
   

<br>

**ABGameMode.h**

```c++
#pragma once

//헤더 파일 변경
#include "ArenaBattle.h"
#include "GameFramework/GameModeBase.h"
#include "ABGameMode.generated.h"

UCLASS()
class ARENABATTLE_API AABGameMode : public AGameModeBase
{
	GENERATED_BODY()

//ABGameMode 생성자 추가
public:
	AABGameMode();
	
};
```

<br>

**ABGameMode.cpp**

```c++
#include "ABGameMode.h"
#include "ABPawn.h"

AABGameMode::AABGameMode()
{
    //클래스 정보는 언리얼 헤더에 의해 자동으로 생성된다!
	//StaticClass 함수를 호출해 ABPawn의 정보 가져옴
	DefaultPawnClass = AABPawn::StaticClass();
}
```

<br>

---

# 2. 플레이어 컨트롤러

## 2.1. 플레이어 입장

* `플레이어 컨트롤러` : 현실 세계의 플레이어를 대변하는 액터(비가시)
  * 플레이어와 상호작용 후 폰 조종
* `폰(Pawn)` : 게임 세계에 배치되며 플레이어 컨트롤러에 의해 조종되는 액터

* `빙의(Possess)` : 플레이어가 플레이어 컨트롤러를 통해 폰을 조종하는 행위
* `로그인(Login)` : 플레이어가 게임에 입장

<br>

**게임 시작 과정**

1. 플레이어 컨트롤러 생성
2. 플레이어 폰 생성
3. 플레이어 컨트롤러가 폰에 빙의(로그인 과정)
4. 게임 시작

<br>

**ABGameMode.cpp**

```c++
#include "ABGameMode.h"
#include "ABPawn.h"
//플레이어 컨트롤러 헤더 선언
#include "ABPlayerController.h"

AABGameMode::AABGameMode()
{
	DefaultPawnClass = AABPawn::StaticClass();
    //생성자에서 플레이어 컨트롤러 클래스의 속성 변경
	PlayerControllerClass = AABPlayerController::StaticClass();
}
```

<br>

<img src="https://user-images.githubusercontent.com/93882395/221495588-9b3a61af-487b-4400-a8b9-08f6f56cfded.png" alt="image" style="zoom: 80%;" /> 

> 게임 모드에 적용된 결과 화면

<br>

## 2.2. 플레이어 초기화

로그인 후 게임 모드에서 `PostLogin` 함수가 호출된다.

* 폰 생성, 플레이어 컨트롤러 빙의
  * `PostInitializeComponents` : 폰, 픍레이어 컨트롤러가 생성되는 시점 확인
  * `(플레이어 컨트롤러)Possess`, `(폰)PossessedBy` : 빙의 진행 시점 확인

<br>

**플레이어 설정 진행 과정(`PostLogin`함수에 로그 찍기!)**

<br>

**ABGameMode.h**

```c++
public:
	AABGameMode();
	
	virtual void PostLogin(APlayerController* NewPlayer) override;
```

<br>

**ABGameMode.cpp**

```c++
void AABGameMode::PostLogin(APlayerController* NewPlayer)
{
	ABLOG(Warning, TEXT("PostLogin Begin"));
	Super::PostLogin(NewPlayer);
	ABLOG(Warning, TEXT("PostLogin End"));
}
```

<br>

**ABPlayerController.h**

```c++
#include "ArenaBattle.h"
...
class ARENABATTLE_API AABPlayerController : public APlayerController
{
	GENERATED_BODY()

public:
	virtual void PostInitializeComponents() override;
	virtual void OnPossess(APawn* aPawn) override;
};
```

🐣**Possess함수 오류 해결**<br>Possess 함수를 컴파일하면 오류가 발생한다! 언리얼 4,22 버전부터 Possess 함수로 상속을 받을 수 없다고 한다.<br>`Possess(APawn* aPawn)`으로 함수명을 바꾸어 컴파일하면 해결된다.<br>UnPossess도 `OnUnProsess()`로 `On` 접두사를 붙여 작성하자.
{: .notice--primary}



<br>

**ABPlayerController.cpp**

```c++
#include "ABPlayerController.h"

void AABPlayerController::PostInitializeComponents()
{
	Super::PostInitializeComponents();
	ABLOG_S(Warning);
}

void AABPlayerController::OnPossess(APawn* aPawn)
{
	ABLOG_S(Warning);
	Super::OnPossess(aPawn);
}
```

<br>

**ABPawn.h**

```c++
#pragma once

#include "ArenaBattle.h"
#include "GameFramework/Pawn.h"
#include "ABPawn.generated.h"

UCLASS()
class ARENABATTLE_API AABPawn : public APawn
{
...

public:	
...
	virtual void PostInitializeComponents() override;
	virtual void PossessedBy(AController* NewController) override;
};
```


<br>

**ABPawn.cpp**

```c++
...
    
void AABPawn::PostInitializeComponents()
{
	Super::PostInitializeComponents();
	ABLOG_S(Warning);
}

void AABPawn::PossessedBy(AController* NewController)
{
	ABLOG_S(Warning);
	Super::PossessedBy(NewController);
}
...
```

<br>

![image](https://user-images.githubusercontent.com/93882395/221504537-eb625fd0-cdaf-44c3-9420-0453f09594ac.png) 

> **로그 결과 화면**
>
> 플레이어 컨트롤러 생성 후 `PostLogin Begin`를 거치면 폰이 생성된다. 
>
> 빙의를 마치면 `PostLogin End` 로그가 출력된다! 플레이어의 초기 설정이 완료되었다는 뜻이다.

<br>

새 액터를 레벨에 배치한 후 디테일 윈도우의 `Auto Possess Player`항목을 `Player0`*(로컬 플레이어)*로 설정하면 별도의 코드 연결 없이도 플레이어 컨트롤러가 빙의할  수 있다.

액터의 폰 설정에 플레이어가 빙의하도록 속성을 변경한 후 플레이 버튼을 누르면 게임 모드가 `ABPawn` 대신 해당 액터에 빙의하도록 명령을 내린다.
