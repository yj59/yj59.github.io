---
title: "이득우의 언리얼 C++ 게임 개발의 정석 #1"

header:
  overlay_image: https://user-images.githubusercontent.com/93882395/212655591-8ae50295-2acd-4d6b-9a8d-b6924f334ab3.jpg
  overlay_filter: 0.7 # 투명도

tags:
  - Unreal
  - Tutorial
  - Game

toc: true
toc_sticky: true
toc_label: "Unreal Tutorial"
last_modified_at: 2023-01-02

use_math: true
---

> ***🤍ref.***
>
> 1. **이득우의 언리얼 C++ 게임 개발의 정석**  *이득우, 에이콘출판사, 2018*
>2. [Unreal Engine 4.27 Documentation](https://docs.unrealengine.com/4.27/ko/)
>     *   [유니티 개발자를 위한 언리얼 엔진4](https://docs.unrealengine.com/4.27/ko/Basics/UnrealEngineForUnityDevs/)

***✨info***<br> 본 튜토리얼 교재는 언리얼 4.19v 기준이나, 4.27v으로 실습 후 게시글을 작성하였습니다.<br>교재의 흐름보다는 새로 알게 된 내용과 추가로 공부한 내용 위주로 서술합니다. 배우는 과정이라 부족한 점이 많습니다.😊
{: .notice--info}

<br>

# 1. 언리얼 구성요소

## 1.1. 프로젝트 파일

<img src="https://user-images.githubusercontent.com/93882395/210505817-14201f2b-74e5-4c50-9499-194752b26df5.png" alt="image" style="zoom: 67%;"/> 

*   `Config`: 프로젝트 설정 값 보관
*   `Content`: 프로젝트에서 사용하는 애셋 관리
*   `Intermediate`: 프로젝트 관리에 필요한 임시 파일 저장 => 제거해도 자동 생성
*   `Saved`: 작업 중 생성된 결과물 저장 *(세이브 파일, 스크린샷 등)* => 프로젝트에 영향 x

*   C++ 프로젝트 확장시 생성되는 폴더
    *   `Binaries`: 컴파일된 결과물 저장 => 빌드마다 재생성
    *   `Source`: 소스 코드 저장 공간
    *   `*.sln`: C++ 프로젝트를 관리하기 위한 VS solution 파일
        *   솔루션이 관리하는 프로젝트 파일 -> `Intermediate` -> `ProjectFiles`
        *   `uproject` -> `Generate Visual Studio project file`에서 재생성 가능

<br>**💙 UnrealTutorial.uproject**



<script src="https://gist.github.com/yj59/3ab5724895ffd7694e1e6c22aa2df6ed.js"></script>

프로젝트를 에디터로 불러들이기 위한 정보를 저장한다.
*   JSON 형식으로 기록
    *   `{}`: 객체
    *   `Key:Value`: 객체가 가지는 속성
    *   `""`: 문자열 데이터
    *   `[]`: 배열
*   `EngineAssociation`: 구동할 에디터 버전
*   `Modules`: C++ 모듈*(컴파일 결과물)* 함께 로딩
    *   `Binaries` -> `UE4Editor-[프로젝트명].dll` 검색 후 탑재

<br>**💙 C++ 개발 환경 설정**: 모듈 빌드 구성



<img src="https://user-images.githubusercontent.com/93882395/210510210-13343762-d1f1-40ca-895e-36abbebacbf3.png" alt="image" style="zoom: 80%;"/> 

*   `DebugGame`: 디버깅을 위해 최적화가 안 된 결과물 생성, exe 파일 생성
    *   `DebugGame Editor`: `DebugGame`과 동일한 수준의 에디터용 DLL 파일 생성
*   `Development`: 중간 수준의 최적화, 디버징 가능한 결과물 생성, exe 파일 생성
    *   `Development Editor`: `Development`와 동일한 수준의 에디터용 DLL 파일 생성
*   `Shipping`: 최종 배포를 위해 최적화된 코드 생성, exe 파일 생성

<br>

## 1.2. 에디터 용어

**💙 액터 Actor**

<img src="https://user-images.githubusercontent.com/93882395/212664989-36880ac8-5a24-4230-a0a8-c93c02b9bd23.png" alt="image" style="zoom:67%;" /> 

*   `라벨`: 액터의 명칭. 여러 액터가 같은 이름을 가질 수 있음
*   `유형`: 액터가 담당하는 역할으로 액터의 클래스명

<br>

**💙 레벨 Level**

플레이어가 이동하는 스테이지 영역으로, 한 월드에 배치된 액터들을 통칭한다. 일반적으로 액터를 조정하고 속성을 설정하는 등 한 레벨에서 게임을 구현하는 과정을 `레벨을 설계`한다고 부른다.

<br>

**💙 [컴포넌트 Component](https://docs.unrealengine.com/4.27/ko/Basics/Components/)**

액터는 여러 개의 컴포넌트를 가질 수 있으며, 하나의 대표 컴포넌트를 지정해야 한다(`Loot Component`). 

<br>

**💙 [모빌리티](https://docs.unrealengine.com/4.27/ko/Basics/Actors/Mobility/)**

<img src="https://user-images.githubusercontent.com/93882395/212656604-432e92eb-da0f-45e5-9646-87961c33c8e3.png" alt="image" style="zoom:67%;" /> 

액터가 게임플레이 도중 어떤 방식으로 이동/변화할지 제어한다. 주로 스태틱 메시 액터나 라이트 액터에 적용된다.

*   `Static`: 액터 고정
*   `Stationary`: 위치는 고정이나 상태 변화 가능
*   `Movable`: 추가/제거/이동

라이트 액터의 모빌리티의 경우, 유니티의 `Lightmapping Settings`을 월드 단위로 관리하는 방식이다. 각각 `Baked`, `Mixed`, `Realtime`으로 매치해볼 수 있겠다. 일반적으로 `Static`라이트를 통해 라이팅을 미리 빌드하는 것이 좋다. 무버블 라이트는 플레이 도중 실시간으로 라이트를 계산해 최적화가 비효율적이며, 퀄리티도 스태틱에 비해 좋은 편이 아니다.

*(교재 실습에서는 튜토리얼의 편의를 위해 라이트 액터 모빌리티를 무버블로 설정했다.)*

<br>**💙 [프로젝트 세팅](https://docs.unrealengine.com/4.27/ko/Basics/Projects/ProjectSettings/) - 맵&모드**

<img src="https://user-images.githubusercontent.com/93882395/212662103-14a06551-d941-4758-9425-1410550ac973.png" alt="image" style="zoom:67%;" /> 

*   **컨트롤** - 특정 섹션의 설정값 제어
    *   `Export...`: 설정값을 외부 환경설정(`.ini`) 파일로 익스포트
    *   `Import...`: 환경설정(`.ini`) 파일에 저장된 값 로드 -> 현재 값 대체
*   **맵 & 모드**: 기본적으로 로드할 맵&모드와 방식. 로컬 멀티플레이어 화면 레이아웃 설정 등 전반적인 맵 설정 제어

<br>

**🪄*Tip)* [이주하기 (Migration)](https://docs.unrealengine.com/4.27/ko/Basics/AssetsAndPackages/Migrate/)**<br>`Migrate Tool`을 활용해 애셋들을 한 프로젝트에서 다른 프로젝트로 복사할 수 있다.<br>*eg)* 머티리얼을 하나 이주시키면 해당 머티리얼을 정의하는 텍스처 애셋 자동 복사<br>`Input`, `Collision`, `Map` 등 프로젝트 설정과 언리얼 하위 버전으로는 이주가 불가능하니 주의하자.
{: .notice--primary}

<br>
