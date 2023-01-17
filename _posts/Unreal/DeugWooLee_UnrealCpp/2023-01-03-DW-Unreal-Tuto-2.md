---
title: "언리얼 튜토리얼 #2 액터 설계, 클래스 살펴보기"

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

***✨info***<br> 본 튜토리얼 교재는 언리얼 4.19v 기준이나, 4.27v으로 실습 후 게시글을 작성하였습니다.<br>교재보다는 새로 알게 된 내용과 추가로 공부한 내용 위주로 서술합니다. 배우는 과정이라 부족한 점이 많습니다.😊
{: .notice--info}

<br>

# 1. 액터 설계

액터 구성하려면 어떤 컴포넌트가 필요한지, 선언부에는 뭐가 들어가야 하고 구현부에는 어떤게 들어가야 하는지 등

