# ReactFiberReconciler.js

## Table of Contents
- [createContainer](#ReactFiberReconciler.createContainer)
- [updateContainer](#ReactFiberReconciler.updateContainer)
- [scheduleTopLevelUpdate](#ReactFiberReconciler.scheduleTopLevelUpdate)

<a name="ReactFiberReconciler.createContainer"></a>
## createContainer
```
function createContainer(containerInfo) {
  return ReactFiberRoot.createFiberRoot(containerInfo)
}
```

<a name="ReactFiberReconciler.updateContainer"></a>
## updateContainer
#### what it looks like
```
updateContainer(
  element: ReactNodeList,
  container: OpaqueRoot,
  parentComponent: ?ReactComponent,
  callback: ?Function,
)
```
#### what it does
Retrieves context from `parentComponent` and sets `container.context` or `container.pendingContext`. It then calls [`scheduleTopLevelUpdate`](#ReactFiberReconciler.scheduleTopLevelUpdate) to update the container's fiber.
```
const context = getContextForSubtree(parentComponent);
if(container.context === null) {
  container.context = context;
} else {
  container.pendingContext = context;
}

const current = container.current;
scheduleTopLevelUpdate(current, element, callback);
```

<a name="ReactFiberReconciler.scheduleTopLevelUpdate"></a>
## scheduleTopLevelUpdate
#### what it looks like
```
scheduleTopLevelUpdate(
  current: Fiber,
  element: ReactNodeList,
  callback: ?Function,
)
```
#### what it does
Determines the priority level of this element's pending update, then schedules the update with the determined priority level.

See `[ReactFiberScheduler.scheduleUpdate()](ReactFiberScheduler.js.MD#ReactFiberScheduler.scheduleUpdate)`
```
const forceAsync = //TRUE if element is an async wrapper component. This results in LowPriority.
const priorityLevel = ReactFiberScheduler.getPriorityContext(current, forceAsync)

const nextState = {element};

ReactFiberUpdateQueue.addTopLevelUpdate(current, nextState, callback, priorityLevel);
ReactFiberScheduler.scheduleUpdate(current, priorityLevel);
```
