# diff 算法

## Diff 策略
React 在比较新旧 2 棵虚拟 DOM 树的时候，会同时考虑两点：
	* 尽量少的创建 / 删除节点，多使用移动节点的方式
	* 比较次数要尽量少，算法要足够的快

React 选用了启发式的算法，将时间复杂度控制在 O(n) 的级别。这个算法基于以下 2 个假设：
	* 如果 2 个节点的类型不一样，以这 2 个节点为根结点的树会完全不同
	* 对于多次 render 中结构保持不变的节点，开发者会用一个 key 属性标识出来，以便重用
特点
React 遇到不同的节点会删除节点重新创建
React Diff 同级别比较，深度优先遍历

具体比较算法

```javascript
// ReactCompositeComponent shouldUpdateReactComponent.js
function shouldUpdateReactComponent(prevElement, nextElement) {
    var prevEmpty = prevElement === null || prevElement === false;
    var nextEmpty = nextElement === null || nextElement === false;
    if (prevEmpty || nextEmpty) {
        return prevEmpty === nextEmpty;
    }

    var prevType = typeof prevElement;
    var nextType = typeof nextElement;
    if (prevType === 'string' || prevType === 'number') {
        return (nextType === 'string' || nextType === 'number');
    } else {
        return (
            nextType === 'object' &&
            prevElement.type === nextElement.type &&
            prevElement.key === nextElement.key
        );
    }
}
```


```javascript
// ReactMultiChild 的 _updateChildren
_updateChildren: function (nextNestedChildrenElements, transaction, context) {
    …
    
    for (name in nextChildren) {
        if (!nextChildren.hasOwnProperty(name)) {
            continue;
        }
        var prevChild = prevChildren && prevChildren[name];
        var nextChild = nextChildren[name];
        if (prevChild === nextChild) {    ------------------------------------ 1）
            updates = enqueue(
                updates,
                this.moveChild(prevChild, lastPlacedNode,
                    nextIndex, lastIndex)
            );
            lastIndex = Math.max(prevChild._mountIndex,
                lastIndex);
            prevChild._mountIndex = nextIndex;
        } else {                          —————————————————— 2）
            if (prevChild) {
                // Update `lastIndex` before `_mountIndex` gets unset by unmounting.
                lastIndex = Math.max(prevChild._mountIndex,
                    lastIndex);
                // The `removedNodes` loop below will actually remove the child.
            }
            // The child must be instantiated before it's mounted.
            updates = enqueue(
                updates,
                this._mountChildAtIndex(
                    nextChild,
                    mountImages[nextMountIndex],
                    lastPlacedNode,
                    nextIndex,
                    transaction,
                    context
                )
            );
            nextMountIndex++;
        }
        nextIndex++;
        lastPlacedNode = ReactReconciler.getHostNode(nextChild);
    }
    // Remove children that are no longer present.
    for (name in removedNodes) {
        if (removedNodes.hasOwnProperty(name)) {
            updates = enqueue(
                updates,
                this._unmountChild(prevChildren[name],
                    removedNodes[name])
            );
        }
    }
    if (updates) {
        processQueue(this, updates);
    }
    this._renderedChildren = nextChildren;
},
```

## 参考
* [React 源码深度解读（十）：Diff 算法详解 - 有赞美业前端团队 - SegmentFault 思否](https://segmentfault.com/a/1190000017039293)