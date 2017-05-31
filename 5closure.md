## Scope Closure

Simple definition:

***Closure is when a function is able to remember and access its lexical scope even when being executed outside of its lexical scope.***

Example:

```javascript
function foo() {
    var a = 2;
    
    function bar() {
        console.log(a);
    }
    
    bar();
}

foo();
```
Here we can say that function `bar()` has a closure over the scope of `foo()` (and over the rest of the outer scopes). It's nested inside of `foo()` therefore `bar()` closes over the scope of `foo()`.

The more observable example of closures:

```javascript
function foo() {
    var a = 2;
    function bar() {
        console.log(a);
    }
    return bar;
}
var baz = foo();
baz(); // 2 -> example of closure
```
Here we return the function `bar()` and pass it as a value. We return the function object itself that `bar` references. Then we execute `foo()` and assign the result (returned `bar` function object) to baz and then we invoke `baz()`, which invokes the inner function `bar()` just by a different identifier reference. `bar()` is executed but **outside of its declared lexical scope**.

Usually the function scope goes away after the function is executed, however here the inner scope of `foo()` is kept alive via function `bar()` that closes over the scope of `foo()`. `bar()` still has a reference to that scope and this reference is called *closure*. So when the `baz()` is invoked it has access to author-time lexical scope, even though the function `bar()` is being invoked well-outside of its author-time lexical scope.

Closure lets the function continue to access the lexical scope it was defined it at author-time.

More meaningful example:

```javascript
function wait(message) {
    setTimeout(function timer() {
        console.log(message);
    }, 1000);
}
wait('Hello, closure!');
```
We take an inner function `timer` and pass it to `setTimeout(..)`. The timer has a scope closure over the scope of `wait(..)`, thus keeping the reference to the variable message.

1000ms later, when the scope of `wait(..)` should already be gone, the anonymous function still has closure over that scope.

The built in utility `setTimeout` has reference to some parameter, so when *Engine* invokes that function, which is invoking our `timer(..)` function, the lexical scope reference is still intact.

*Whenever the functions are treated as first-class values and are passed around, they are likely exercising a closure - timers, event handlers, ajax requests, cross-window messaging, web workers, any asynchronous callbacks, etc.*

IIFE is not strictly an example of closure that fits our definition, simply because such function is not executed outside its lexical scope:

```javascript
var a = 2;
(function IIFE() {
    console.log(a)
})();
```
`a` is found via a normal lexical scope look-up, not really via a closure.

Closure might be happening at declaration time, but it's not strictly observable. IIFE is not itself an example of observed closure, but it creates scope and it's one of the most common tools to create a scope which can be closed over.

#### Loops and closure

```javascript
for (var i = 0; i <= 5; i++) {
    setTimeout(function timer() {
        console.log(i);
    }, i * 1000);
}
```
This mistake is so common, that linters advice against using functions inside loops.

If this code is run, we get `6` printed out 5 times which might be unexpected to many developers.

`i` is equal to `6` because this is the termination condition (when `i` is not `5`). So the output is reflecting the final value of the `i` after the loop terminates.

`setTimeout` function callbacks are running way after the completion of the loop. In fact, even if we had `setTimeout(.., 0)` on each iteration, all callbacks would still run after the completion of the loop.

So, what's missing from our code?

We want each iteration to capture its own copy of `i`, however the way scope works, **all 5 function are closed over the same shared global scope** with only one `i` in it. It would be the same if all 5 timeout callbacks were declared one after another with no loop at all. To fix this we need a closured scope for each iteration of the loop:

```javascript
for (var i = 0; i <= 5; i++) {
    (function () {
        var j = i; // now the scope has its own variable which is a copy of i
                   // for each iteration of the loop
        setTimeout(function timer() {
            console.log(j);
        }, j * 1000);
    })();
}
```
A variation using a parameter would be:

```javascript
for (var i = 0; i <= 5; i++) {
    (function (j) {
        setTimeout(function timer() {
            console.log(j);
        }, j * 1000);
    })(i);
}
```
Here we create a new scope for each iteration, which has reference to the current value of `i`, so our timeout callbacks can close over a new scope for each iteration with the correct variable in it.

