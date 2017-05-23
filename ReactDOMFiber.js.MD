# ReactDOMFiber.js
`const DOMRenderer = require('ReactFiberReconciler')`

## Table of Contents
- [validateContainer](#validateContainer)
- [renderSubtreeIntoContainer](#renderSubtreeIntoContainer)

<a name="render"></a>
## render
#### what it looks like
```
render(
    element: ReactElement,
    container: DOMContainerElement,
    callback: ?Function,
)
```
#### what it does
- Validates the container with [`validateContainer`](#validateContainer)
- Returns the result of [`renderSubtreeIntoContainer`](#renderSubtreeIntoContainer)

<a name="validateContainer"></a>
## validateContainer
#### what it looks like
```
validateContainer(container)
```
#### what it does
Checks to see if the target container is a DOM element and throws an Error if it is not:
```
function validateContainer(container) {
  if (!isValidContainer(container)) {
    throw new Error('Target container is not a DOM element.');
  }
}

function isValidContainer(node) {
  return !!(node &&
    (node.nodeType === ELEMENT_NODE ||
      node.nodeType === DOCUMENT_NODE ||
      node.nodeType === DOCUMENT_FRAGMENT_NODE));
}
```

<a name="renderSubtreeIntoContainer"></a>
## renderSubtreeIntoContainer
#### what it looks like
```
renderSubtreeIntoContainer(
  parentComponent: ReactComponent,
  children: ReactNodeList,
  containerNode; DOMContainerElement | Document,
  callback: ?Function
)
```
#### what it does
- If there is no root inside of the container, it will:
```
//clear existing content inside container
while(container.lastChild) container.removeChild(container.lastChild);
//create new root and attach to DOMContainer
const newRoot = DOMRenderer.createContainer(container);
const root = container._reactRootContainer = newRoot
```
- It will then call [DOMRenderer.updateContainer()](ReactFiberReconciler.js.MD#updateContainer).

  *Note that DOMRenderer = require(ReactFiberReconciler.js)*
```
DOMRenderer.updateContainer(children, root, parentComponent, callback);
```
- Finally, we return [DOMRenderer](ReactFiberReconciler.js.MD).getPublicRootInstance(root)
