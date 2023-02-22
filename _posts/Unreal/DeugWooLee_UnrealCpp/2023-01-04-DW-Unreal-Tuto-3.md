---
title: "[이득우의 C++ 언리얼] #3 움직이는 액터 제작"

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
last_modified_at: 2023-01-05

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

# **1. 개발 환경 점검**



## **1.1. 로깅 환경**

<br>

언리얼에서는 기능별 로그를 구분할 수 있도록 서로 다른 카테고리를 선언해 로그를 남긴다. 

로그를 확인해 에디터 초기화부터 개발자 작업까지의 엔진 동작 수행 과정을 확인할 수 있다.

*   **로깅 매크로**: `UE_LOG(카테고리, 로깅 수준, 형식 문자열, 인자 ...)`

    *   파일/출력 로그 윈도우에서 로그 확인
        *   파일: `{프로젝트명}/Saved/Logs/{프로젝트명}.log`
        *   출력 로그 윈도우: `창` -> `개발자 툴` -> `출력 로그`

*   **로깅 수준은** 출력 로그 윈도우에 표시

    *   <span style="background-color: #ffffff !important">메시지*(Log)*</span>, <span style="background-color: #fff5b1  ! important">경고*(Warning)*</span>, <span style="background-color: #ffdce0  ! important">에러*(Error)*</span>
        *   에디터 개인 설정에서 로그 색상 변경
    *   연결된 안드로이드 기기 로그도 에디터 윈도우에 표시 설정 가능

*   사용자가 원하는 로깅 수준과 카테고리에 해당하는 정보만 출력하도록 **로그 필터** 기능 제공

*   **형식 문자열**: 로그 매크로에서 형식 문자열 기능 지원 *(`printf`와 유사)*

    *   문자열 정의시 `TEXT` 매크로 사용 권장
    *   문자열 관리 클래스 `FString` 제공
        *   `FString` 타입의 문자열 정보를 가져오려면 `*` 연산자 사용 필요
    
    
    *eg. `FString::Printf(TEXT("Actor Name: %s, ID: %d, LocationX %.3f"), *GetName(), ID, GetActorLocation().X);`*

<br>

**로깅 매크로 설정 (로그 카테고리 선언)**

게임 개발에 필요한 로그 카테고리를 직접 선언할 수 있다.

로그 카테고리 선언을 위해 헤더 파일과 소스 파일에 언리얼에서 제공하는 매크로를 각각 선언하는 것이 일반적이다. 

*eg) `ArenaBattle.h`, `ArenaBattle.cpp`*

<br>

**ArenaBattle.h**

```c++
#pragma once

//CoreMinimal.h 대신 EngineMinimal.h 참조
#include "EngineMinimal.h"

//Log 카테고리 선언
DECLARE_LOG_CATEGORY_EXTERN(ArenaBattle, Log, All);

//로그 매크로 선언
#define ABLOG_CALLINFO (FString(__FUNCTION__) + TEXT("(") + FString::FromInt(__LINE__) + TEXT(")"))
#define ABLOG_S(Verbosity) UE_LOG(ArenaBattle, Verbosity, TEXT("%s"), *ABLOG_CALLINFO)
#define ABLOG(Verbosity, Format, ...) UE_LOG(ArenaBattle, Verbosity, TEXT("%s %s"), *ABLOG_CALLINFO, *FString::Printf(Format, ##__VA_ARGS__))
```

* `ABLOG_CALLINFO` : 코드를 호출한 함수(`__FUNCTION__`), 라인(`__LINE__`) 정보 저장 *(로그 매크로 X)*

  * 긴 문자열 형태를 매크로로 저장해 `ABLOG_S`의 로깅 매크로 인자로 쓰임

* `ABLOG_S` : `ABLOG_CALLINFO`정보를 `AretaBattle` 카테고리로 로그 남김

  * 로그를 사용한 함수의 실행 시점 파악 용이

  * `Verbosity` : 로그를 남길 모듈 파일에서 로깅 수준을 인자로 전달

    *(예시는 `Fountain.cpp`참고!)*

* `ABLOG` : `ABLOF_S` 정보에 형식 문자열을 추가한 후 로그 출력

<br>

**Arenabattle.cpp**

