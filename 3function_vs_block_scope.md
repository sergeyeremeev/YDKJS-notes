## Function scope

Example:

```javascript
function foo(a) {
    var b = 2;
    // ...
    function bar() {
        // ...
    }
    // ...
    var c = 3;
}
```
All the identifiers inside of `foo(..)` are accessible inside of `bar(..)` but would result in `ReferenceError` if tried to be accessed from the outer (global in this case) scope.

Function scope encourages that all variables belong to the function and can be used from anywhere inside the function and also accessible to nested scopes. This can lead to some unexpected pitfalls.

#### Hiding in plain scope

Any section of code can be wrapped in a function declaration and all the declarations would be tied to the scope of the new wrapping function, in other words they can be hidden in this function. This can be very useful, especially if following the design principle called **Principle of Least Privilege** (Least Authority/Least Exposure). It states that in the design of software - API or module/object - the absolute minimum should be exposed and everything else should be hidden. In this case we should at least hide as much as possible from the global scope.

Example:

```javascript
function doSomething(a) {
    b = a + doSomethingElse(a * 2);
    console.log(b * 3);
}

function doSomethingElse(a) {
    return a - 1;
}

var b;

doSomething(2); // 15
```

`doSomething` and `doSomethingElse` are most likely the private details of how the program does its job. Their exposure is both, unnecessary and dangerous, as they can be used in wrong way, either by accident or on purpose.

Proper design:

```javascript
function doSomething(a) {
    function doSomethingElse(a) {
        return a - 1;
    }
    
    var b;
    
    b = a + doSomethingElse(a * 2);
    
    console.log(b * 3);
}

doSomething(2); // 15
```

This time only what's necessary is exposed and the functionality has not been affected, which provides for a better design.

#### Collision avoidance

Example:

```javascript
function foo() {
    function bar(a) {
        i = 3; // changes 'i' in the enclosing scope's for-loop
        console.log(a + i);
    }
    
    for (var i = 0; i < 10; i++) {
        bar(i * 2); // infinite loop created
    }
}

foo()
```
The assignment inside `bar(..)` needs to declare a local variable to avoid the infinite loop problem. An **additional** solution would be to use a different identifier name. But using scope to hide the inner declarations is the best option.

This is especially important when talking about global scope. A great example is using a library, which typically introduce a single object into a global scope which serves as a namespace. All the necessary exposure are written as properties of such object.

Finally, there are dependency managers which prevent **any** identifiers being added to the global scope, but require to have those identifiers be explicitly imported into specific scopes where they are required. These tools don't posess any magic but use rules of scoping, so the same effect can be achieved even without them.

### Functions as Scopes

Declaring a function as a wrapper for a chunk of code means that we have to introduce an identifier (function name) and the we have to explicitly call this function. Instead we can use Immediately Invoked Function Expression:

```javascript
var a = 2;

(function foo() {
    var a = 3;
    console.log(a); // 3
})();

console.log(a); // 2
```
Identifier `foo` is bound only inside of its own function expression and does not pollute the enclosing scope.

Note: The easiest way to distinguish between a function expression vs declaration is to look at the position of `function` keyword in the statement. If it's the very first thing - it's a function declaration, if not - it's a function expression. Function expressions can be anonymous, declarations have to have an identifier.

Example:

```javascript
setTimeout(function () {
    console.log('I waited 1 second!');
}, 1000);
```
Anonymous function expressions are faster to type but the have drawbacks:

* there is no useful name in stack traces, making debugging more difficult;

* Self-reference might be useful for unbinding itself as an event handler or recursion calls - the deprecated arguments.callee reference in unfortunately required;

* A descriptive name is often helpful for more readable/understandable code.

The best practice is to always name function expressions.

#### Immediate invocation

The first enclosing pair of parentheses `(..)` make function an expression, the second pair executes the function - pattern is called IIFE. IIFE can also be named to get all the aforementioned benefits.

Two variations of placing the second set of parentheses:

`(function (..) {..}())` and `(function (..) {..})()` are identical in functionality, just a stylistic choice. Arguments can be passed to IIFE, for example:

