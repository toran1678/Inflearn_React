

# State Lifting

![](https://velog.velcdn.com/images/minari0v0/post/518929ea-cc80-40a3-8890-ff46faba6457/image.png)

예시 형제 컴포넌트인 `Viewer`와 `Controller` 사이에는 서로의 state인 count나 setCount를 직접 접근하거나 공유할 수 없다

<br>

![](https://velog.velcdn.com/images/minari0v0/post/f48f488b-f3d0-42b8-b716-fc529db3c865/image.png)

이럴 때 `App`이라는 부모 컴포넌트에서 자식 컴포넌트, 단방향으로 데이터를 공유할 수 있다.

```app
App
 ├ Viewer
 └ Controller
```

- `Viewer` → `count` 전달

- `Controller` → `setCount` 전달

처럼 props로 내려보내는데 이걸 React에서 **단방향 데이터 흐름** 이라고 함

이렇게

> state를 공통 부모 컴포넌트로 올려서 여러 자식이 공유하게 하는 것

을 **State Lifting(State 끌어올리기)** 이라고 함

---

# Life Cycle

![](https://velog.velcdn.com/images/minari0v0/post/53fe3256-d281-4f7e-a46e-60223be89a3b/image.png)

- 리액트의 컴포넌트는 위처럼 세 단계로 구분되는 라이프 사이클을 가짐

## Mount

- 컴포넌트가 탄생하는 순간
- 화면에 처음 렌더링 되는 순간
  - "A 컴포넌트가 Mount 되었다."
  
## Update

- 컴포넌트가 다시 렌더링 되는 순간
- **리렌더링 될 때**를 의미
- 다음과 같은 상황일 때 발생

  - state가 변경될 때
  - props가 변경될 때
  - 부모 컴포넌트가 리렌더링될 때
  - context 값이 변경될 때


## UnMount

- 컴포넌트가 화면에서 사라지는 순간
- **렌더링에서 제외**되는 순간

---

## useEffect
- 리액트 컴포넌트의 **사이드 이펙트**를 제어하는 새로운 리액트 훅



### 사이드 이펙트
- 컴포넌트 동작에 따라 파생되는 여러 효과


| 동작 | 사이드 이펙트 |
|---|---|
| Mount | console.log("Mount") |
| Update | console.log("Update") |
| Unmount | console.log("Unmount") |

```js
// 예시
useEffect(() => {
  console.log("Mount")

  return () => {
    console.log("Unmount")
  }
}, [])
```

### Dependency Array
- useEffect 두 번째 인자로 전달하는 배열

| 형태      | 실행 시점         |
| ------- | ------------- |
| 없음      | 렌더링마다 실행      |
| []      | Mount 시 한번 실행 |
| [state] | state 변경 시 실행 |


```js
// 매 렌더링마다 실행
useEffect(() => {
  console.log("render")
})

// mount 한번만
useEffect(() => {
  console.log("mount")
}, [])

// 특정 값 변경 시
useEffect(() => {
  console.log("count update")
}, [count])
```

---

### Cleanup 정리 함수

- `useEffect`에서 **return으로 전달하는 함수**
- effect가 종료될 때 실행되는 함수

<br>

다음 상황에서 실행됨

- 컴포넌트가 **Unmount 될 때**
- effect가 **다시 실행되기 직전**

<br>

주로 아래 같은 작업을 정리할 때 사용됨

- 이벤트 리스너 제거
- 타이머 제거
- 구독(subscription) 해제

<br>

```js
useEffect(() => {
  const id = setInterval(() => {
    console.log("running")
  }, 1000)

  // cleanup
  return () => {
    clearInterval(id)
  }
}, [])
```

---

### useEffect 실행 타이밍

- `useEffect`는 **컴포넌트가 렌더링 된 이후 실행됨**
- **화면이 먼저 그려지고 나서 실행되는 코드**

<br>

보통 아래와 같은 작업을 처리할 때 사용됨

- API 요청
- 이벤트 등록
- 타이머 설정
- 외부 데이터 동기화

```js
useEffect(() => {
  fetch("/api/data")
    .then(res => res.json())
    .then(data => console.log(data))
}, [])
```

