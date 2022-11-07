# Code comments

## Changelog

[https://keepachangelog.com/en/1.0.0/](https://keepachangelog.com/en/1.0.0/)

- ****Keep Team Members on the Same Page****
- ****Re-learning From the Past****

### Types of changes

- `Added` for new features.
- `Changed` for changes in existing functionality.
- `Deprecated` for soon-to-be removed features.
- `Removed` for now removed features.
- `Fixed` for any bug fixes.
- `Security` in case of vulnerabilities.

## Rules

[https://stackoverflow.blog/2021/12/23/best-practices-for-writing-code-comments/](https://stackoverflow.blog/2021/12/23/best-practices-for-writing-code-comments/)

### ****Rule 2: Good comments do not excuse unclear code****

```java
private static Node getBestChildNode(Node node) {
    Node n; // best child node candidate
    for (Node node: node.getChildren()) {
        // update n if the current state is better
        if (n == null || utility(node) > utility(n)) {
            n = node;
        }
    }
    return n;
}
```

The need for comments could be eliminated with better variable naming:

```java
private static Node getBestChildNode(Node node) {
    Node bestNode;
    for (Node currentNode: node.getChildren()) {
        if (bestNode == null || utility(currentNode) > utility(bestNode)) {
            bestNode = currentNode;
        }
    }
    return bestNode;
}
```

### ****Rule 8: Add comments when fixing bugs****

Comments should be added not just when initially writing code but also when modifying it, especially fixing bugs. Consider this comment:

```java
  // NOTE: At least in Firefox 2, if the user drags outside of the browser window,
  // mouse-move (and even mouse-down) events will not be received until
  // the user drags back inside the window. A workaround for this issue
  // exists in the implementation for onMouseLeave().
  @Override
  public void onMouseMove(Widget sender, int x, int y) { .. }
```

### **Rule 9: Use comments to mark incomplete implementations**

Sometimes it’s necessary to check in code even though it has known limitations. While it can be tempting not to share known deficiencies in one’s code, it is better to make these explicit, such as with a TODO comment:

```java
// TODO(hal): We are making the decimal separator be a period,
// regardless of the locale of the phone. We need to think about
// how to allow comma as decimal separator, which will require
// updating number parsing and other places that transform numbers
// to strings, such as FormatAsDecimal
```

## Python

[https://www.stepsize.com/blog/the-engineers-guide-to-writing-code-comments](https://www.stepsize.com/blog/the-engineers-guide-to-writing-code-comments)

For functions, you should include the following code tags:

- @desc - Write down a description for your function
- @param - Describe all input parameters the function accepts. Make sure to define the input types.
- @returns - Describe the returned output. Make sure to define the output type.
- @throws - Describe the error type the function can throw
- @example - Include one or multiple examples that show the input and expected output

```python
/**
 * Creates an array of elements split into groups the length of `size`.
 * If `array` can't be split evenly, the final chunk will be the remaining
 * elements.
 *
 * @since 3.0.0
 * @category Array
 * @param {Array} array The array to process.
 * @param {number} [size=1] The length of each chunk
 * @returns {Array} Returns the new array of chunks.
 * @example
 *
 * chunk(['a', 'b', 'c', 'd'], 2)
 * // => [['a', 'b'], ['c', 'd']]
 *
 * chunk(['a', 'b', 'c', 'd'], 3)
 * // => [['a', 'b', 'c'], ['d']]
 */
function chunk(array, size = 1) {
  // logic
}
```

## Github is your friend 🐻

[https://github.com/angular/angular/releases](https://github.com/angular/angular/releases)

[https://github.com/kubernetes/kubernetes/blob/master/pkg/controller/deployment/progress.go](https://github.com/kubernetes/kubernetes/blob/master/pkg/controller/deployment/progress.go)

Many more…
