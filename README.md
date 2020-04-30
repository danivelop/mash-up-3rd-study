### 리액트 엘리먼트 & virtual DOM

```javascript
import React, { useState, useCallback } from 'react';

function MyPage () {
  const [color, setColor] = useState('skyblue');
  
  const randomColor = useCallback(() => {
    ...
  }, []);
  
  const handleChangeColor = useCallback(() => {
    const newColor = randomColor();
    setColor(newColor);
  }, [randomColor]);
  
  return (
    <div>
	  <Title name="Mike" />
	  <p>제가 가장 좋아하는 색은 {color} 입니다.</p>
	  <button onClick={handleChangeColor}>색상 변경</button>
    </div>
  );
}

function Title ({ name }) {
  return (
    <h1>제 이름은 {name} 입니다.</h1>
  );
}
```

리액트를 어느정도 사용해보셨다면 위의 코드에 대한 결과는 쉽게 예측하실 수 있을거라고 생각합니다. 리액트는 주로 엘리먼트를 정의할 때, JSX 문법을 사용합니다.  JSX문법은 자바스크립트 표준 문법이 아니기 때문에 `babel`과 같은 트랜스 파일러를 통해 변환이 필요합니다. 결국, 위의 코드를 변환하게 되면, 아래와 같은 코드가 됩니다.

``` javascript
...

function MyPage () {

  ...
  
  return (
    React.createElement(
      'div',
      null,
      React.createElement(Title, { name: 'Mike' }),
      React.createElement('p', null, '제가 가장 좋아하는 색은 ', color, ' 입니다.'),
      React.createElement('button', { onClick: handleChangeColor }, '색상 변경'),
    )
  );
}

...
```

위와 같이 리액트는 `React.element(<엘리먼트>, <속성>, ...<자식 엘리먼트>)`를 통해 엘리먼트를 정의하게 됩니다. 

리액트는 렌더링을 효율적으로 하기 위해서 **virtual DOM**이라는 개념을 사용합니다. 바로 실제로 우리가 눈으로 볼 수 있는 화면으로 렌더링 하기 전, 렌더링을 하기 위한 엘리먼트들의 정보를 담고 있는 객체를 만들게 됩니다. 이 객체는 매번 리액트의 컴포넌트가 렌더링 될 때마다 생성되며 이전에 생성된 객체와 비교하여 실제로 변경 되어야 하는 부분을 찾아 돔 변경을 최소화 합니다.

리액트는 위의 코드를 어떻게 virtual DOM을 만드는지 보겠습니다.

```javascript
function () {
 return (
	<div>
	  <Title name="Mike" />
	  <p>제가 가장 좋아하는 색은 {color} 입니다.</p>
	  <button onClick={handleChangeColor}>색상 변경</button>
    </div>
  );
}
{
  type: 'div',
  key: null,
  ref: null,
  props: {
    children: [
      {
        type: Title,
        props: { name: 'Mike' }
        ...
      },
      {
        type: 'p',
        props: { children: ['제가 가장 좋아하는 색은 ', 'skyblue', ' 입니다.'] }
        ...
      },
      {
        type: 'button',
        props: {
          onClick: handleChangeColor
          children: '색상 변경'
        }
        ...
      }
    ]
  }
  ...
}
```
