---
title: "[이득우의 C++ 언리얼] #3 움직이는 액터의 제작"

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

***✨info***<br> 본 튜토리얼 교재는 언리얼 4.19v 기준이나, 4.27v으로 실습 후 게시글을 작성하였습니다.<br>교재 흐름보다는 새로 알게 된 내용과 추가로 공부한 내용 위주로 서술합니다. 배우는 과정이라 부족한 점이 많습니다.😊
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

*   **형식 문자열**: 로그 매크로에서 형식 문자열 기능 지원

    *   문자열 정의시 `TEXT` 매크로 사용 권장
    *   `FString` 타입의 문자열 정보를 가져오려면 `*` 연산자 사용 필요

    *eg. `FString::Printf(TEXT("Actor Name: %s, ID: %d, LocationX %.3f"), *GetName(), ID, GetActorLocation().X);`*

<br>

**로깅 매크로 설정 (로그 카테고리 선언)**

<br>



<br>

## **1.2. 어설션**
