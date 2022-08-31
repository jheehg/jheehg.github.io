---
layout: post
title: Lottie Animation 리액트 프로젝트에 적용하기
description: 리액트 Lottie Animation으로 toggle 버튼을 구현해봅니다.
summary: 리액트 Lottie Animation으로 toggle 버튼을 구현해봅니다.
comments: true
categories: [FE]
tags: [react, lottie, animation, typescript]
---

<br>

리액트에서 사용 가능한 `lottie` 모듈 중 `lottie-react` 를 사용하였다. <br>
어떤 목적으로 사용하는지에 따라 다르겠지만 토글버튼을 사용하기 위해서 프레임 조정 기능을 지원하는 모듈이 필요했다.<br><br>

### react-lottie
- playSegments를 내부적으로만 사용하기 때문에 `forceFlag` 값을 따로 지정 못하는 문제점이 있었다.<br><br>
### react-lottie-segments
- playSegments를 간단하게 사용할 수 있었지만 지속적으로 관리되는 모듈을 사용하고자 하였다.<br><br>
### 고려했던 부분들
1. 클릭 이벤트 발생 시 프레임을 바꿔주려면 playSegments 메서드를 직접 호출해야 함
2. 타입스크립트 지원이 되는 모듈

<br><br>

### playSegments(segments, forceFlag)
```js
{
segments: [start, stop],
forceFlag: true,
}
```
<br>
**segments**<br>
	Array타입<br>
	첫번째, 두번째 인덱스에는 각각 시작 프레임과 정지프레임을 넣어준다. `ex. [0, 1]` <br><br>
**forceFlag**<br>
	Boolean 타입<br>
	false일 경우, 현재 세그먼트가 완료될 떄까지 기다린다. <br>
	true일 경우, 세그먼트 값을 바로 업데이트 한다. (인자를 true로 넘겨줘야 토글이 제대로 동작한다)<br><br>

#### 프레임 지정
- toggle 버튼이 시작점에서 ON -> OFF -> ON 순서에 따라 원점으로 돌아오는 동작을 기준으로 한다.
- 프레임의 초기값은 [0, 1] 로 넣어줘야 시작점에서 멈춰있는 상태로 대기하게 된다.
- 초기값을 [0, 0] 으로 지정할 경우 애니메이션이 루프를 한번 돌게 되므로 주의한다.
- ON -> OFF -> ON 각각의 상태에 따라 프레임 설정을 아래와 같이 지정하였다.<br><br>

|  ON(멈춤)  |ON → OFF(이동)|OFF(멈춤)|OFF → ON(이동)|
|------|---|---|---|
|[0, 1]|[0, 전체프레임/2]|[전체프레임/2, 전체프레임/2 + 1]|[전체프레임/2, 전체프레임]|

<br><br>

#### lottie-React
Componenet, Hook 2가지 타입으로 import해서 사용이 가능하다. 여기서는 Hook을 사용했다.<br><br>

**👉 Hook의 사용**

```js
 const style = {
    height: 36,
    width: 60,
  };
  const options = {
    loop: false,
    autoplay: false,
    animationData: jsonData,
    rendererSettings: {
      preserveAspectRatio: 'xMidYMid slice',
    },
  };

const { View, playSegments, setSpeed } = useLottie(options, style);
```
<br>
**👉 jsx 리턴**

```js
  /*
   View의 형태는 아래와 같다. lottie-react github 참고
    const View = <div style={style} ref={animationContainer} {...rest} />;
  */
  
  return (
    <Wrapper
      onClick={onToggleHandler}>
      {View}
    </Wrapper>
  );
```

<br><br>

**👉 State 관리 및 playSegments의 사용**
- playSegments을 호출해야 애니메이션 프레임이 움직인다. 
- forceFlag 값은 true여야 동작에 문제가 없다.

```js
const state = useLocalObservable(
  (): IState => ({
    sequence: {
      segments: initialSegments,
      forceFlag: true,
    },
    setSequence(start: number, stop: number) {
      state.sequence = {
        segments: [start, stop],
        forceFlag: true,
      };
      playSegments([start, stop], true);
    },
  })
);
```
<br><br>

### Reference.
<https://betterprogramming.pub/how-to-use-manipulate-airbnbs-lottie-animations-in-react-6e63b94012ca><br>
<https://lottiereact.com><br>
<https://github.com/Gamote/lottie-react><br>