This can also be solved with `let` keyword:

```javascript 1.7
for (var i = 0; i <= 5; i++) {
    let j = i; //  turn the block scope into scope that can be closed over
    setTimeout(function timer() {
        console.log(j);
    }, j * 1000);
}
```
But we can do even better. If the `let` declaration used inside the head of the loop, the variable will not just be declared once for the loop, but for **each iteration**. It will be initialized at each subsequent iteration with the value from the end of the previous one:

```javascript 1.8
for (let i = 0; i <= 5; i++) {
    setTimeout(function timer() {
        console.log(i);
    }, i * 1000);
}
```

### Modules

**Revealing module pattern** (the most common one):

```javascript
function RevealingModule() {
    var something = 'cool',
        another = [1, 2, 3];
    
    function doSomething() {
        console.log(something);
    }
    
    function doAnother() {
        console.log(another.join(' ! '));
    }
    
    return {
        doSomething: doSomething,
        doAnother: doAnother
    }
}

var foo = RevealingModule();
foo.doSomething(); // 'cool'
foo.doAnother(); // '1 ! 2 ! 3'
```
Here we create a function which has to be invoked in order for the module instance to be created, otherwise the creation of inner scope and closures would not occur. This function returns an object which references the inner functions (but not the variables, which are kept hidden and private). The return value is essentially the public API for our module. We can just as well return a function, which has other functions as its properties (this is what jQuery does).

The inner functions have closure over the inner module scope, when we transport them outside by the way of property references of the object we return, we now have an observable closure.

There are two requirements for the module pattern to work:

* There must be an outer enclosing function and it must be invoked at least once (each invocation creates a module instance);

* This function must return at least one inner function so that it has a closer over the private scope and can access/modify it.

A variation of a module is a singleton, when we only need one instance:

```javascript
var foo = (function RevealingModule() {
    // ...
    return {
        // ...
    }
})();
```
Here we immediately invoke our module function and assign it to a single module instance identifier `foo`.

As modules are mere functions, the can have parameters:
```javascript
function RevealingModule(id) {
    function identify() {
        console.log(id);
    }
    
    return {
        identify: identify
    }
}
var foo1 = RevealingModule('foo1');
foo1.identify(); // 'foo1'
```
Another thing that can be done is to name the object we are returning, so that there is an inner reference to the public API, allowing us to change and modify it from the inside:

```javascript
var foo = (function RevealingModule(id) {
    function change() {
        publicAPI.identify = identify2;
    }
    
    function identify1() {
        console.log(id);
    }
    
    function identify2() {
        console.log(id.toUpperCase());
    }
    
    var publicAPI = {
        identify: identify1,
        change: change
    };
    
    return publicAPI;
})('foo module');

foo.identify(); // 'foo module'
foo.change();
foo.identify(); // 'FOO MODULE'
```

#### ES6 modules

Function-based modules are not statically recognized patterns - their API can be changed at runtime (as we saw above). ES6 modules are static, and compiler knows about it. It checks during compilation (and file loading) that a reference to a member of an imported module's API exists. If there is no reference the compiler will throw an early error at compile time (it will not wait until code execution).

ES6 modules must be defined in separate files (one per module)/ The browsers/engines have a default module loader that synchronously loads a module file when it's imported.

`import` imports one or more members from a module's API into the current scope, each to a bound variable: `import foo from 'foo'`. 

`module` imports an entire module API to a bound variable: `module foo from 'foo`.

`export` exports an identifier (variable or a function) to the public API of the current module.

The contents inside the module file are treated as if enclosed in a scope closure, just like within the function closure modules above.

Example:

bar.js
```javascript 1.8
function hello(who) {
    return 'This is: ' + who;
}

export hello;
```

foo.js
```javascript 1.8
import hello from 'bar';

var hungry = 'hippo';

function awesome() {
    console.log(hello(hungry).toUpperCase());
}
```

baz.js
```javascript 1.8
module foo from 'foo';
module bar from 'bar';

console.log(bar.hello('rhino')); // 'This is: rhino'
foo.awesome(); // 'THIS IS: HIPPO'
```
