---
date: 2025-04-30(WED)
tags:
  - "#CG"
math: true
---
- Culling: Z값을 이용해, 뒤에 있는 건 가려지고, 앞에 있는건 보이게 하는 기법
- Clipping: 보여지는 영역 밖은 안그리는 기법
- Stencil: 특정 부분은 안그려지게 하는 기법

# Culling by depth

멀리 있는 물체는, 가까운 물체에 의해 가려져서 안보일 것이다.
즉 보이는 가장 가까운 것들만 그리면 된다. 그것에 관한 방법들에 대해 알아보자.

## (1) Depth-Sorting(Object Space Method-CPU에서 처리)

1. Z값 기준으로 정렬.
2. 그러고선 '먼' 물체부터 그려서, 마지막에 '가까운' 물체를 그린다.
(실제 유화를 그리는 방식과 동일)

##### 문제점

![](attachment/60839c2a17c1044f3ad249bd35698046.png)
▲ 대체 z값을 어떤 기준으로 정렬??

위 그림의 경우
1. 물체의 가장 앞 z값 기준 정렬? $\rightarrow$ 노랑이가 하늘색에게 안덮힘
2. 물체의 가장 뒤 z값 기준 정렬? $\rightarrow$ 하늘이가 다른 애들을 못덮음 

그래서 문제점들:

- 물체들의 z값이 겹칠 때에는?? 그릴 수가 없다.
- 픽셀을 너무 많이 그린다.
- 물체 n개, 총 픽셀 수 m의 경우: nlogn(정렬) + m 인데 불완전하기까지 ㄷㄷ

## (2) Depth-Buffering(Image Space Method-GPU에서 처리)

물체 단위로 z(depth) 고려해서 그리지 말고, ==Pixel 단위로== 하자! 는 전략.
그런데 픽셀마다 정렬하는건 너무 비효율적이다.
$\Rightarrow$ =='정렬을 안하고' 그냥 '그리디하게' '가장 앞의 점만 저장'하자!!!==

그러기 위해선 또다른 버퍼가 필요하다.

#### Depth-buffer

픽셀 별로 제일 가까운 물체 depth를 기록해두는 buffer.
float type으로 기록된다.

![](attachment/544d2812fbeb6ca292c55a7caa0d3ba5.png)
▲ Pipeline 상에서는 Rasterizer 이후, Fragment processor 이후에서 처리한다.
(OpenGL은 Fragment processor 다음에 처리)

##### 여전히 문제점이..

- 소수점 정확도 문제(소수점 비교가 부정확해서 거의 겹칠 경우 지글지글..)
- 투명한(Transparent) 것들은????

### solution: A-Buffer

depth-buffer로는 투명한 것들과 함께 그릴때 문제가 발생한다.
(e.g. 여러 유리가 겹치는 상황, z값을 하나만 기록하면 망함)

따라서 다음 전략을 사용한다. 이는 GPU, 메모리 등이 발전해서 가능해진 해결.

<u>"불투명한 것들 중에서 제일 가까운 것을 Tail로, head까지 그 사이 투명한 것들 저장"</u>

![](attachment/8ccb424c0224335d291bd1b056b55a67.png)

##### 그래도 문제점?

- 정확하게 그리지만, 계산량이 많다는 단점...

$\Rightarrow$ 그래서 결국, ==Object Space에서 '정확도를 해치치 않는' 선에서 계산량을 줄여주도록 최대한 처리하고, 나머지를 Image-Space에서 처리!!!==


## (3) Hierarchical-Z

z-pyramid?

---

# Clipping

화면에 보여지는 부분 이외의 부분들은 그릴 필요가 없다.
이를 어떻게 구별하는지, 방법들을 보자.

#### Coordinate Systems and Clipping

Clipping은 사실 어느 좌표계에서든지 가능하다.

![](attachment/4344af7fb048f2f68efe564f2fd27b8a.png)
▲ Clipping Window 밖인지 체크 / N.S. 밖인지 체크 / Viewport 밖인지 체크

그저 Screen에 그리기 전에 Clipping 하기만 하면 된다.

#### two kinds of clipping

(0) Point Clipping

그냥 어떤 물체의 점이 영역 밖의 점인지 판별하는 방법.

Problem: 그러나 이는 "모든 점"을 다 봐야하기에, 상당히 비효율적이다.

##### (1) Line Clipping

영역을 이루는 4개의 직선($y_{min}, y_{max}, x_{min}, x_{max}$)과 각 물체의 직선을 비교.
"교점"을 파악하면 된다.

1. 교점 0개의 경우

- 직선이 Viewport 내부에서만 존재.
- 직선이 아예 외부에만 존재.

2. 교점 1개의 경우

- 직선이 꼭지점에서 접함.
- 직선 시작 혹은 끝점 둘 중 하나가 내부에 존재.

3. 교점 2개의 경우: "두 교점 사이만" 그리면 됌

==Problem: 그러나 Polygon 물체의 경우, 내부 색칠하기가 불가능==하다.

