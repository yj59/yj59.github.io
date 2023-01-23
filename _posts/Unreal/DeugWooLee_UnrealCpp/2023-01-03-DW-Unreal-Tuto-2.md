---
title: "언리얼 튜토리얼 #2 액터 설계"

header:
  overlay_image: https://user-images.githubusercontent.com/93882395/212655591-8ae50295-2acd-4d6b-9a8d-b6924f334ab3.jpg
  overlay_filter: 0.7 # 투명도

categories:
  - Unreal Tutorials

tags:
  - Unreal
  - Tutorial
  - Game 

toc: true
toc_sticky: true
toc_label: "Unreal Tutorial"
toc_icon: "fa-regular fa-gamepad"
last_modified_at: 2023-01-03

use_math: true
---

> ***🤍ref.***
>
> 1. **이득우의 언리얼 C++ 게임 개발의 정석**,  *이득우, 에이콘출판사, 2018*
> 2. [Unreal Engine 4.27 Documentation](https://docs.unrealengine.com/4.27/ko/)
>     *   [유니티 개발자를 위한 언리얼 엔진4](https://docs.unrealengine.com/4.27/ko/Basics/UnrealEngineForUnityDevs/)

***✨info***<br> 본 튜토리얼 교재는 언리얼 4.19v 기준이나, 4.27v으로 실습 후 게시글을 작성하였습니다.<br>교재 흐름보다는 새로 알게 된 내용과 추가로 공부한 내용 위주로 서술합니다. 배우는 과정이라 부족한 점이 많습니다.😊
{: .notice--info}

<br>

---

# **1. 액터 설계**

분수대 액터에 스태틱메시 컴포넌트를 추가해보자.

*   분수대 액터: **구조대**와 **물**으로 구성
    *   **구조대**: 스태틱메시 컴포넌트*(비주얼, 충돌)* => 루트 컴포넌트
    *   **물**: 스태틱메시 컴포넌트*(비주얼)*

<br>

한 엑터가 두 개의 스태틱메시 컴포넌트를 가지려면 액터의 멤버 변수에 두 개의 컴포넌트 클래스 포인터를 선언해야 한다.

`CoreMinimal.h` -> `EngineMinimal.h` 으로 변경해 분수대 액터 클래스가  `UStaticMeshComponent` 클래스를 참조할 수 있도록 하자.

*   `CoreMinimal.h`: 언리얼 오브젝트가 동작하는 최소 기능 선언*(기본적인 연산, 타입 지정 등)*
*   `EngineMinimal.h`:  `CoreMinimal.h`를 포함한 엔진 클래스 선언의 집합

불필요한 헤더 파일을 참조하는데 걸리는 컴파일 시간과 인텔리센스의 부하를 최소화하기 위해 클래스 템플릿의 기본 헤더가 `CoreMinimal.h`으로 변경되었다. 콘텐츠를 제작하기 위해선 다양한 엔진 기능이 필요하니 `EngineMinimal.h`로 변경해주자.

<br>

## **1.1. Fountain.h**

```c++
//Fountain.h
#pragma once
#include "EngineMinimal.h"
#include "GameFramework/Actor.h"
#include "Fountain.generated.h"

UCLASS()
class UNREALTUTORIAL_API AFountain : public AActor
{
	GENERATED_BODY()

...
    
public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;

    UPROPERTY(VisibleAnywhere)
    UStaticMeshComponent* Body;

    UPROPERTY(VisibleAnywhere)
    UStaticMeshComponent* Water;

    UPROPERTY(VisibleAnywhere)
    UPointLightComponent* Light;

    UPROPERTY(VisibleAnywhere)
    UParticleSystemComponent* Splash;

    UPROPERTY(EditAnywhere, Category=ID)
    int32 ID;
};
```

*   `UStaticMeshComponent`포인터 변수로 분수대 액터 선언 (`Body`, `Water`)
    *   `UPointLightComponent`: 조명 컴포넌트 클래스
    *   `UParticleSystemComponent`: 이펙트 컴포넌트 클래스
*   **`UPROPERTY()`**: 언리얼 실행 환경이 객체를 자동으로 관리해주는 매크로
    *    객체가 더 사용되지 않으면 할당된 메모리 자동 소멸
    *    포인트 선언 코드 윗줄에 매크로 추가
    *    **언리얼 오브젝트**에만 매크로 사용 가능
         *    언리얼 실행 환경에 의해 관리되는 C++ 객체
         *    콘텐츠를 구성하는 객체 전반을 일컬음
    *    데이터 자료형에 매크로를 선언해줄 경우 기본값이 자동으로 지정됨 *(eg. 정수형 -> 초기값 0 보장)*

언리얼 에디터에서 직접 컴포넌트의 속성을 편집하려면 `UPROPERTY` 매크로 안에 [**프로퍼티 지정자**](https://docs.unrealengine.com/4.27/ko/ProgrammingAndScripting/GameplayArchitecture/Properties/)를 추가해야 한다.

**대표적인 프로퍼티 지정자**

*   `VisibleAnywhere`: 속성값 확인 가능
    *   4.27.2 버전에선 키워드를 추가하지 않으면 디테일 윈도우에서 선언한 컴포넌트 자체를 볼 수 없음
*   `EditAnywhere`: 속성의 데이터 변경시 사용

>   `UPROPERTY` 매크로의 키워드가 `VisibleAnywhere`일 경우, 디테일 윈도우에서 값 유형은 변경할 수 없다. 왜 객체의 속성은 편집할 수 있을까?
>
>   => `VisibleAnywhere` 키워드는 객체를 볼 수는 있지만 다른 객체로 변경은 불가능하다. 
>   <br>예를 들어, `UStaticMeshComponent`로 선언한 컴포넌트를 라이트 컴포넌트로 바꿀 수 없다. 단, 객체는 레벨에 객체를 구성하기 위한 세부 데이터들이 있으므로 에디터에서 해당 값들을 보여주고 편집할 수 있게 한다.
>
>   <br>
>
>   *+) `VisibleAnywhere` 키워드를 작성하지 않고 헤더를 컴파일했을 때 디테일 뷰에서 컴포넌트 자체를 확인할 수 없었다. 에디터에서 컴포넌트를 보려면 `VisibleAnywhere`를 꼭 적어주자.*

<br>

### **1.1.1. 언리얼 오브젝트**

> [UE 4.27 Doc](https://docs.unrealengine.com/4.27/ko/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Objects/)

**언리얼 오브젝트 클래스 조건**

1.   **클래스 선언 매크로**: 해당 클래스가 언리얼 오브젝트임을 선언
     *   언리얼 선언 윗줄:`UCLASS`
     *   클래스 내부 `GENERATED_BODY`
2.   **클래스 이름 접두사**
     *   액터 클래스: `A` *(분수대 액터 -> **A**Foundation)*
     *   액터가 아닌 클래스: `U` *(스태틱메시 -> **U**StaticMeshComponent)*
     *   일반 c++클래스/구조체*(언리얼과 관련x)*: `F`
3.   **`generated.h`헤더 파일**: 언리얼 헤더 툴이 생성한 부가 정보 파일을 가져오도록 함
     *   `(클래스명).generated.h` 선언
4.   **외부 모듈 공개 여부**: 다른 모듈에서 해당 객체에 접근할 수 있도록 설정
     *   클래스 선언 앞: `(모듈명)_API`

<br>

### **1.1.2. 언리얼 속성 값**

언리얼 오브젝트의 속성 값은 객체를 관리하는 객체 유형과 값을 관리하는 값 유형으로 나뉜다. `UStaticMeshComponent* Body;`와 같은 오브젝트 클래스 포인터는 객체 유형에 해당한다.

<br>

[**대표적인 언리얼 엔진 자료형**](https://docs.unrealengine.com/4.27/ko/ProgrammingAndScripting/GameplayArchitecture/Properties/)

*   **byte**: `uint8`
*   **정수**: `int32`
*   **실수**: `float`
*   **문자열**: `FString`, `FName`
*   **구조체**: `FVector`, `FRotator`, `FTransform`

<br>

## **1.2. Fountain.cpp**

```c++
//Fountain.cpp
#include "Fountain.h"

AFountain::AFountain()
{
	PrimaryActorTick.bCanEverTick = true;

	Body = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("BODY"));
	Water = CreateAbstractDefaultSubobject<UStaticMeshComponent>(TEXT("WATER"));

	RootComponent = Body;
	Water->SetupAttachment(Body);
    
	Water->SetRelativeLocation(FVector(0.0f, 0.0f, 135.0f));
}
...
```

액터의 구축은 클래스의 생성자 코드에서 작성한다. 생성자 코드에서 컴포넌트를 생성하기 위해 `CreateDefaultSubobject` API 함수를 사용한다.

`Body` 컴포넌트가 루트 컴포넌트이므로, `Water`가 `Body`의 자식이 되도록 설정한다.

*   `CreateDefaultSubobject`: 
    *   `TEXT("해시값")`: 컴포넌트를 구별하기 위한 해시 값. `TEXT`매크로를 통해 문자열 체계 통일
*   `SetRelativeLocation`: 컴포넌트의 기본 위치 변경

<br>

---

>   
