---
layout: post
title: react-query, react-table 와 상태관리
description: react-query, react-table 를 사용하면서 겪은 문제점에 대한 글입니다. (해결중)
summary: react-query, react-table 를 사용하면서 겪은 문제점에 대한 글입니다. (해결중)
comments: true
tags: [react-table, react-query, react]
---

이번 새로운 프로젝트를 진행하면서 **react-query** 를 처음 사용하고 있는 중인데 <br>
역시 라이브러리가 가진 개념과 방향을 이해하는 게 너무 중요하다는 것을 깨닫는 중이다.

**react-query** 를 사용해서 데이터를 **fetch** 할 때는 되도록 **state** 에 저장해서 관리하지 않아야 할 것 같다.<br>
생각해보면 그럴 필요도 없고 **state** 값이 업데이트 되면서 원치 않는 이펙트가 발생되기도 한다.<br>
(애초에 **react-query** 를 쓰면서 왜 **state** 으로 관리 할 생각을 했는지.. 나자신이여..😂)<br><br>

현재 공통적으로 문제가 되는 부분은 **state** 이 업데이트 되면서 그 사이에 기존 값이 보였다가 그 후에 변경이 문제이다.<br><br>

select 컴포넌트에서 옵션으로 사용될 데이터를 **useQuery** 로 가져오고 그 데이터 값을<br>
특정 함수를 통해 가공 후에 옵션 **prop** 으로 넘겨준다.<br>

여기서 이 옵션값을 **state** 로 저장할 필요가 없다. 이 불필요한 **state** 때문에 select 컴포넌트가 값이 변경 <br>
될 때마다 깜빡이면서 잠시 사이에 기존 값을 보여주고 난 후에 변경되게 된다. (으으 😵)
<br><br>

```tsx
const SomeSelect = ({queryKey, apiFn, renderFunction, ...someProps}) => {
  const [options, setOptions] = useState<someInterface[]>([]);

  const { data, isSuccess } = customHookForUseQuery({
    queryKey,
    queryFn: apiFn,
  });

  // 문제되는 부분
  useEffect(() => {
    if (isSuccess) {
      const option = renderFunction
        ? data?.map(renderFunction)
        : data?.map(defaultFunction);

      setOptions(option);
    }
  }, [isSuccess, queryKey]);

    return (
      <Select
        options={options}
        ref={ref}
        {...somePropsForSelect}
      />
    );
  }
};

```

<br>
아래 방법으로 수정하니 해결되었다. **useMemo** 를 써도 호출하는 횟수에 개선은 없어서 사용하지는 않았다.<br>

```tsx
const SomeSelect = ({queryKey, apiFn, renderFunction, ...someProps}) => {

  const { data, isSuccess } = customHookForUseQuery({
    queryKey,
    queryFn: apiFn,
  });

  // 변경한 부분
  const options = renderFunction ? data?.map(renderFunction) : data?.map(defaultFunction);

    return (
      <Select
        options={options}
        ref={ref}
        {...somePropsForSelect}
      />
    );
  }
};

```

<br><br>

#### 또 다른 문제는..?

테이블 각 **row** 의 **cell** 에서 **input** 으로 값을 수정해야 하는 화면이 있는데<br>
기존 프로젝트에서는 **react-hook-form** 이라는 폼 라이브러리를 사용했어서 이러한 문제가 없었는데<br>
라이브러리 없이 사용하니 생각지도 못한 문제점에 부딪혔다.

<br>

**react-table** 에서는 **editable data** 를 처리할때 다음과 같은 솔루션을 제안한다. <br>
(왜냐면 **onChange** 이벤트만을 사용 시 포커스가 **input** 밖으로 빠지는 버그가 있다)<br><br>

1. **onChange** 이벤트에서는 **cell** 내부에서 **local state** 으로 **value** 를 관리해서 즉시 업데이트 한다.
2. 이후 **onBlur** 이벤트에서는 업데이트된 값을 최종적으로 테이블 밖에 선언된 **로컬 state** 에 업데이트 한다.<br><br>

**onBlur** 로 로컬 state 값을 수정하고 난 뒤 다시 변경된 데이터가 react-table 컴포넌트에 전달되게 된다.<br>
그럼 테이블 훅에서는 다시 이 데이터를 받아 새로운 row 데이터를 리턴하고 각 table row 에 렌더링하는 일을 할 것이다.<br>
이 과정에는 문제가 없다. 문제는 이렇게 상태값이 변경될 때마다 순간적으로 기존 값을 보여주다가 업데이트가 된다는 것이다. <br>
신기한건 **react-table** 에서 보여주는 예시 프로젝트에는 이런 현상이 없는데 뭘까.
<br><br>

#### 그래서 생각해 본 해결방안은

1. 일부 문제되는 페이지에 대한 **react-table** 사용을 포기하기 -> _아주 쉬운 방법이긴 하지만.._
2. **react-hook-form** 라이브러리를 적용하기 -> _폼이 들어가는 페이지 모두 수정해야 하는 문제가 있다_
3. state 으로 관리하지 않고 폼 submit 할때 **FormData** 객체로 값을 생성시켜 서버에 sending 하기 -> _state 구조가 복잡하므로 FormData 로 생성시키는 작업이 딱히 적절하지 않아보인다._
4. what else..? <br><br>

항상 드는 생각이지만 리액트 동작에 대한 컨트롤은 여전히 어렵다는 것이다.<br>
아직 해결중이므로 나중에 어떻게 해결했는지 더 이어서 작성해야겠다.

<br><br>

<https://react-table-v7.tanstack.com/docs/examples/editable-data>
