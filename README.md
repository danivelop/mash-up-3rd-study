# React의 렌더링 과정
React를 사용해 개발을 하다보면 한 컴포넌트가 렌더링을 연속으로 여러번 하거나, 결과가 의도치 않게 나오는 경우가 종종 있습니다. 혹은, 최적화 등 성능적으로 고민해야 할 때도 있습니다. 저도 처음에는 단순히 React의 사용법만 익히고 개발했지만 위와 같은 경우를 겪었을 때, 해결하기 위해선 React가 어떻게 렌더링되고 DOM의 변경을 최소화 하는지 등 더 깊게 알아야 했습니다.
그래서 React의 렌더링 과정과 virtual DOM이 어떻게 동작하는지 공부한 것들을 정리했습니다.

> 혹시 틀린 내용이나 수정할 부분이 있으면 언제든지 말씀해 주세요.

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
### 컴포넌트의 업데이트
React에서 리렌더링이 일어나는 경우는 크게 3가지가 있습니다. (이 외에도 더 있지만 글에서는 3가지만 다루겠습니다.)

- 컴포넌트의 `state`가 변경된 경우
- 컴포넌트의 `props`가 변경된 경우
- 부모 컴포넌트가 리렌더링 된 경우

그리고 React에서 컴포넌트의 업데이트는 2가지 단계로 나뉩니다.

- 렌더단계 : 실제 DOM에 반영하기 전, 변경사항을 파악하는 단계
- 커밋단계 : 렌더단계에서 파악된 변경사항을 실제 DOM에 반영하는 단계