```javascript
(function IIFE(global) {
    // ...
})(window);
```
This can be useful if we want to make sure that `undefined` identifier value is indeed `undefined` and was not changed. So we can name parameter `undefined` and not pass any value to it to guarantee that the `undefined` identifier is in fact the `undefined` value in a block of code:

```javascript
(function IIFE(undefined) {
    // ...
})();
```

Finally, the argument to the IIFE can be another function, which is used in the UMD (Universal Module Definition) project, which can be a little bit cleaner to understand:

```javascript
(function IIFE(def) {
    def(window);
})(function def(global) {
    // ...
});
```
The `def` function is defined in the second half and the passed as a parameter to the IIFE function defined in the first half. Then the parameter `def` in invoked passing `window` in as the `global` parameter;

### Blocks as scopes

Block scoping makes a lot of sense especially when talking about the Principle of Least Privilege. However, the variables defined inside blocks would still belong to the enclosing function scope:

```javascript
for (var i = 0; i < 10; i++) {
    console.log(i);
}
```
It would make a lot of sense for `i` to be block-scoped and not pollute the enclosing scope. However, this is not the case.

But there are couple of ways to achieve block scoping:

1. `with` - already discussed. Creates a scope from the object which exists for the lifetime of the `with` statement. Bad practice - don't use!!!

2. `try/catch` - since ES3 the `catch` clause in the `try/catch` provides block scoping to the enclosed declarations:

```javascript
try {
    undefined(); // illegal operation
} catch (err) {
    console.log(err);
}
console.log(err); // ReferenceError: 'err' not found
```
In this example `err` exists only in the `catch` clause. This functionality can be very useful (discussed later).

3. `let` - attaches the variable declaration to the scope of whatever block it's contained in. It implicitly hijacks any block scope for its variable declaration:

```javascript 1.7
if (true) {
    let bar = 2;
    console.log(bar); // 2
}
console.log(bar); // ReferenceError
```
To achieve an explicit block scoping we can create an arbitrary block for `let` to create a visual representation of the zone affected by `let`:

```javascript 1.7
if (true) {
    { // block encompassing everything related to 'let'
        let bar = 2;
        console.log(bar);
    }
}
```

Note: `let` declarations will not hoist, so such declarations will not observably "exist" until the declaration statement is reached

```javascript 1.7
{
    console.log(bar); // ReferenceError
    let bar = 3;
}
```

#### Garbage Collection

Example:

```javascript
function process(data) {
    // ...
}

var someBigData = { /* ... */ };

process(someBigData);

document.getElementById('button').addEventListener('click', function click(e) {
    // ...
});
```
Here as soon as `process(..)` runs, we don't need `someBigData` anymore, so it can be garbage collected. However, as the `click` function has a closure over the entire scope, it won't be garbage collected (most likely). An alternative is to write code like this:

```javascript 1.7
function process(data) {
    // ...
}

{
    let someBigData = { /* ... */ };    
    process(someBigData);
}

document.getElementById('button').addEventListener( /* ... */ );
```
Here we explicitly make it clear that the reference to `someBigData` is not needed after the `process(..)` is run. In this example, everything declared inside the block can be garbage collected.

#### Let loops

If we declare a variable with the `let` keyword inside the loop:

```javascript 1.8
for (let i = 0; i < 10; i++) {
    console.log(i);
}
console.log(i); // ReferenceError
```
the `i` would be bound for the loop body **AND** it would be rebound for each iteration of the loop, making sure to reassign the value from the previous loop iteration. In other words:

```javascript 1.8
{
    let i;
    for (i = 0; i < 10; i++) {
        let j = i; // re-bound for each iteration
        console.log(j);
    }
}
```

4. `const` - behaves similar to `let`. The difference is that any attempt to change the value of `const` after it has been declared will result in an error. Also, some value should be assign to `const` as it is declared. This value is fixed (can't be changed *completely*).

Arguably both function and block scoping should be used together where appropriate to product a more readable/maintainable code (both `var` and `let/const` should be used).
