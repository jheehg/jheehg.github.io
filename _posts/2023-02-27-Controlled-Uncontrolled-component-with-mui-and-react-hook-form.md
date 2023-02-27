---
layout: post
title: material UI 에 react-hook-form 적용 시 주의할 점
description: material UI 에 react-hook-form 적용했을 때 겪은 문제점에 대해 공부한 내용입니다.
summary: material UI 에 react-hook-form 적용했을 때 겪은 문제점에 대해 공부한 내용입니다.
comments: true
categories: [FE]
tags: [react-hook-form, react, material UI]
---

<br>

### 문제 상황

- 폼 컴포넌트가 사용자의 동작으로 인해 데이터가 변경이 되어야 하는데 값이 바뀌지 않음.
- 세부 동작
  - 👉 `react-hook-form` 의 `useForm` 으로 폼 데이터를 관리 (ex. select, radio, input ..)
  - 👉 특정 함수가 실행될 때 마다 사용자에게 입력 받은 데이터를 `reset` 에 넣고 실행하여 `react-hook-form` 에서 관리하는 값을 변경해주는데 정작 폼 컴포넌트 자체는 변화가 없음.

<br>

### 원인

- `mui` 와 같은 `controlled component` 에 `react-hook-form` 를 적용할 때는 `Controller` 라는 Wrapper 컴포넌트를 사용해야 함.
- 아래 설명과 함께 **Controller**, **Custom Register** 두가지 방법으로 처리가 가능하다는 예시가 나와있다.

<br>

> ### Controlled mixed with Uncontrolled Components
>
> React Hook Form 은 uncontrolled components 를 수용하지만 controlled components 와도 호환이 가능하다.
> mui, Antd 와 같은 대부분의 UI 라이브러리는 controlled components 를 지원하도록 구축되어 있다.
> 그러나 React Hook Form 을 적용하면 controlled components 의 리렌더링도 또한 optimize 가 가능하다.

<https://react-hook-form.com/advanced-usage#ControlledmixedwithUncontrolledComponents>

<br>

\+ 추가로 참고할 내용

React Hook Form 는 uncontrolled components 와 native inputs 을 지원하는데
React-Select, AntD and MUI와 같은 controlled component 에도 적용을 하기 위해서는 Controller 라는 wrapper 를 사용할 수 있다.

이미 Controller 자체로도 registration 과정을 거치고 있으니 input 에 다시 register 를 사용하지 않아도 된다.

<https://react-hook-form.com/api/usecontroller/controller/#example>

<br><br>

생각해보면 Controllered Component, Uncontrolled Component 에 대해 정확하게 이해를 못한 상태에서 <br>
사용하고 있던 것 같아 개념을 다시 정리해보았다.

<br><br>

### Controllered Component vs UnControlled Component

#### Controllered Component

폼 컴포넌트 (input, checkbox, or select..) 의 상태에 대한 매니징은 서로 다른 방법으로 할 수 있다.<br>
`Controllered component` 는 매니징을 리액트 동작에 맡기는 방법이다. <br>
사용자가 입력값을 바꾸면 리액트의 로컬상태 값도 같이 바뀌기 때문에 철저하게 로컬상태와 동기화 된다. <br>
폼에 값이 입력되면 `onChange` 이벤트 핸들러를 통해 `value` 값을 변경한다. <br><br>

#### UnControlled Component

반면 `UnControllered component` 의 경우 `DOM` 에 의해서 관리되고 `ref` 를 통해서 이 컴포넌트에 접근을 한다. <br>
컴포넌트의 값을 바꾸고 싶으면 해당 `DOM node` 에 직접 접근해서 값을 바꾸게 되는데 이는 관리를 좀 더 까다롭게 만들기도 한다.<br><br>

react-hook-form 에서 제공하는 Controller 컴포넌트를 사용하여 변경하였다.

```js
return (
   <FormControl>
      <FormLabel id=“select-label”>{label}</FormLabel>
         <Controller
            control={control}
            name={name}
            render={({ field }) => {
               return (
                  <Select {...field}>
                     {data?.map((option) => (
                        <MenuItem key={option.value} value={option.value}>
                           {option.name}
                        </MenuItem>
                     ))}
                  </Select>
               );
            }}
         />
   </FormControl>
);
```

<br>

### 정리

- mui 컴포넌트를 react-hook-form 으로 관리할 경우 controlled component 를 uncontrolled component 방법으로 제어 중인지 확인한다.
- controlled component 방법을 적용할 경우 `Controller` 라는 Wrapper component 를 사용한다.
- 그 외에도 값이 의도한 대로 변경이 되지 않을 경우 `ref` 로 컴포넌트가 관리되는지 확인이 필요하다.

<br>