```c++
#include "ArenaBattle.h"
#include "Modules/ModuleManager.h"

// 헤더의 로그 선언 정의
DEFINE_LOG_CATEGORY(ArenaBattle);
IMPLEMENT_PRIMARY_GAME_MODULE( FDefaultGameModuleImpl, ArenaBattle, "ArenaBattle" );
```

<br>

**Fountain.h**

```c++
//Fountain의 헤더가 ArenaBattle.h를 참조
// #include "EngineMinimal.h"
#include "ArenaBattle.h"
...
```

<br>

**Fountain.cpp**

```c++
//게임이 시작하자마자 ArenaBattle 카테고리 로그 출력
void AFountain::BeginPlay()
{
	Super::BeginPlay();
    
    //일반적으로 로깅 수준은 Log로 지정 (예제에서는 눈에 띄도록 Warning으로 설정했다!)
    ABLOG_S(Warning);
	ABLOG(Warning, TEXT("Actor Name : %s, ID : %d, Location X : %.3f"), *GetName(), ID, GetActorLocation().X);
}
```



![image](https://user-images.githubusercontent.com/93882395/220549084-ca21b898-d073-467d-9efe-2e3537c8a8e2.png)

>  로그 매크로 출력 결과



## **1.2. 어설션**

게임 진행 도중 문제가 발생하면 게임 실행을 중지하고 오류 구문을 띄우는 점검 코드를 말한다. 문제가 발생했는데도 게임이 계속 진행된다면 더 복잡한 상황을 야기할 수 있으므로 어설션 구문을 반드시 확인해야 한다.

<br>

[**대표적인 어설션 구문**](https://docs.unrealengine.com/4.27/ko/ProgrammingAndScripting/ProgrammingWithCPP/Assertions/)

* `check`  : 조건을 통과하지 못하면 크래시 화면 생성 및 언리얼 에디터 종료
* `ensure` : 언리얼 에디터를 종료하지 않고 로그 윈도우에 오류 출력

어설션 기능이 효과적으로 발휘되도록 에픽게임즈 런처의 엔진 옵션에서 디버깅 기호를 설치하자.



<img src="https://user-images.githubusercontent.com/93882395/220573458-cbfcdc6b-3af3-4c4f-a665-0a1d24feb35d.png" alt="image" style="zoom:50%;" /> 

<br>

## **1.3. 주요 이벤트 함수**

**게임 진행별 액터의 수명 과정**

1. 준비
   * 액터를 구성하는 모든 컴포넌트의 세팅 완료
   * 언리얼에서 해당 액터의 `PostInitializeComponents` 함수 호출 
2. 게임 참여
   * 첫 게임 참여시 `BeginPlay` 함수 호출
   * 매 프레임마다 `Tick` 함수 호출
3. 퇴장
   * 메모리에서 소멸
   * `EndPlay` 함수 호출

<br>

**Fountain.h**

```c++
UCLASS()
class ARENABATTLE_API AFountain : public AActor
{
	GENERATED_BODY()
	
public:	
	// Sets default values for this actor's properties
	AFountain();

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;
	virtual void EndPlay(const EEndPlayReason::Type EndPlayReason) override;
	virtual void PostInitializeComponents() override;
```

* `override` : 함수가 상속받은 함수인지, 새로운 함수인지 컴파일러에게 알림
  * 자식 클래스에서 `override`키워드 사용

**Fountain.cpp**

```c++
void AFountain::EndPlay(const EEndPlayReason::Type EndPlayReason)
{
	Super::EndPlay(EndPlayReason);
	ABLOG_S(Warning);
}

void AFountain::PostInitializeComponents()
{
	Super::PostInitializeComponents();
	ABLOG_S(Warning);
}
```

<br>

언리얼의 액터 클래스 이벤트 함수는 자식이 상속받을 수 있도록 가상함수 `virtual` 키워드로 선언되어 있다. 

액터의 가상 함수 로직에는 언리얼 프레임워크가 동작하기 위해 반드시 실행되어야 하는 로직들이 포함되어 있다. 자식 클래스가 게임에 필요한 로직을 실행하기 전에 먼저 부모 클래스 함수에 있는 로직을 실행시켜야 한다.

<img src="https://user-images.githubusercontent.com/93882395/220594197-cb619780-d966-4544-a369-092bb9af524f.png" alt="image" style="zoom: 60%;" /> 

> 이벤트 함수별 로그

<br>

---

# **2. 움직이는 액터**



## **2.1. Tick 함수**

틱 함수를 사용해 규칙적으로 특정 동작을 반복할 수 있다.

* 렌더링 프레임 단위로 동작
  * `DeltaSeconds` : 이전 렌더링 프레임 ~ 현재 랜더링 프레임 소요 시간 반환
  * 컴퓨터 성능에 따라 렌더링 프레임 값이 불규칙하게 전달됨

<br>

**데이터 은닉**

* 변수를 `private`로 선언하면 컴파일 에러 발생
  * 에디터에서 변수에 접근할 수 없기 때문
* `UPROPERTY` 매크로에 `AllowPrivateAccess` META 키워드 추가
  * 에디터에서 변수 편집 가능, 정보 은닉
  * 프로그래밍시 캡슐화 가능



**Fountain.h** *변수 선언 후 캡슐화 과정*

```c++
#pragma once

#include "ArenaBattle.h"
#include "GameFramework/Actor.h"
#include "Fountain.generated.h"

UCLASS()
class ARENABATTLE_API AFountain : public AActor
{
	...

private:
	//에디터에서 편집할 수 있도록 EditAnywhere 키워드 작성
	UPROPERTY(EditAnywhere, Category = Stat, Meta = (AllowPrivateAccess = true))
	float RotateSpeed;
};
```

<br>

> **cpp 코드부터 에디터 편집까지 프로그램 동작 과정**
>
> 1. 컴파일 진행 전 언리얼 매크로*(`UPROPERTY` 등)*가 언리얼 헤더 툴 프로그램으로 분석됨
> 2. 매크로 분석 후 `generated`이름의 코드가 자동 생성
> 3. `generated`를 포함한 소스 코드 컴파일
> 4. 언리얼 실행 환경이 오브젝트 관리
> 5. 언리얼 에디터에서 편집 인터페이스 제공

<br>

**분수대 움직임 함수 작성**

1. 생성자에서 `RotateSpeed` 속성의 기본값 초기화
2. `Tick` 함수에 `AddActorLocalRotation` 함수 사용
   * `AddActorLocalRotation` : 틱마다 액터 회전
3. `FRotator`를 사용해 회전값 지정
   * `FRotator([Pitch], [Yaw], [Roll]);`
     * `Pitch` : 좌우(Y축)를 기준으로 회전
     * `Yaw` : 상하(Z축)를 기준으로 회전
     * `Roll` : 전후(X축)를 기준으로 회전

<br>

**Fountain.cpp**

```c++
#include "Fountain.h"

AFountain::AFountain()
{
	...
	RotateSpeed = 30.0f;
}

...

void AFountain::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);
    
	// z축을 기준으로(고정축) Fountain 액터 회전
	// 프레임 타임 정보 DeltaTime를 활용해 초당 일정한 속도로 분수대 회전
	AddActorLocalRotation(FRotator(0.0f, RotateSpeed * DeltaTime, 0.0f));
}
```

<br>

> **언리얼 시간 관리자(TimeManager)**
>
> 월드의 시간 관리 기능을 활용해 게임에 필요한 다양한 시간 값들을 다룰 수 있다.
>
> * Tick 함수의 DeltaSeconds: `GetWorld()->GetDeltaSeconds()`
> * 게임 경과 시간: `GetWorld()->GetTimeSeconds()`
>   * 현실  경과 시간: `GetWorld()->GetRealTimeSeconds()`
> * 게임 중지를 제외한 시간: `GetWorld()->GetUnpousedTimeSeconds()`
>   * 현실 시간: `GetWorld()->GetAudioTimeSeconds()`

<br>

## **2.2. 무브먼트 컴포넌트**

움직임 요소를 분리해 액터와 별도로 관리할 수 있도록 움직임 전반을 책임지는 프레임워크로, 액터는 무브먼트 컴포넌트가 제공하는 이동 매커니즘에 따라 움직인다.

* `FloatingPawnMovement` : 중력의 영향을 받지 않는 액터의 움직임
* `RotatingMovement` : 지정 속도로 액터 회전
* `InterpMovement` : 지정 위치로 액터 이동
* `ProjectileMovement` : 중력에 영향을 받아 포물선을 그리는 액터의 움직임 제공

<br>

**Fountain.h**

```c++
...
// 회전 무브먼트 컴포넌트 프레임워크 선언
#include "GameFramework/RotatingMovementComponent.h"
...

UCLASS()
class ARENABATTLE_API AFountain : public AActor
{
	GENERATED_BODY()
	
	...
	
public:
 	// 회전 컴포넌트 선언
	UPROPERTY(VisibleAnywhere)
	URotatingMovementComponent* Movement;
	
    ...

};
```

<br>

**Fountain.cpp**

```c++
#include "Fountain.h"

AFountain::AFountain()
{
	// Tick함수 실행하지 않음
	PrimaryActorTick.bCanEverTick = false;

	...
	// 액터에 무브먼트 컴포넌트 구현
	Movement = CreateDefaultSubobject<URotatingMovementComponent>(TEXT("MOVEMENT"));

	...
	// z축을 기준으로 RotateSpeed만큼 회전
	RotateSpeed = 30.0f;
	Movement->RotationRate = FRotator(0.0f, RotateSpeed, 0.0f);
```

<br>

<img src="https://user-images.githubusercontent.com/93882395/220618462-ab53cc02-6018-4b42-b403-3756a0a0acee.png" alt="image" style="zoom:67%;" /> 

> Movement 컴포넌트가 액터의 별도 영역에 독립적으로 부착된다.
>
> * 씬 컴포넌트: 트랜스폼 정보가 필수적인 컴포넌트
> * 액터 컴포넌트: 부가 기능 제공 컴포넌트



<img src="https://user-images.githubusercontent.com/93882395/220621102-b7b79003-a673-4282-9ae1-86c42c1e3f4f.gif" alt="ezgif com-video-to-gif" style="zoom:67%;" /> 

> 회전하는 분수대 플레이 화면

<br>

---

# **3. 프로젝트 정리**

언리얼 에디터는 액터 제거 메뉴를 제공하지 않아 수동으로 액터를 삭제해주어야 한다.

<br>

**프로젝트 재구성 과정**

1. `.vs`, `Binaries`, `Intermediate`, `ArenaBattle.sln` 삭제

   <img src="https://user-images.githubusercontent.com/93882395/220622148-644a206f-3ddc-4deb-a2b6-cbe368665f37.png" alt="image" style="zoom: 67%;" /> 

   <br>

2. `ArenaBattle.uproject` 파일 우클릭 후 `Generate Visual Studio project files` 선택

   * 언리얼 엔진이 자동으로 비주얼 스튜디오 솔루션 생성

   <img src="https://user-images.githubusercontent.com/93882395/220625227-5dde6e4a-66fa-487c-b218-a86d13cf84eb.png" alt="image" style="zoom: 67%;" /> 

   <br>

3. 생성된 솔루션 파일 컴파일

   * `Binaries->UE4Editor-ArenaBattle.dll` 파일 생성 

> **왜 .sln파일이 자동으로 재생성될까?**
>
> 언리얼 엔진이 언리얼 빌드 툴을 가동시키기 때문!
>
> * 언리얼 빌드 툴: 게임 프로젝트의 폴더 구조와 파일 분석 후 솔루션 파일과 프로젝트 파일 생성
>   * `Source`폴더를 분석해 솔루션 파일 생성
>   * 솔루션 파일이 참조할 프로젝트 파일을 `Intermediate->ProjectFile`폴더에 생성

<br>

**액터 삭제 방법**

1. `Source`폴더에서 삭제할 액터와 관련된 헤더/모듈 파일 삭제
2. `.uproject`파일에서 비주얼 스튜디오 프로젝트 재생성
3. 에디터를 열어 레벨에 변경 사항을 발생시킨 후 저장

<br>

<img src="https://user-images.githubusercontent.com/93882395/220630269-65e975aa-e654-4037-8074-924eb83007fd.png" alt="image" style="zoom:67%;" />

> 액터를 삭제하고 에디터를 실행하면 관련 로드 오류 메시지를 알리는 팝업 윈도우가 뜬다! 
>
> 삭제한 액터를 제외하고 게임이 잘 동작하는지 확인했으면 레벨을 저장하자.
