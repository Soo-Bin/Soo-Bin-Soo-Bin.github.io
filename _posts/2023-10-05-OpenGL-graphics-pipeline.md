---
title: OpenGL 그래픽스 파이프라인
categories: [OpenGL]
tags: [OpenGL]
comments: true
---

그래픽스 파이프라인(Graphics pipeline) 또는 랜더링 파이프라인(Rendering pipeline)이라고 불린다. 이는 OpenGL이 객체를 **렌더링**할 때 수행하는 일련의 단계다. 여기서 렌더링은, 쉽게 말하면 3차원 개체를 가져와서 2차원 화면에 픽셀로 표현하는 것이다.

OpenGL 렌더링의 기본 단위는 **프리미티브(primitive)**다. 많은 타입의 프리미티브를 지원하지만 기본 렌더링 가능 타입은 **점, 선, 삼각형**이다. 애플리케이션은 보통 복잡한 서피스를 많은 수의 삼각형으로 분할하고, OpenGL로 보내서 **래스터라이저(rasterizer)**라고 불리는 하드웨어 가속기를 사용하여 렌더링한다. 래스터라이저는 3차원으로 표현된 삼각형을 화면에 그려질 일련의 픽셀로 변환하는 전용 하드웨어다.

랜더링 파이프라인은 일단 시작되면 다음과 같은 순서로 작동한다.

![[그림 1]](/assets/img/post/gl_graphics_pipeline.png)

- 둥근 박스: 고정 함수 스테이지
- 일반 박스: 프로그래밍 가능 스테이지

파이프라인을 보면 두 개의 주요 파트로 분할된다.

1. 위에서 미리 설명했지만, **프론트엔드**라고 흔히 부르는 것으로 버텍스와 프리미티브를 처리하여 점, 선, 삼각형으로 구성하고 이들을 래스터라이저에 보내는 역할을 한다. 이를 **프리미티브 어셈블리(primitive Assembly)**라고도 한다.
2. 래스터라이즈 이후에는 지오메트리가 벡터 형태에서 대량의 각각의 픽셀로 변환된다. 이들은 **백엔드**로 전달되고, 여기에는 깊이 및 스텐실 테스트, 프래그먼트 쉐이딩, 블렌딩, 출력 이미지 갱신 등의 작업이 포함된다.

### Reference

• OpenGL Super Bible 개정 6판

• [https://www.khronos.org/opengl/wiki/Rendering_Pipeline_Overview](https://www.khronos.org/opengl/wiki/Rendering_Pipeline_Overview)
