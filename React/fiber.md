# React Fiber
React 是一个用于构建用户交互界面的 JavaScript 库，其核心  [机制](https://medium.freecodecamp.org/what-every-front-end-developer-should-know-about-change-detection-in-angular-and-react-508f83f58c6a?gi=b442b00cd6c3)  就是跟踪组件的状态变化，并将更新的状态映射到到新的界面。在 React 中，我们将此过程称之为**协调**。我们调用 setState 方法来改变状态，而框架本身会去检查 state 或 props 是否已经更改来决定是否重新渲染组件。
React 的官方文档对  [协调机制](https://reactjs.org/docs/reconciliation.html)  进行了良好的抽象描述： React 的元素、生命周期、 render 方法，以及应用于组件子元素的 diffing 算法综合起到的作用，就是协调。从 render 方法返回的不可变的 React 元素通常称为「虚拟 DOM」。这个术语有助于早期向人们解释 React，但它也引起了混乱，并且不再用于 React 文档。在本文中，我将坚持称它为 React 元素的树。
除了 React 元素的树之外，框架总是在内部维护一个实例来持有状态（如组件、 DOM 节点等）。从版本 16 开始， React 推出了内部实例树的新的实现方法，以及被称之为 **Fiber** 的算法。如果想要了解 Fiber 架构带来的优势，可以看下  [React 在 Fiber 中使用链表的方式和原因](https://medium.com/dailyjs/the-how-and-why-on-reacts-usage-of-linked-list-in-fiber-67f1014d0eb7) 。

### Fiber 节点
在协调期间，从 render 方法返回的每个 React 元素的数据都会被合并到 Fiber 节点树中。每个 React 元素都有一个相应的 Fiber 节点。与 React 元素不同，不会在每次渲染时重新创建这些 Fiber 。这些是持有组件状态和 DOM 的可变数据结构。
我们之前讨论过，根据不同 React 元素的类型，框架需要执行不同的活动。在我们的示例应用程序中，对于类组件 ClickCounter ，它调用生命周期方法和 render 方法，而对于 span 宿主组件（DOM 节点），它进行得是 DOM 修改。因此，每个 React 元素都会转换为  [相应类型](https://github.com/facebook/react/blob/769b1f270e1251d9dbdce0fcbd9e92e502d059b8/packages/shared/ReactWorkTags.js)  的 Fiber 节点，用于描述需要完成的工作。
**您可以将 Fiber 视为表示某些要做的工作的数据结构，或者说，是一个工作单位。Fiber 的架构还提供了一种跟踪、规划、暂停和销毁工作的便捷方式。**

### current 树及 workInProgress 树
在第一次渲染之后，React 最终得到一个 Fiber 树，它反映了用于渲染 UI 的应用程序的状态。这棵树通常被称为 **current 树（当前树）**。当 React 开始处理更新时，它会构建一个所谓的 **workInProgress 树（工作过程树）**，它反映了要刷新到屏幕的未来状态。
所有工作都在 workInProgress 树的 Fiber 节点上执行。当 React 遍历 current 树时，对于每个现有 Fiber 节点，React 会创建一个构成 workInProgress 树的备用节点，这一节点会使用 render 方法返回的 React 元素中的数据来创建。处理完更新并完成所有相关工作后，React 将准备好一个备用树以刷新到屏幕。一旦这个 workInProgress树在屏幕上呈现，它就会变成 current 树。
React 的核心原则之一是一致性。 React 总是一次性更新 DOM - 它不会显示部分中间结果。workInProgress 树充当用户不可见的「草稿」，这样 React 可以先处理所有组件，然后将其更改刷新到屏幕。


### Fiber 树的根节点
每个 React 应用程序都有一个或多个充当容器的 DOM 元素。在我们的例子中，它是带有 ID 为 container 的 div元素。React 为每个容器创建一个  [Fiber 根](https://github.com/facebook/react/blob/0dc0ddc1ef5f90fe48b58f1a1ba753757961fc74/packages/react-reconciler/src/ReactFiberRoot.js#L31)  对象。您可以使用对 DOM 元素的引用来访问它：

```javascript
const fiberRoot = query('#container')._reactRootContainer._internalRoot

```

### Fiber 节点结构

```javascript
{
    stateNode: new ClickCounter,
    type: ClickCounter,
    alternate: null,
    key: null,
    updateQueue: null,
    memoizedState: {count: 0},
    pendingProps: {},
    memoizedProps: {},
    tag: 1,
    effectTag: 0,
    nextEffect: null
}

```

## 通用算法
React 在两个主要阶段执行工作：render 和 commit。

第一个 render 阶段的工作是可以异步执行的。**React 可以根据可用时间片来处理一个或多个 Fiber 节点，然后停下来暂存已完成的工作，并转而去处理某些事件，接着它再从它停止的地方继续执行。但有时候，它可能需要丢弃完成的工作并再次从顶部开始。由于在此阶段执行的工作不会导致任何用户可见的更改（如 DOM 更新），因此暂停行为才有了意义。**与之相反的是，后续 commit 阶段始终是同步的。**这是因为在此阶段执行的工作会导致用户可见的变化，例如 DOM 更新。这就是为什么 React 需要在一次单一过程中完成这些更新。

React 要做的一种工作就是调用生命周期方法。一些方法是在 render 阶段调用的，而另一些方法则是在 commit阶段调用。这是在第一个 render 阶段调用的生命周期列表：
* [UNSAFE_]componentWillMount（弃用）

* [UNSAFE_]componentWillReceiveProps（弃用）

* getDerivedStateFromProps

* shouldComponentUpdate

* [UNSAFE_]componentWillUpdate（弃用）

* render


### Render 阶段

协调算法始终使用  [renderRoot ](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L1132) 函数从最顶层的 HostRoot 节点开始。不过，React 会略过已经处理过的 Fiber 节点，直到找到未完成工作的节点。例如，如果在组件树中的深层组件中调用 setState 方法，则 React 将从顶部开始，但会快速跳过各个父项，直到它到达调用了 setState 方法的组件。



### 工作循环的主要步骤
nextUnitOfWork 持有 workInProgress 树中的 Fiber 节点的引用，这个树有一些工作要做。当 React 遍历 Fiber 树时，它会使用这个变量来知晓是否有任何其他 Fiber 节点具有未完成的工作。处理过当前 Fiber 后，变量将持有树中下一个 Fiber 节点的引用或 null。在这种情况下，React 退出工作循环并准备好提交更改。
遍历树、初始化或完成工作主要用到 4 个函数：
*  [performUnitOfWork](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L1056) 

*  [beginWork](https://github.com/facebook/react/blob/cbbc2b6c4d0d8519145560bd8183ecde55168b12/packages/react-reconciler/src/ReactFiberBeginWork.js#L1489) 

*  [completeUnitOfWork](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L879) 

*  [completeWork](https://github.com/facebook/react/blob/cbbc2b6c4d0d8519145560bd8183ecde55168b12/packages/react-reconciler/src/ReactFiberCompleteWork.js#L532) 


```javascript
function workLoop(isYieldy) {
  if (!isYieldy) {
    while (nextUnitOfWork !== null) {
      nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
    }
  } else {...}
}

```

## commit 阶段
在 commit 阶段运行的主要函数是  [commitRoot ](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L523) 。它执行如下下操作：
* 在标记为 Snapshot 副作用的节点上调用 getSnapshotBeforeUpdate 生命周期

* 在标记为 Deletion 副作用的节点上调用 componentWillUnmount 生命周期

* 执行所有 DOM 插入、更新、删除操作

* 将 finishedWork 树设置为 current

* 在标记为 Placement 副作用的节点上调用 componentDidMount 生命周期

* 在标记为 Update 副作用的节点上调用 componentDidUpdate 生命周期

在调用变更前方法 getSnapshotBeforeUpdate 之后，React 会在树中提交所有副作用，这会通过两波操作来完成。第一波执行所有 DOM（宿主）插入、更新、删除和 ref 卸载。然后 React 将 finishedWork 树赋值给 FiberRoot，将 workInProgress 树标记为 current 树。这是在提交阶段的第一波之后、第二波之前完成的，因此在 componentWillUnmount 中前一个树仍然是 current，在 componentDidMount/Update 期间已完成工作是 current。在第二波，React 调用所有其他生命周期方法和引用回调。这些方法单独传递执行，从而保证整个树中的所有放置、更新和删除能够被触发执行。

```javascript
function commitRoot(root, finishedWork) {
    commitBeforeMutationLifecycles()
    commitAllHostEffects();
    root.current = finishedWork;
    commitAllLifeCycles();
}

```


### 更新前的生命周期方法
这一副作用意味着会调用 getSnapshotBeforeUpdate 生命周期方法

```javascript
function commitBeforeMutationLifecycles() {
    while (nextEffect !== null) {
        const effectTag = nextEffect.effectTag;
        if (effectTag & Snapshot) {
            const current = nextEffect.alternate;
            commitBeforeMutationLifeCycles(current, nextEffect);
        }
        nextEffect = nextEffect.nextEffect;
    }
}

```

### DOM 更新
```javascript

function commitAllHostEffects() {
    switch (primaryEffectTag) {
        case Placement: {
            commitPlacement(nextEffect);
            ...
        }
        case PlacementAndUpdate: {
            commitPlacement(nextEffect);
            commitWork(current, nextEffect);
            ...
        }
        case Update: {
            commitWork(current, nextEffect);
            ...
        }
        case Deletion: {
            commitDeletion(nextEffect);
            ...
        }
    }
}

```
React 调用 componentWillUnmount 方法作为 commitDeletion 函数中删除过程的一部分。

### 更新后的生命周期方法

 [commitAllLifecycles](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L465)  是 React 调用所有剩余生命周期方法的函数。在 React 的当前实现中，唯一会调用的变更方法就是 componentDidUpdate。


## 参考
* [https://medium.com/react-in-depth/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react-e1c04700ef6e](https://medium.com/react-in-depth/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react-e1c04700ef6e)
* [https://juejin.im/post/5c052f95e51d4523d51c8300#heading-21](https://juejin.im/post/5c052f95e51d4523d51c8300#￼heading-21)