# React Fiber Implementation Notes

I've been trying to wrap my head around Fiber's source code, so I figured I'd share my notes. Will hopefully organize this better once I get a better idea of how everything fits together.

To follow along, clone React:

`git clone https://github.com/facebook/react.git`

The Fiber source code can be found at `react/src/renderers/shared/fiber`.

ReactDOMFiber can be found at `react/src/renderers/dom/fiber`.

# Sample Step-Through
With the following app code:
```es6
const TextBox = ({ onChange }) => (
  <input onChange={ onChange }></input>
);

const Display = ({ text }) => (
  <div>{ text }</div>
)

class App extends Component {
  constructor() {
    super();
    this.state = {
      text: 'type',
    };
  };

  changeText = (event) => {
    const value = event.target.value;
    this.setState((state, props) => ({
      text: value
    }));
  };

  render() {
    return (
      <div>
        <Display text={ this.state.text }/>
        <TextBox onChange = { this.changeText } />
      </div>
    )
  }
}

ReactDOM.render(<App />, document.getElementById('root'))
```
Trace the initial mounting of the app:
- call [`ReactDOM.render()`](#ReactDOM.render(App)), which calls
- [`ReactDOM.renderSubtreeIntoContainer()`](#ReactDOM.renderSubtreeIntoContainer(App)), which calls
- [`ReactFiberReconciler.updateContainer()`](#ReactFiberReconciler.updateContainer(App)). 
  - This begins the traversal by calling [`scheduleTopLevelUpdate()`](#ReactFiberReconciler.scheduleTopLevelUpdate(App)).
  
  
- [`scheduleTopLevelUpdate()`](#ReactFiberReconciler.scheduleTopLevelUpdate(App)) kicks off the excitement with the following calls:
  - [`ReactFiberUpdateQueue.addTopLevelUpdate()`](#ReactFiberUpdateQueue.addTopLevelUpdate(App))
    - [`insertUpdate()`](#ReactFiberUpdateQueue.insertUpdate(App))

<a name="ReactDOM.render(App)"></a>
## ReactDOM.render(\<App />, \<div id="root">...\</div>)
#### Arguments
- element:
```es6
{
    type: function App()
    //...
}
```
- container: 
```es6
<div id="root">...</div>
```

- callback: 
```es6
undefined
```

#### Execution
```es6
renderSubtreeIntoContainer(null, element, container, callback);
```

<a name="ReactDOM.renderSubtreeIntoContainer(App)"></a>
## ReactDOM.renderSubtreeIntoContainer(null, element, container, callback)
#### Arguments
- parentComponent:
```es6
null
```
- children: 
```es6
//this is the element retruned from <App />'s render().
{
 type: function App(),
 //...
}
```
- containerNode:
```es6
<div id="root"></div>
```
- callback: 
```es6
undefined
```

#### Execution
First, set `container: DOMContainerElement` based on `containerNode.nodeType`.  
This is a safety check - if `containerNode` is a `DOCUMENT_NODE`, we get the [root element of the document](https://developer.mozilla.org/en-US/docs/Web/API/Document/documentElement).

Our container, `<div id="root" />` is an `ELEMENT_NODE`, so we just assign `container = containerNode`.

```es6
let container: DOMContainerElement = containerNode.nodeType === DOCUMENT_NODE
    ? (containerNode: any).documentElement
    : (containerNode: any);
```
Then set `root` equal to `container._reactRootContainer`. In our case, `root === undefined`, so we enter the following conditional block. 

In the block, we prepare our `root` for mounting in two steps:
1. clear existing content from the container.
2. Create a container with `DOMRenderer.createContainer(container)`

Then, call `DOMRenderer.updateContainer()` with our `root`. Note that we do *not* batch the update if this is our initial mount, indicated by a nonexistent `container._reactRootContainer`.

This `DOMRenderer.updateContainer()` call will reconcile and mount the whole tree (!!!)
```es6
let root = container._reactRootContainer;

if (!root) {
    // First clear any existing content.
    while (container.lastChild) {
      container.removeChild(container.lastChild);
    }
    
    const newRoot = DOMRenderer.createContainer(container);
    root = container._reactRootContainer = newRoot;
    
    // Initial mount should not be batched.
    DOMRenderer.unbatchedUpdates(() => {
      DOMRenderer.updateContainer(children, newRoot, parentComponent, callback);
    });
} else {
    DOMRenderer.updateContainer(children, root, parentComponent, callback);
}
```

<a name="ReactFiberReconciler.updateContainer(App)"></a>
## ReactFiberReconciler.updateContainer(children, root)
#### Arguments
- element:
```es6
//this is the element retruned from <App />'s render().
{
 type: function App(),
 //...
}
```
- container:
```es6
{
 context: null,
 pendingContext: null,
 current: [object Object],
 //...
}
```
- *updateContainer() also accepts `parentComponent` and `callback` arguments, but they are both `null` here.*
#### Execution
- call `getContextForSubtree(parentComponent)`. In this case, it returns an empty object. 
- assign `context` to `container.context`.
- call `scheduleTopLevelUpdate(current, element)`
```es6
//parentComponent is null here. getContextForSubtree() returns an empty object.
const context = getContextForSubtree(parentComponent)

//container.context === null
if(container.context === null) container.context = context;
else container.pendingContext = context;

//container.current is the container's current Fiber.
scheduleTopLevelUpdate(container.current, element);
```

<a name="ReactFiberReconciler.scheduleTopLevelUpdate(App)"></a>
## ReactFiberReconciler.scheduleTopLevelUpdate() for \<App />
#### Arguments
- current
```es6
//<App />'s fiber. This is passed to the next call - we do not use any of its properties here.
{
 //...
}
```
- element
```es6
//this is the element retruned from <App />'s render().
{
 type: function App(),
 //...
}
```
- *`scheduleTopLevelUpdate()` also accepts a `callback` argument, but it is null here.*
#### Execution
- Retrieve the fiber's priority level for scheduling.
  - In the real source, this also checks if the element is an async wrapper component. If so, it will return `LowPriority`.
- Call `ReactFiberUpdateQueue.addTopLevelUpdate()`
- Call `ReactFiberScheduler.scheduleUpdate()`
```es6
const priorityLevel = getPriorityContext(current);
const nextState = { element };

addTopLevelUpdate(current, nextState, callback, priorityLevel);
scheduleUpdate(current, priorityLevel);
```


<a name="ReactFiberUpdateQueue.addTopLevelUpdate(App)"></a>
## ReactFiberUpdateQueue.addTopLevelUpdate() for \<App />
#### Arguments
- fiber
```es6

```
- partialState
```es6
{
 //this is the element retruned from <App />'s render().
 element: {
  type: function App(),
  //...
 }
}
```
- priorityLevel
```es6
1
```
- *`addTopLevelUpdate()` also accepts a `callback` argument, but it is `null` here.*
#### Execution
- Generate an `update` object and call `insertUpdate`:
```es6
const update = {
    priorityLevel,
    partialState,
    callback,
    isReplace: false,
    isForced: false,
    isTopLevelUnmount,
    next: null,
  };
  
  insertUpdate(fiber, update);
```


<a name="ReactFiberUpdateQueue.insertUpdate(App)"></a>
## ReactFiberUpdateQueue.insertUpdate() for \<App />
#### Arguments
#### Execution