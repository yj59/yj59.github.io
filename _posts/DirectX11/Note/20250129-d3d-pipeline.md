---
title: "Direct3D 그래픽 파이프라인"

date: 2025-01-29 00:00:00 +0900
author: yejin

categories: [DirectX 11]
tags: [RasterTek, tuto]

math: true
---



> **참고 자료**
>
> [Microsoft Learn - 그래픽 파이프라인](https://learn.microsoft.com/ko-kr/windows/uwp/graphics-concepts/graphics-pipeline)



![Image](https://github.com/user-attachments/assets/d9f1115a-8a8f-40cf-920f-f8a9b5443216)
_그래픽 파이프라인_



# Input-Assembler



그래픽 데이터를 파이프라인에 전달하는 첫 번째 단계

* Vertex data와 Index data를 입력
  * CPU가 정점/인덱스 버퍼로 저장된 데이터를 GPU에 전송
* 원시 그래픽 형상(Primitive) 생성
  * 입력받은 정점/인덱스 데이터를 사용해 이후 단계에서 사용할 기본 그래픽 형상(선, 삼각형 등) 생성

<br>

*왜 사용할까?*

1. CPU가 렌더링할 도형 정보를 GPU로 전송
2. 버퍼에서 데이터를 읽어들여 효율성 높임
   * 버퍼에 저장되어 있으므로 빠른 접근 가능
   * GPU 메모리에 저장되어 데이터 일관성 유지
   * 재사용성 높임
3. [시스템 생성 값](https://learn.microsoft.com/ko-kr/windows/uwp/graphics-concepts/using-system-generated-values)(VertexID, PrimitiveID, InstantID)을 셰이더 단계에 연결
   * 셰이더가 ID을 식별해 처리가 빨라짐

4. 정점 데이터를 다양한 도형으로 조합, 인접 데이터를 활용해 더 정교한 도형 구현 가능



# Vertex Shader

