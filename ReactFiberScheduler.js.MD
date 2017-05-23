# ReactFiberScheduler.js

## Table of Contents
- [scheduleUpdate](#ReactFiberScheduler.scheduleUpdate)
- [performWork](#ReactFiberScheduler.performWork)
- [workLoop](#ReactFiberScheduler.workLoop)

<a name="ReactFiberScheduler.scheduleUpdate"></a>
## scheduleUpdate
#### what it looks like
```
scheduleUpdate(fiber: Fiber, priorityLevel: PriorityLevel)
```
#### what it does
- It is possible there is now higher priority work. Check the priority level and reset `ReactFiberScheduler`'s `nextUnitOfWork` if necessary.
```
if(priorityLevel <= nextPriorityLevel) nextUnitOfWork = null;
```

- The rest occurs inside of a loop. From React's source:

  > We walk the parent path to the root. Once we reach a node whose priority (and alternate's priority) matches, we can exit safely knowing that the rest of the path is correct.

  #### `while(node !== null && shouldContinue)`


- Check the `pendingWorkPriority`, and set it equal to `priorityLevel` if the new `priorityLevel` is higher priority.
```
let node = fiber;
if (
    node.pendingWorkPriority === NoWork ||
    node.pendingWorkPriority > priorityLevel
) {
    shouldContinue = true;
    node.pendingWorkPriority = priorityLevel;
}
```
*If `node` has an alternate, update the alternate's priority too.*

- If the node is the root of a host tree:
 - `scheduleRoot(root, priorityLevel);`
 - depending on the `priorityLevel`, either
   - [`performWork`](#ReactFiberScheduler.performWork) now (for `SynchronousPriority`)
   - schedule a callback to perform work later

- continue climbing up the parent path

  `node = node.return;`
  
<a name="ReactFiberScheduler.performWork"></a>
## performWork
#### what it looks like
```
performWork(priorityLevel: PriorityLevel, deadline: Deadline | null)
```
#### what it does
- check to see if there are pending commits from the previous frame

  `if(pendingCommit !== null && !deadlineHasExpired) commitAllWork(pendingCommit)`

- call [`workLoop`](#ReactFiberScheduler.workLoop) - logic extracted to a separate function so that it can be optimized by the JS engine

<a name="ReactFiberScheduler.workLoop"></a>
## workLoop
#### what it looks like
```
workLoop(priorityLevel: PriorityLevel, deadline: Deadline | null)
```

#### what it does
- If our unit of work is HostRoot with children, start the hostRootTimeMarker timer:
```
if (nextUnitOfWork.tag === HostRoot && nextUnitOfWork.child !== null) {
    hostRootTimeMarker = 'React update' + componentName;
    console.time(hostRootTimeMarker)
  }
```
- if we have a deadline, and the priority is not Task work (meaning the priority is not very high), perform work in a loop that checks the deadline on every iteration:
```
if(deadline !== null && priorityLevel > TaskPriority) {
    while(nextUnitOfWork !== null && !deadlineHasExpired) {
      // ...
```
 - If we have time left, call performUnitOfWork.
 - If we've completed a root, a pendingCommit exists. Otherwise, restart the loop.
 - If a pendingCommit exists, check to see if we have enough time to commit it. Either commit the work, or mark the deadline as expired.

```
  if(deadline.timeRemaining() > timeHeuristicForUnitOfWork) {
    //may result in a pendingCommit
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork);

    if(pendingCommit !== null) {
      if(deadline.timeRemaining() > timeHeuristicForUnitOfWork) {
        commitAllWork(pendingCommit);
        nextUnitOfWork = findNextUnitOfWork();
      }
      else deadlineHasExpired = true;
    }

  }

  else deadlineHasExpired = true;
```

- if we do not have a deadline, or if we are performing Task (important) work, use a loop that does not check how much time is remaining. It will keep running until we run out of work at this high priority level (Synchronous and Task priority)
```
if(deadline !== null && priorityLevel > TaskPriority) {
    while(nextUnitOfWork !== null && !deadlineHasExpired) {
      // ...
    }
    // ...
} else {
    while(/* there is more work at this priority level*/) {
      nextUnitOfWork = performUnitOfWork(nextUnitOfWork)

      //if we just committed a root, performUnitOfWork returns null.
      if(nextUnitOfWork === null) nextUnitOfWork = findNextUnitOfWork()
    }
}
```
