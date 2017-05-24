# React Fiber Implementation Notes

I've been trying to wrap my head around Fiber's source code, so I figured I'd share my notes. Will hopefully organize this better once I get a better idea of how everything fits together.

To follow along, clone React:

`git clone https://github.com/facebook/react.git`

The Fiber source code can be found at `react/src/renderers/shared/fiber`.

ReactDOMFiber can be found at `react/src/renderers/dom/fiber`.

## Sample Step-Through
With the following app code:
```$xslt
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
### ReactDOM.render(\<App />, \<div id="root">...\</div>)
#### Arguments
```$xslt
element: {
    type: function App()
    //...
}
container: <div id="root'>...</div>
callback: undefined
```
#### Actions
```$xslt
renderSubtreeIntoContainer(null, element, container, callback);
```

### renderSubtreeIntoContainer(null, element, container, callback)



## Key Terms

### `alternate`
from [acdlite's React Fiber Architecture](https://github.com/acdlite/react-fiber-architecture#alternate):

> At any time, a component instance has at most two fibers that correspond to it: the current, flushed fiber, and the work-in-progress fiber.

> The alternate of the current fiber is the work-in-progress, and the alternate of the work-in-progress is the current fiber.
