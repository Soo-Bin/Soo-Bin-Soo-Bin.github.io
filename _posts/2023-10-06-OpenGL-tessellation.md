---
title: OpenGL 테셀레이션
categories: [OpenGL]
tags: [OpenGL, Python]
comments: true
---

테셀레이션(tessellation, 조각화)은 고차 프리미티브(OpenGL에서는 patch로 알려져 있다)를 더 작고, 단순한 여러 개의 렌더링 가능한 프리미티브(예를 들면 삼각형)로 분할하는 작업이다.

이론적으로 테셀레이션 단계는 버텍스 쉐이딩 스테이지 바로 다음에 위치하며, tessellation control shader, tessellation engine, tessellation evaluation shader 세 부분으로 구성된다.

## tessellation control shader

컨트롤 쉐이더 혹은 TCS라고도 부른다. 이 쉐이더는 버텍스 쉐이더로부터 입력을 받아 주로 2가지 일을 수행한다.

1. tessellation engine에 보낼 tessellation level을 결정하는 일
2. tessellation이 수행된 다음에 실행되는 tessellation evaluation shader에 보낼 데이터를 생성하는 일

쉐이더 작성은 아래와 같이 할 수 있다.

```python
tess_shader_source = """
#version 430 core
layout(vertices=3) out;

void main()
{
    if(gl_InvocationID == 0)
    {
        gl_TessLevelInner[0] = 5.0;
        gl_TessLevelOuter[0] = 5.0;
        gl_TessLevelOuter[1] = 5.0;
        gl_TessLevelOuter[2] = 5.0;
    }
    gl_out[gl_InvocationID].gl_Position = gl_in[gl_InvocationID].gl_Position;
}
"""

tess_shader = shaders.compileShader(tess_shader_source, GL_TESS_CONTROL_SHADER)
shader = shaders.compileProgram(..., tess_shader, ...)
```

`layout(vertices=N) out;`은 패치당 N개의 제어점을 사용한다는 것이다. 기본적으로 패치당 제어점 개수는 3이다.
TCS를 작성하지 않는다면 코드에서 `glPatchParameteri(GL_PATCH_VERTICES, N)`로 대체 가능하다.

또한, `gl_TessLevelInner`로 내부 테셀레이션 레벨 설정을, `gl_TessLevelOuter`로 외부 테셀레이션 레벨 설정을 할 수 있다.
이 부분도 TCS를 작성하지 않는다면 코드에서 `glPatchParameterfv(GL_PATCH_DEFAULT_INNER_LEVEL, ...)`과 `glPatchParameterfv(GL_PATCH_DEFAULT_OUTER_LEVEL, ...)`로 대체 가능하다.

## tessellation engine

테셀레이션 엔진은 OpenGL 파이프라인의 고정 함수로서, 패치로 표현되는 고차 서피스를 점, 선, 삼각형 같은 작은 프리미티브로 분할하는 역할을 수행한다.
프리미티브들이 생성되면, 해당 버텍스들은 tessellation evaluation shader로 전달된다.

## tessellation evaluation shader

TES라고도 불린다. 테셀레이션 레벨이 높으면 TES가 매우 많이 수행되기 때문에, 복잡한 TES와 높은 테셀레이션 레벨을 사용한다면 성능에 주의해야 한다.

쉐이더 작성은 아래와 같이 할 수 있다.

```python
tess_evaluation_shader_source = """
#version 430 core
layout(triangles) in;

void main()
{
    gl_Position = (gl_TessCoord.x * gl_in[0].gl_Position) +
                  (gl_TessCoord.y * gl_in[1].gl_Position) +
                  (gl_TessCoord.z * gl_in[2].gl_Position);
}
"""

tess_evaluation_shader = shaders.compileShader(tess_evaluation_shader_source, GL_TESS_EVALUATION_SHADER)
shader = shaders.compileProgram(..., tess_evaluation_shader, ...)
```

쉐이더 시작 부분에 보면 레이아웃 지시어로 테셀레이션 모드를 설정하는 것을 확인할 수 있다. 여기서는 삼각형을 사용하는 모드를 선택했다.
나머지 부분에서는 버텍스 쉐이더처럼 `gl_Position`에 값을 할당한다. `gl_TessCoord` 또한 내장 변수로 테셀레이터가 생성한 버텍스의 **무게중심 좌표**다.

테셀레이션의 결과를 보려면 OpenGL이 결과 삼각형의 가장자리만 그리도록 할 필요가 있다.
`glPolygonMode(GL_FRONT_AND_BACK, GL_LINE)`을 호출하면 된다.

---

아래 예시는 tessellation outer level을 4,1,6으로 설정하고 inner level을 5로 설정했을 때 결과다.

![gl_tessellation](/assets/img/post/gl_tessellation.png)

### Reference

• OpenGL Super Bible 개정 6판

• [https://www.khronos.org/opengl/wiki/Tessellation](https://www.khronos.org/opengl/wiki/Tessellation)
