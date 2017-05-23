# ReactFiberScheduler.js
`let nextPriorityLevel;`
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
   - perform work now (for `SynchronousPriority`)
   - schedule a callback to perform work later

- continue climbing up the parent path

  `node = node.return;`
