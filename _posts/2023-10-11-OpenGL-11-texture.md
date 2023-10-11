---
title: OpenGL [11] 데이터(4) - 텍스처
categories: [OpenGL]
tags: [OpenGL, Python]
comments: true
---

OpenGL은 데이터를 저장하고 접근하기 위해 2개 형태의 데이터 스토리지를 제공한다.

1. 버퍼
2. **텍스처**

# 텍스처

텍스처(texture)는 쉐이더가 읽기도 하고 쓰기도 하는 구조화된 스토리지 형식을 갖는다.
대부분 이미지 데이터를 저장할 때 사용되지만, 여러 다른 용도로 사용할 수도 있다.

## 텍스처 생성 및 읽기

텍스처를 생성하려면 이름을 생성하고, 그 이름을 텍스처 타깃에 바인딩하고, OpenGL에 저장할 이미지 사이즈를 알려줘야 한다.
새로운 텍스처 객체는 처음 바인된 타깃에 기반하여 타입이 정해진다.

1D, 2D, 3D, RECTANGLE, CUBE, MULTISAMPLE 등 여러 텍스처 타깃이 있다.

쉐이더에서 텍스처를 액세스하려면, 텍스처 타깃에 맞게 샘플러 타입을 생성하면 된다.

1D면 sampler1D, 2D면 sampler2D 등 자세한 정리는 [_[여기]_](https://registry.khronos.org/OpenGL-Refpages/gl4/html/atomicAdd.xhtml)에서 `Sampler types` 목록에서 확인할 수 있다.

## 이미지로부터 텍스처 로딩하기

삼각형 안을 벽돌 이미지로 채워보자.

![gl_texture](/assets/img/post/gl_texture.png)

전체 코드는 [_[깃허브]_](https://github.com/Soo-Bin/helloGL/blob/main/11-texture-1.py) 를 확인하면 된다.

벽돌로 이루어진 정육면체를 만들어보자.

![gl_roate_cube_texture](/assets/img/post/gl_roate_cube_texture.gif)

전체 코드는 [_[깃허브]_](https://github.com/Soo-Bin/helloGL/blob/main/11-texture-2.py) 를 확인하면 된다.

## 여러 텍스처 사용하기

하나의 쉐이더에서 여러 텍스처를 사용하려면 여러 샘플러 유니폼을 만들고 각각 다른 텍스처 유닛을 참조하도록 설정하면 된다.
이 때 여러 텍스처를 한꺼번에 콘텍스트에 바인딩해야 한다.

`GL_MAX_COMBIED_TEXTURE_IMAGE_UNITS` 인자를 사용하여 `glGetIntegerv()`를 호출하면 모든 쉐이더 스테이지에서 사용할 수 있는 최대 동시 텍스처 유닛 개수를 얻을 수 있다.

## 밉맵

밉맵(mipmap)은 렌더링 성능과 비주얼 퀄리티를 모두 향상시킬 수 있는 강력한 텍스처 기법이다. 표준 텍스처 매칭에는 잘 알려진 두 가지 문제가 있다.

1. 반짝거림 현상(또는 계단 현상)
   > 적용된 텍스처의 크기에 비해 매우 작은 크기로 화면에 렌더링되는 객체의 서피스에 나타나는 현상
2. 성능
   > 텍스처를 담기 위해 많은 텍스처 메모리가 사용되는 경우, 화면상의 인접 프래그먼트가 실제 텍스처 공간상에서 멀리 떨어진 텍셀들과 근접하는 경우 발생
   > 텍스처 크기가 크고 드문드문 액세스하는 경우에는 텍스처링 성능이 더 낮아짐

이 두 가지 문제에 대한 해결책은 밉맵을 사용하는 것이다. (이미지 피라미드와 개념이 비슷한듯?)

### Reference

• OpenGL Super Bible 개정 6판
