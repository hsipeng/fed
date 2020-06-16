# React hook

## Hook 简介
*Hook* 是 React 16.8 的新增特性。它可以让你在不编写 class 的情况下使用 state 以及其他的 React 特性。

### 那么，什么是 Hook?
Hook 是一些可以让你在函数组件里“钩入” React state 及生命周期等特性的函数。Hook 不能在 class 组件中使用 —— 这使得你不使用 class 也能使用 React。（我们 [不推荐](https://zh-hans.reactjs.org/docs/hooks-intro.html#gradual-adoption-strategy) 把你已有的组件全部重写，但是你可以在新组件里开始使用 Hook。）


```javascript
import React, { useState } from ‘react’;

function Example() {
  // 声明一个新的叫做 “count” 的 state 变量
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}

```

## State Hook
useState 就是一个 *Hook* （等下我们会讲到这是什么意思）。通过在函数组件里调用它来给组件添加一些内部 state。React 会在重复渲染时保留这个 state。useState 会返回一对值：**当前**状态和一个让你更新它的函数，你可以在事件处理函数中或其他一些地方调用这个函数。它类似 class 组件的 this.setState，但是它不会把新的 state 和旧的 state 进行合并。（我们会在 [使用 State Hook](https://zh-hans.reactjs.org/docs/hooks-state.html)  里展示一个对比 useState 和 this.state 的例子）

```javascript
import React, { useState } from ‘react’;
function Example() {
  // 声明一个叫 “count” 的 state 变量。
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```


## Effect Hook
你之前可能已经在 React 组件中执行过数据获取、订阅或者手动修改过 DOM。我们统一把这些操作称为“副作用”，或者简称为“作用”。
useEffect 就是一个 Effect Hook，给函数组件增加了操作副作用的能力。它跟 class 组件中的 componentDidMount、componentDidUpdate 和 componentWillUnmount 具有相同的用途，只不过被合并成了一个 API。（我们会在 [使用 Effect Hook](https://zh-hans.reactjs.org/docs/hooks-effect.html)  里展示对比 useEffect 和这些方法的例子。）

```javascript
import React, { useState, useEffect } from ‘react’;
function Example() {
  const [count, setCount] = useState(0);

  // 相当于 componentDidMount 和 componentDidUpdate:
  useEffect(() => {
    // 使用浏览器的 API 更新页面标题
    document.title = `You clicked ${count} times`;
  });
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```


## 自定义 Hook
有时候我们会想要在组件之间重用一些状态逻辑。目前为止，有两种主流方案来解决这个问题： [高阶组件](https://zh-hans.reactjs.org/docs/higher-order-components.html) 和  [render props](https://zh-hans.reactjs.org/docs/render-props.html) 。自定义 Hook 可以让你在不增加组件的情况下达到同样的目的。

```javascript
import React, { useState, useEffect } from ‘react’;

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}


function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);
  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```
## 其他 Hook
*  [useContext](https://zh-hans.reactjs.org/docs/hooks-reference.html#usecontext)  让你不使用组件嵌套就可以订阅 React 的 Context。
*  [useReducer](https://zh-hans.reactjs.org/docs/hooks-reference.html#usereducer)  可以让你通过 reducer 来管理组件本地的复杂 state。

## Hook 使用规则
Hook 就是 JavaScript 函数，但是使用它们会有两个额外的规则：
* 只能在**函数最外层**调用 Hook。不要在循环、条件判断或者子函数中调用。
* 只能在 **React 的函数组件**中调用 Hook。不要在其他 JavaScript 函数中调用。（还有一个地方可以调用 Hook —— 就是自定义的 Hook 中，我们稍后会学习到。）
同时，我们提供了  linter 插件 来自动执行这些规则。这些规则乍看起来会有一些限制和令人困惑，但是要让 Hook 正常工作，它们至关重要。

[linter 插件](https://www.npmjs.com/package/eslint-plugin-react-hooks)


## 综合应用
包含axios 请求数据, useReducer useEffect 相结合使用


```javascript
import React, { useState, useEffect, useReducer } from “react”;
import ReactDOM from “react-dom”;
import axios from “axios”;

const dataFetchReducer = (state, action) => {
  switch (action.type) {
    case “FETCH_INIT”:
      return {
        …state,
        isLoading: true,
        isError: false
      };
    case “FETCH_SUCCESS”:
      return {
        ...state,
        isLoading: false,
        isError: false,
        data: action.payload
      };
    case "FETCH_FAILURE":
      return {
        ...state,
        isLoading: false,
        isError: true
      };
    default:
      throw new Error();
  }
};

const useDataApi = (initialUrl, initialData) => {
  const [url, setUrl] = useState(initialUrl);

  const [state, dispatch] = useReducer(dataFetchReducer, {
    isLoading: false,
    isError: false,
    data: initialData
  });

  useEffect(() => {
    const fetchData = async () => {
      dispatch({ type: "FETCH_INIT" });

      try {
        const result = await axios(url);

        dispatch({ type: “FETCH_SUCCESS”, payload: result.data });
      } catch (error) {
        dispatch({ type: “FETCH_FAILURE” });
      }
    };

    fetchData();
  }, [url]);
  return [state, setUrl];
};

// const App = () => {
//   const [data, setData] = useState({ hits: [] });
//   const [query, setQuery] = useState("redux");
//   const [search, setSearch] = useState(“redux”);
//   const [isLoading, setIsLoading] = useState(false);
//   useEffect(() => {
//     const fetchData = async () => {
//       setIsLoading(true);
//       const result = await axios(
//         `https://hn.algolia.com/api/v1/search?query=${search}`
//       );
//       setData(result.data);
//       setIsLoading(false);
//     };
//     fetchData();
//   }, [search]);
//   return (
//     <div>
//       <input onChange={(event) => setQuery(event.target.value)} value={query} />
//       <button type="button" onClick={() => setSearch(query)}>
//         Search
//       </button>
//       {isLoading ? (
//         <div>Loading… </div>
//       ) : (
//         <ul>
//           {data.hits.map(function (item) {
//             return <li>{item.title}</li>;
//           })}
//         </ul>
//       )}
//     </div>
//   );
// };

const App = () => {
  const [query, setQuery] = useState("redux");
  const [{ data, isLoading, isError }, doFetch] = useDataApi(
    "https://hn.algolia.com/api/v1/search?query=redux",
    { hits: [] }
  );
  return (
    <div>
      <input onChange={event => setQuery(event.target.value)} value={query} />
      <button
        type=“button”
        onClick={() =>
          doFetch(`https://hn.algolia.com/api/v1/search?query=${query}`)
        }
      >
        Search
      </button>
      {isError && <div>Something went wrong …</div>}
      {isLoading ? (
        <div>Loading... </div>
      ) : (
        <ul>
          {data.hits.map(function(item) {
            return <li>{item.title}</li>;
          })}
        </ul>
      )}
    </div>
  );
};

const rootElement = document.getElementById("root");
ReactDOM.render(<App />, rootElement);


```



## 参考资料

* [官方](https://zh-hans.reactjs.org/docs/hooks-intro.html)

* [Dan的《useEffect完全指南》](https://overreacted.io/zh-hans/a-complete-guide-to-useeffect/)

* [衍良同学的《React Hooks完全上手指南》](https://zhuanlan.zhihu.com/p/92211533)

* [hooks fetch data](https://www.robinwieruch.de/react-hooks-fetch-data)

* [自定义react hooks](https://juejin.im/post/5be3ea136fb9a049f9121014)

* [实际案例](https://juejin.im/post/5d594ea5518825041301bbcb)

