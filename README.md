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
The app flows like so:
- call [ReactDOM.render()](#ReactDOM.render(App)), which calls
- [ReactDOM.renderSubtreeIntoContainer()](#ReactDOM.renderSubtreeIntoContainer(App)), which calls
- [ReactFiberReconciler.updateContainer(children, newRoot, parentComponent, callback)](#ReactFiberReconciler.updateContainer(App)). 
  - This begins the traversal by calling `scheduleTopLevelUpdate()`.

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
 containerInfo: div#root,
 context: null,
 pendingContext: null,
 current: [object Object],
 //...
}
```
- updateContainer() also accepts `parentComponent` and `callback` arguments, but they are both `null` here.
#### Execution
- call `getContextForSubtree(parentComponent)`. In this case, it returns an empty object. 
- assign `context` to `container.context`.
- call `scheduleTopLevelUpdate(current, element)`
```es6
const context = getContextForSubtree

//container.context === null
if(container.context === null) container.context = context;
else container.pendingContext = context;

scheduleTopLevelUpdate(container.current, element);
```
## Key Terms

### `alternate`
from [acdlite's React Fiber Architecture](https://github.com/acdlite/react-fiber-architecture#alternate):

> At any time, a component instance has at most two fibers that correspond to it: the current, flushed fiber, and the work-in-progress fiber.

> The alternate of the current fiber is the work-in-progress, and the alternate of the work-in-progress is the current fiber.
