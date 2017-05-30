## Hoisting

Part of the *Compiler's* job during the compilation phase is to find and associate all declarations with their appropriate scopes. So the way to think about things is that all declarations (variables and functions) are processed first before any code is executed.

`var a = 2;` is technically two statements:

* `var a;` - the declaration, processed during compilation phase;

* `a = 2;` - the assignment, left in place for execution phase;

So this code:

```javascript
a = 2;
console.log(a);
var a;
```

is actually processed like:

```javascript
var a;
a = 2;
console.log(a);
```

So metaphorically speaking, the variable and function declarations are "moved" to the top of the code during the compilation phase. This is called *Hoisting*.

Hoisting happens separately inside each scope.

Function declarations are hoisted, but **function expressions are not**:

```javascript
foo();
bar();
var foo = function bar() { /* ... */ }
```
The first line of the code raises a `TypeError`, not a `ReferenceError`, because the variable is declared, but the function expression wasn't assigned to it yet, when we tried calling it as a function, so we attempted to invoke a variable with a value of `undefined` as a function, which is an illegal operation.

The second line of this code raises a `ReferenceError` simply because it's a named function expression, so its identifier is only available from within the function expression, not from the enclosing scope.

#### Functions are hoisted first

Consider:

```javascript
foo();

var foo;

function foo() {
    console.log(1);
}

foo = function () {
    console.log(2);
}
```
The snippet is interpreted by the engine like that:

```javascript
function foo() {
    console.log(1);
}

foo(); // 1

foo = function () {
    console.log(2);
}
```

The *Engine* treats `var foo;` as a duplicate declaration and simply ignores it. Even though it came before the function declaration `function foo(..)`, function declarations are hoisted before normal variables.

Note: multiple `var` declarations are ignored, but subsequent `function` declarations override the previous ones:

```javascript
foo();

function foo() {
    console.log(1);
}

var foo = function () {
    console.log(2);
};

function foo() {
    console.log(3);
}
```

`3` would be logged. This shows that duplicate definitions in the same scope can be a very bad idea.

Note: it's probably best to avoid declaring functions in blocks, because currently they usually hoist to the top of the enclosing scope, but this is likely to be changed in the future:

```javascript
if (true) {
    function foo() {console.log('hi')} // bad idea
}
```
