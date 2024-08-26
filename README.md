# TC39 Proposal: Enhanced Try Syntax

## Status

**Stage:** 0  
**Champion:** [Mohsen Azimi](https://x.com/mohsen____)

_For more information see the [TC39 proposal process](https://tc39.github.io/process-document/)._

## Authors

* [Mohsen Azimi](https://x.com/mohsen____)

## Summary

This proposal suggests extending the `try` statement in JavaScript to allow for more flexible error handling patterns, building upon the precedent set by the ["throw expressions" proposal](https://github.com/tc39/proposal-throw-expressions). Specifically, it proposes:

1. Allowing `try` blocks without `catch` or `finally` clauses.
2. Permitting `try` to precede any expression, treating it as a single-line `try` block.

## Motivation

The goal of this proposal is to provide developers with more concise and flexible ways to handle potential errors in their code. By allowing `try` to be used without explicit error handling in some cases, we can simplify code in situations where errors can be safely ignored or where the current syntax feels overly verbose.

This proposal aims to complement the ["throw expressions" proposal](https://github.com/tc39/proposal-throw-expressions) (currently at Stage 2), which allows throwing exceptions in expression contexts. Our enhanced `try` syntax would provide a natural counterpart for handling such throw expressions more flexibly.

## Proposed Syntax

### 1. Try without Catch or Finally

```javascript
try {
  // Code that might throw an error
}
// Execution continues here if an error is thrown
```

### 2. Try with Expression

```javascript
try expression
// Execution continues here if expression throws an error
```

## Semantics

1. In a `try` block without `catch` or `finally`, any error thrown within the block is silently caught and ignored. Execution continues with the next statement after the `try` block.

2. For a `try` expression:

   a. If the expression does not throw an error, the `try` expression evaluates to the result of the expression.

   b. If the expression throws an error, the `try` expression evaluates to `undefined`.

3. In both cases, if no error is thrown, execution proceeds normally.

## Examples

```javascript
// Example 1: Try without catch
try {
  tryInstallingPollyfill('feature1');
  tryInstallingPollyfill('feature2');
}
console.log("This always runs, even if tryInstallingPollyfill() throws");

// Example 2: Try with expression
try fs.unlinkSync('temp.txt');
console.log("Continued execution, regardless of whether the file was deleted successfully");

// Example 3: Combining with throw expressions (from the throw expressions proposal)
function getEncoder(encoding) {
  return try (
    encoding === "utf8" ? new UTF8Encoder() 
    : encoding === "utf16le" ? new UTF16Encoder(false) 
    : encoding === "utf16be" ? new UTF16Encoder(true) 
    : throw new Error("Unsupported encoding")
  );
}

// Example 4: Async code
async function fetchData() {
  try await fetch('https://api.example.com/data')
  console.log("Fetch completed or failed silently");
}

// Example 5: Assignment with fallback
let config = try JSON.parse(userInput) || {};
// If JSON.parse succeeds, config will be the parsed object
// If JSON.parse throws, try expression evaluates to undefined, so config will be {}

// Example 6: Demonstrating undefined result on error
let result = try JSON.parse('{"invalid": json}');
console.log(result); // undefined

// Example 7: Using with nullish coalescing and optional chaining
let data = try fetchData()?.result ?? 'default';
// If fetchData() succeeds, data will be the result or 'default' if result is nullish
// If fetchData() throws, data will be 'default'
```

## Detailed Behavior

1. **Evaluation to `undefined` on Error**: When a `try` expression catches an error, it evaluates to `undefined`. This allows for easy combination with boolean operators (`||` and `&&`) and the nullish coalescing operator (`??`) to provide fallback values.

2. **Seamless Integration with Existing Operators**: The `undefined` result on error allows `try` expressions to work well with optional chaining (`?.`) and nullish coalescing (`??`), providing elegant ways to handle potential errors in expressions.

3. **No Special Interaction with `undefined`**: A `try` expression that evaluates to `undefined` normally (without throwing an error) will still return `undefined`, maintaining consistent behavior.

4. **Short-circuiting**: In expressions like `try a() || b()`, if `a()` throws, `b()` will still be evaluated, as the `try` only wraps `a()`.

## Grammar

```grammarkdown
TryExpression[Yield, Await] :
  `try` Expression[+In, ?Yield, ?Await]

TryStatement[Yield, Await, Return] :
  `try` BlockStatement[?Yield, ?Await, ?Return] Catch[?Yield, ?Await, ?Return]
  `try` BlockStatement[?Yield, ?Await, ?Return] Finally[?Yield, ?Await, ?Return]
  `try` BlockStatement[?Yield, ?Await, ?Return] Catch[?Yield, ?Await, ?Return] Finally[?Yield, ?Await, ?Return]
  `try` BlockStatement[?Yield, ?Await, ?Return]
```

## Alternatives Considered

**Empty catch Blocks**: Requiring empty catch blocks or finally blocks to ignore errors was considered but deemed unnecessarily verbose.

## Benefits

1. **Simplicity**: This feature reduces boilerplate code by eliminating the need for empty catch or finally blocks when the intention is to ignore errors.

2. **Clarity**: It clarifies intent by explicitly indicating that errors are expected and intentionally ignored, rather than relying on empty catch blocks.

3. **Flexibility**: It provides more flexibility in handling errors in scenarios where developers want to test or execute code that might throw an error without interrupting the flow of the program.

## Potential Issues and Considerations

1. **Silent Error Suppression**: This is the point of the proposal. It's a syntactic sugar around an empty catch block, providing a more concise way to ignore errors when that's the desired behavior.

2. **Debugger Challenges**: Some debuggers have the ability to pause the JavaScript execution when an error is thrown even if it's caught (Chrome Dev Tools's `pause on cought exceptions`). Where in the source file this pause should happen is an open question.

3. **Interaction with Existing Try-Catch Semantics**: We need to carefully consider how this interacts with existing `try`-`catch`-`finally` semantics and ensure backwards compatibility.

4. **Precedence and Ambiguity**: Similar to the considerations in the "throw expressions" proposal, we need to carefully define the precedence of `try` expressions to avoid ambiguity.

5. **Indistinguishable `undefined`**: The design doesn't distinguish between an expression that threw an error and one that legitimately evaluated to `undefined`. Users need to be aware of this behavior and handle it appropriately.


## More Examples

These examples illustrate the detailed behavior of the enhanced `try` syntax:

### 1. Basic Usage

```javascript
// Simple try without catch
try {
  riskyOperation();
}
console.log("This always runs");

// Try with expression
let result = try JSON.parse('{"key": "value"}');
console.log(result); // Outputs: {key: "value"}
```

### 2. Error Handling and Undefined Result

```javascript
// When an error occurs, try expression evaluates to undefined
let data = try JSON.parse('{"invalid": json}');
console.log(data); // Outputs: undefined

// Using with nullish coalescing for fallback values
let config = try JSON.parse(userInput) ?? {};
console.log(config); // {} if parsing fails, parsed object otherwise
```

### 3. Interaction with Optional Chaining and Nullish Coalescing

```javascript
function fetchData() {
  // Simulating an API call that might throw
  if (Math.random() < 0.5) throw new Error("Fetch failed");
  return { result: "data" };
}

let data = try fetchData()?.result ?? 'default';
console.log(data); // "data" if successful, "default" if fetchData throws or result is nullish
```

### 4. Short-circuiting Behavior

```javascript
function a() { throw new Error("Error in a"); }
function b() { return "b called"; }

let result = try a() || b();
console.log(result); // Outputs: b called
// Note: b() is called because try only wraps a(), and a() throwing doesn't short-circuit the ||
```

### 5. Async Operations

```javascript
async function asyncOperation() {
  try await fetch('https://api.example.com/data');
  console.log("Fetch completed or failed silently");
}

// Using with async/await
async function processData() {
  let data = try await fetchData();
  console.log(data ?? "Fetch failed or returned null/undefined");
}
```

### 6. Nested Try Expressions

```javascript
function parseAndProcess(input) {
  return try processData(try JSON.parse(input));
}

let result = parseAndProcess('{"key": "value"}');
console.log(result); // Processed data if both parse and process succeed, undefined otherwise
```

### 7. Try with Throw Expressions (assuming Throw Expressions proposal is adopted)

```javascript
function getEncoder(encoding) {
  return try (
    encoding === "utf8" ? new UTF8Encoder() 
    : encoding === "utf16le" ? new UTF16Encoder(false) 
    : encoding === "utf16be" ? new UTF16Encoder(true) 
    : throw new Error("Unsupported encoding")
  );
}

let encoder = getEncoder("ascii");
console.log(encoder); // undefined, as it throws for unsupported encoding
```

These examples demonstrate how the enhanced `try` syntax provides a concise way to handle potential errors in various scenarios, from simple operations to more complex async and conditional logic.

## Backwards Compatibility

This proposal is designed to be fully backwards compatible. Existing `try`-`catch`-`finally` constructs will continue to work as before. The new syntax only adds capabilities and does not change existing behavior.