Q. 엥 왜 불가능함?
A. 하나의 다각형으로 인지하지를 않음. 그냥 자르기만함. 그래서 보간할 수가 없음.
(e.g. 삼각형이 클리핑 되어서 사각형이 되었다고 해보자. 근데 잘린 두 선분, 안잘린 한 선분은 인지하지만, 클리핑된 선분은 인지를 못함. 삼각형 내부 선분-원래 없었음)

![](attachment/b6096083a054da162f6f132ac8ccd155.png)


##### (2) Polygon Clipping

영역을 이루는 4개의 직선($y_{min}, y_{max}, x_{min}, x_{max}$)을 순서대로 "한번씩 기준잡고 교점 자르기."
그러면서, 잘린 경계면의 점들을 추적해서, 새로운 다각형으로 구성시켜줌. 그래서 보간 가능!


![](attachment/7b6fcd121c05393a73bbb50c664ac366.png)
▲ $x_{min}$직선으로 영역 나누고, $x_{max}$직선으로 영역 나누고, $y_{min}$직선으로 영역 나누고, $y_{max}$로 마무리

##### 구체적으론

![](attachment/750ebcb800b3fa06a910255e7f4ab302.png)
▲ TOP 직선($y_{max}$)을 기준으로 교점을 구해내고 커트.

![](attachment/a0de57c33ed5ec146b138f6f6d0c74ed.png)
▲ 순서대로 한번씩 기준되어 교점들 구해지며 커트.

## Clipping pipeline

![](attachment/59a1e66c440dbda903fcf9e0a74a99c8.png)

## Clipping in OpenGL and Front/Back Face

##### glViewport(x, y, w, h)

Clipping Space(=Normalized Coordinate)에서 ViewPort로 매핑하면서... 음..?

### 카메라 등진 면(Back-face)은 안그리도록 하는 방법

카메라의 View Vector과, 어떤 면의 Surface Normal이 이루는 각이 '예각'이면? 
$\rightarrow$ 카메라를 등지고 있는 면이기에, 안그려도 된다!!
(각은 surface normal • viewing vector 내적으로 구해낼 수 있다)

#### Surface Normal?

![](attachment/b1dec08227a99f0a72aa13b3e18b16a8.png)
▲ CCW순으로 그려서, 아래벡터$\times$위벡터 = 법선벡터 위방향 (오른나사로 보면 될듯)

면의 법선 벡턴데, 이게 법선 벡터는 외적 곱하는 순서에 따라 방향이 다르다.
<u>OpenGL에서는 CCW 방향으로 변들을 그려서, 오른나사 엄지방향이라고 보면 된다.</u>

##### glFrontFace(mode)

- mode: GL_CCW(default), GL_CW

##### glCullFace(mode): 안 그리는(cull out) 면 지정

- mode: GL_BACK(default), GL_FRONT, GL_FRONT_AND_BACK

###### cull out On & Off

- glEnable(GL_CULL_FACE)
- glDisable(GL_CULL_FACE)

---

# Stencil

##### What is Stencil?

stencil 방법은 특정 mask를 종이에 갖다 대고 그리면, mask 영역만 보이게 하는 그림 기법이다.

![](attachment/4cc96bbeb0065243473acc467335fc38.png)
▲ Stencil buffer + image $\rightarrow$ test $\rightarrow$ result

##### Stencil Buffer

그 전에 두 개의 버퍼(Vertex buffer, Depth buffer)에 이어, stencil용 새로운 buffer가 필요하다.
그게 바로 Stencil buffer.

- 8 bit per pixel (0~255 value)
- stencil frame으로 add/substract.... 여러 작업 가능

### OpenGL Stencil

##### glStencilFunc(GLenum func, Glint ref, GLuint mask)

- func: test 어떻게 할지 정하는 인자(GL_LEQUAL, GL_ALWAYS)
- ref: test에 쓰이는 비굣값, stencil buffer의 값과 비교된다.
- mask: 비교 전에, ref와 stencil buffer 값에 AND 연산될 mask값

##### glStencilMask(GLuint mask)

stencil을 disable하거나 enable하는 함수(stencil 값과 AND되니까 0x00이나 0xFF로 조작)

- mask에 0x00: Disable
- mask에 0xFF: Enable

##### glStencilMaskSeparate(GLenum face, GLuint mask)

mask는 위에 것과 동일하다. 차이점은 특정 면에 대해서만 적용할 수 있다는 점.

- Face: GL_FRONT, GL_BACK, GL_FRONT_AND_BACK

##### glStencilOp(GLenum sfail, GLenum dpfail, GLenum dppass)

각 인자들에는 해당 결과 후 어떤 작업이 수행될지 함수를 등록.

GL_KEEP(그대로), GL_ZERO(0으로), GL_REPLACE, GL_INCR, GL_INCR_WRAP, GL_DECR, GL_DECR_WRAP, GL_INVERT

- sfail: stencil test 미통과시
- dpfail: stencil test는 통과, depth test 미통과시
- dppass: stencil test, depth test 전부 통과시

##### glStencilOpSeparate(GLenum face, ..)

다 동일하고 면마다 적용할 수 있다는 차이점.



