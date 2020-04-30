# React의 렌더링 과정
React를 사용해 개발을 하다보면 한 컴포넌트가 렌더링을 연속으로 여러번 하거나, 결과가 의도치 않게 나오는 경우가 종종 있습니다. 혹은, 최적화 등 성능적으로 고민해야 할 때도 있습니다. 저도 처음에는 단순히 React의 사용법만 익히고 개발했지만 위와 같은 경우를 겪었을 때, 해결하기 위해선 React가 어떻게 렌더링되고 DOM의 변경을 최소화 하는지 등 더 깊게 알아야 했습니다.
그래서 React의 렌더링 과정과 virtual DOM이 어떻게 동작하는지 공부한 것들을 정리했습니다.
혹시 틀린 내용이나 수정할 부분이 있으면 언제든지 말씀해 주세요.

### Table of contents
- [React element](#react-element)
- [Virtual DOM](#virtual-dom)
- [React 리렌더링](#react-리렌더링)

## React element
먼저, React의 렌더링 과정을 살펴보기 위해 아주 간단한 예제를 만들어 보겠습니다.

#### code example

```
react-rendering-process
└── src
  ├── index.js
  └── components
    ├── MyPage.js
    ├── Color.js
    └── Button.js

```

```javascript
import React, { useState } from 'react';
import Color from './Color';
import Button from './Button';

const names = ['Daniel', 'Sam', 'Mike'];
const colors = ['Skyblue', 'White', 'Rosegold'];

const randomName = () => {
  const index = Math.floor(Math.random() * 3);
  return names[index];
};  

const randomColor = () => {
  const index = Math.floor(Math.random() * 3);
  return colors[index];
};

function MyPage () {
  const [name, setName] = useState('Daniel');
  const [color, setColor] = useState('Skyblue');

  const handleChangeName = () => {
    const newName = randomName();
    setName(newName);
  };
  
  const handleChangeColor = () => {
    const newColor = randomColor();
    setColor(newColor);
  };
  
  return (
    <div>
      <h1>제 이름은 {name} 입니다.</h1>
	  <button onClick={handleChangeName}>이름 변경</button>
	  <Color color={color} />
	  <Button onClick={handleChangeColor}  />
    </div>
  );
}

export default MyPage;
```

```javascript
import React from 'react';

function Color ({ color }) {
  return (
    <p>제가 가장 좋아하는 색은 {color} 입니다.</p>
  );
}

export default Color;
```

```javascript
import React from 'react';

function Button ({ onClick }) {
  return (
    <button  onClick={onClick}>색상 변경</button>
  );
}

export default Button;
```

React를 어느정도 사용해보셨다면 위의 코드에 대한 결과는 쉽게 예측하실 수 있을거라고 생각합니다. React는 주로 element를 정의할 때, JSX 문법을 사용합니다.  JSX문법은 자바스크립트 표준 문법이 아니기 때문에 babel과 같은 트랜스 파일러를 통해 변환이 필요합니다. 결국, 위의 코드를 변환하게 되면 아래와 같은 코드가 됩니다.

``` javascript
...

function MyPage () {

  ...
  
  return (
    React.createElement(
      'div',
      null,
      React.createElement('h1', null, '제 이름은 ', name, ' 입니다.'),
      React.createElement('button', { onClick:  handleChangeName }, '이름 변경'),
      React.createElement(Color, { color: color }),
      React.createElement(Button, { onClick: handleChangeColor }),
    )
  );
}

...
```

위와 같이 React는 `React.element(<element명>, <속성>, ...<자식element>)`를 통해 React element를 정의하게 됩니다. 

> React는 렌더링을 효율적으로 하기 위해서 **Virtual DOM**이라는 개념을 사용합니다. 바로 실제로 우리가 눈으로 볼 수 있는 화면으로 렌더링 하기 전, 렌더링을 하기 위한 element들의 정보를 담고 있는 객체를 만들게 됩니다. 이 객체는 매번 React의 컴포넌트가 렌더링 될 때마다 생성되며 이전에 생성된 객체와 비교하여 실제로 변경 되어야 하는 부분을 찾아 돔 변경을 최소화 합니다.

## Virtual DOM

그럼, React가 위의 [예제코드](#code-example)를 어떻게 Virtual DOM으로 만드는지 보겠습니다.

```javascript
{
  type: 'div', ─── ①
  key: null, ─── ②
  ref: null, ─── ③
  props: {
    children: [
      {
        type: 'h1',
        props: {
          children: ['제 이름은 ', 'Daniel', ' 입니다.'] ─── ④
        },
        ...
      },
      {
        type: 'button',
        props: {
          onClick: handleChangeName, ─── ⑤
          children: '이름 변경'
        },
        ...
      },
      {
        type: Color,
        props: {
          color: 'Skyblue'
        },
        ...
      },
      {
        type: Button,
        props: {
          onClick: handleChangeColor
        },
        ...
      }
    ]
  }
  ...
}
```

다음과 같이 **①** `type`에는 문자열 형태의 html 태그 혹은 컴포넌트가 들어가며 **②** `key`속성에는 해당 element가  `key`를 가지고 있을 경우, 할당됩니다.  **③** `ref`속성도 마찬가지로, 해당 element가 `ref`를 가지고 있다면 할당됩니다. `key`와 `ref`를 제외한 나머지 속성값은 `props`에 들어가게 되는데 그 중, `children`에는 해당 element의 자식element가 할당됩니다. **④** 만약, children에 표현식이 쓰였다면 표현식을 기점으로 분할되어 들어갑니다. **⑤** 또한, `props`로 배열, 객체, 함수 와 같은 참조타입 값이 주어지면 해당 값을 참조하는 값이 들어가게 됩니다.

하지만, 아직 Virtual DOM을 완성하지 못했습니다. React가 Virtual DOM을 통해 실제 DOM을 생성하기 위해서는 Virtual DOM의 모든 `type`이 문자열 즉, html태그여야 합니다. 하지만, 위의 예시에서는 아직 `type`에 `Color`와 `Button`이 남아있습니다. 이들 모두 컴포넌트이기 때문에 functional 컴포넌트라면 그대로 호출하고 class 컴포넌트라면 `render`함수를 호출하여 해당 컴포넌트의 렌더링에 대한 정보를 얻을 수 있습니다. 이 과정을 통해 최종적으로 완성된 Virtual DOM은 다음과 같습니다.

```javascript
<p>제가 가장 좋아하는 색은 {color} 입니다.</p>
<button  onClick={onClick}>색상 변경</button>
{
  type: 'div',
  key: null,
  ref: null,
  props: {
    children: [
      {
        type: 'h1',
        props: {
          children: ['제 이름은 ', 'Daniel', ' 입니다.']
        },
        ...
      },
      {
        type: 'button',
        props: {
          onClick: handleChangeName,
          children: '이름 변경'
        },
        ...
      },
      {
        type: 'p',
        props: {
          children: ['제가 가장 좋아하는 색은 ', 'Skyblue', ' 입니다.']
        },
        ...
      },
      {
        type: 'button',
        props: {
          onClick: handleChangeColor
          children: '색상 변경'
        },
        ...
      }
    ]
  }
  ...
}
```

React는 렌더링을 할 때 마다 이러한 과정으로 Virtual DOM을 생성하고 이전에 생성된 Virtual DOM과 비교하여 변경된 부분을 찾아냅니다. 그럼, React는 어떻게 변경된 부분을 찾아내는지 보겠습니다.

## React 리렌더링
React에서 리렌더링이 일어나는 경우는 크게 3가지가 있습니다. (이 외에도 더 있지만 이 글에서는 3가지만 다루겠습니다.)

- 부모 컴포넌트가 리렌더링 된 경우
- 컴포넌트의 `state`가 변경된 경우
- 컴포넌트의 `props`가 변경된 경우

위의 [예제코드](#code-example)에서 

객체의 불변성

렌더단계
커밋단계

useCallback, React.memo