위의 [예제코드](#code-example)에서 '이름 변경' 버튼을 클릭할 경우, 렌더링 관련하여 어떤 일이 일어나는지 살펴보겠습니다. '이름 변경' 버튼을 클릭하면 `handleChangeName`함수가 호출되고 `setName`함수에 의해 `MyPage`컴포넌트의 상탯값이 변경됩니다. 상탯값이 변경됐으니 `MyPage`컴포넌트는 리렌더링을 하게 됩니다. 이렇게 되면 자식 컴포넌트인 `Color`, `Button` 컴포넌트도 함께 리렌더링을 하게 됩니다.

여기서 중요한 점은 `Color`컴포넌트는 먼저 렌더단계를 거치게 되는데 렌더단계에서는 `Color`컴포넌트에 대한 Virtual DOM을 다시 만들게 되고 이를 이전에 만들어진 Virtual DOM과 비교합니다. 하지만, 위의  [예제코드](#code-example)에서는 `name`을 변경해도 `color`는 바뀌지 않기 때문에 결국 `Color`컴포넌트의 `props`로 전달된 `color`는 바뀌지 않습니다. 즉, `Color`컴포넌트는 실제로 변경된 사항이 없으니 렌더단계는 거치더라도 커밋단계에서는 하는일이 없게 됩니다.

위의 내용을 정리해 보겠습니다.

1. '이름 변경' 버튼을 클릭한다.
2. `MyPage`컴포넌트의 상태가 변하고 리렌더링 된다.
3. 자식 컴포넌트인 `Color`, `Button` 컴포넌트도 리렌더링을 한다.
4. `Color`컴포넌트는 Virtual DOM을 만들고 이전값과 비교하는 렌더단계를 거친다.
5. 실제로 변경된 사항은 없으므로 커밋단계에서는 업데이트 비용이 들지 않는다.

### 최적화
'이름 변경' 버튼을 클릭하여 `name`이 변경됨으로 인해 `Color`컴포넌트가 렌더단계는 거치더라도 실제 DOM에는 반영을 하지 않으므로 성능에 큰 문제는 없습니다. 하지만, 만약 `Color`컴포넌트가 매우 복잡한 컴포넌트라서 Virtual DOM을 생성하고 이전에 생성된 Virtual DOM과 비교하는 작업도 비용이 많이 들어갈 수 있습니다. 이 때, `props`가 변하지 않았다면 이런 과정도 생략을 해줄 수 있습니다.

```javascript
...

function MyPage () {
  ...
}

export default React.memo(MyPage);
```

```javascript
import React from 'react';

class MyPage extends React.PureComponent {
  ...
}

export default MyPage;
```

위의 코드와 같이 functional 컴포넌트에서는 `React.memo`사용하거나 class 컴포넌트에서는 `React.PureComponent`를 상속받게 되면 해당 컴포넌트의 `props`가 변경되지 않았다면 더 이상 렌더링을 진행하지 않습니다. 다시 말해, '이름 변경' 버튼을 클릭하여 `MyPage`컴포넌트가 리렌더링 되어도 `Color`컴포넌트의 `props`로 전달되는 `color`는 변하지 않았으므로 `Color`컴포넌트는 렌더링 단계를 거치지 않습니다.

하지만, 아직 한가지 더 신경 써줘야 할 부분이 있습니다.

```javascript
import React from 'react';

class MyPage extends React.Component {
  ...
  
  handleChangeName = () => { ... };
  handleChangeColor = () => { ... };
  
  render () {
    ...
  }
}
```

```javascript
import React from 'react';

function MyPage () {
  ...
  
  const handleChangeName = () => { ... };
  const handleChangeColor = () => { ... };
  
  return (
    ...
  )
}
```

class 컴포넌트는 초기 생성될 때 인스턴스 형태로 생성되기 때문에 리렌더링 되는 경우에도 class 내부의 `render`함수만 다시 호출할 뿐입니다. 즉, 멤버함수인 `handleChangeName`와 `handleChangeColor`는 리렌더링시에도 다시 재할당되지 않습니다. 하지만 functional 컴포넌트는  리렌더링 되면 컴포넌트 자체를 호출하기 때문에 `handleChangeName`와 `handleChangeColor`가 매번 재할당 됩니다. 따라서, `MyPage`컴포넌트가 리렌더링 될 때마다 `Button`컴포넌트의 `props`로 전달되는 `handleChangeColor`가 재할당 되기 때문에 `Button`컴포넌트는 불필요하게 렌더단계를 거치게 됩니다.

이렇게 functional 컴포넌트에서 매번 함수가 새로 할당되는 경우를 막기 위해 React에서는 `useCallback` hooks를 제공합니다.

```javascript
import React, { useCallback } from 'react';

function MyPage () {
  ...

  const handleChangeName = useCallback(() => {
    ...
  }, []);
  
  const handleChangeColor = useCallback(() => {
    ...
  }, []);

  ...
}

...
```

`useCallback`은 2번째 인자로 넣어준 배열안의 데이터가 변경되지 않았다면 함수를 새로 생성하지 않고 이전에 있던 값을 재사용 합니다. 따라서 '이름 변경' 버튼클 클릭해 `MyPage`컴포넌트가 리렌더링 되도 `handleChangeColor`는 초기에 할당 되었던 값 그대로 재사용되므로 `Button`컴포넌트도 렌더링 단계를 거치지 않게 됩니다.

> React는 이렇게 `props`가 실제로 변경되었는지에 따라 렌더링 과정이 달라지는데 위의 예시에서도 `React.memo`를 통해 `props`가 변경되지 않았다면 렌더링 과정을 진행하지 않는것을 보았습니다. 그럼 React는 어떻게 변경사항을 효율적으로 파악할까요? 여기서 객체의 불변성이라는 개념이 등장합니다.

### 객체의 불변성
`Color`컴포넌트의 변경사항을 파악하기 위해서는 `Color`컴포넌트가 렌더단계를 거치며 새로 전달된 `props`를 이전에 존재했던 `props`와 비교해야 하는데 위의 [예제코드](#code-example)에서는 `props`로 전달된 `color`가 단순 문자열이기 때문에 `prevProps.color === nextProps.color`와 같이 단순 비교로 실제로 변경되었는지 알 수 있습니다.

하지만, 만약 `Color`컴포넌트로 전달된 속성인 `color`가 문자열이 아닌 객체와 같은 참조타입 이었다면 어떻게 됐을까요? 예를 한번 들어보겠습니다.

```javascript
const [colors, setColors] = useState(['Skyblue', 'White', 'Rosegold']);

//1번
onClick1 = () => {
  colors.push('red');
  setColors(colors);
}

//2번
onClick2 = () => {
  const newColors = [...colors, 'red'];
  setColors(newColors);
}
```

위의 예시에서 1번에서는 `colors`내부의 데이터가 실제로 변경되었는지 파악하기 위해선 `colors`의 모든 인덱스를 검사해야 합니다. `colors`안의 인덱스가 100만개라면 100만번 검사해야 하죠. 하지만 2번에서는 `prevColors === nextColors`와 같이 한번의 연산만으로 검사가 가능합니다. 이와 같은 이유로 React는 객체, 배열과 같은 참조타입은 데이터 변경시 새로운 객체를 할당하여 불변성을 유지합니다.

이렇게 불변성을 유지하게 되면

```javascript
const prevProps = {
  names: ['Daniel', 'Sam', 'Mike'],
  colors: ['Skyblue', 'White', 'Rosegold']
}
const nextProps = {
  names: ['Daniel', 'Sam', 'Mike'],
  colors: ['Skyblue', 'White', 'Rosegold', 'red']
}
```

와 같이 `props` 객체에 직접 연결(1-depth)되 있는 값들만 단순비교하면 `props`의 변경 여부를 알 수 있습니다.

---

이렇게 React가 어떤 과정을 통해 렌더링 되고 어떻게 효율적이고 최소한으로 렌더링을 하는지 정리했습니다. React는 이 외에도 서비스를 효율적으로 만들기 위해 더 다양한 방법들을 많이 제공하지만 이 글에서 다룬 가장 기초가 되는 개념들을 알고 있다면 React의 더 깊은 개념을 학습하는데 조금 더 수월할 거라고 생각합니다.
