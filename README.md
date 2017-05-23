# React Fiber Implementation Notes

I've been trying to wrap my head around Fiber's source code, so I figured I'd share my notes. Will hopefully organize this better once I get a better idea of how everything fits together.

To follow along, clone React:

`git clone https://github.com/facebook/react.git`

The Fiber source code can be found at `react/src/renderers/shared/fiber`.

ReactDOMFiber can be found at `react/src/renderers/dom/fiber`.

## Key Terms

### `alternate`
from [acdlite's React Fiber Architecture](https://github.com/acdlite/react-fiber-architecture#alternate):

> At any time, a component instance has at most two fibers that correspond to it: the current, flushed fiber, and the work-in-progress fiber.

> The alternate of the current fiber is the work-in-progress, and the alternate of the work-in-progress is the current fiber.
