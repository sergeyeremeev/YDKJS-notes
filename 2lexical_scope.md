## Lexical scope

There are 2 predominant scope models:

* Dynamic scope (Bash scripting);

* Lexical scope;

The lexing process examines a string of source code characters and assigns semantic meaning to the tokens as a result of some stateful parsing.

It is a scope that is defined at lexing time, in other words it is based on where variables and blocks of scope are authored at write time (and therefore set in stone).

Example:

```javascript
function foo(a) {
    
    var b = a * 2;
    
    function bar(c) {
        console.log(a, b, c);
    }
    
    bar(b * 3);
}

foo(2);
```
There are three scopes in this code:

* Global scope - containing `foo` identifier in it;

* Scope of `foo` - containing `a, b` and `bar` identifiers;

* Scope of `bar` - containing `c` identifier;

Scopes are always fully nested inside each other, no partial nesting is possible (no Venn-like diagrams).

The structure and placement of these scope bubbles explain it to the *Engine* where it should look for a certain identifier.

Scope look-up stops once the variable is found - if the variable is already found, the outer scopes will not be checked. This can be used for **Shadowing** the outer variables by the inner ones with the same identifier.

Note: global variables can be accessed like properties of the global object, ie `window.a`, which allows to access them even if they are shadowed. No other shadowed variables can be accessed.

Function's lexical scope is defined by **where it was declared** no matter how it's invoked or where it's invoked.

Lexical scope look-up process only applies to first-class identifiers, ie if we have a reference of `foo.bar.baz` look-up would only be applied to `foo` identifier. Once it's located the object property-access rules take over to resolve its properties.


### Cheating lexical scope

Cheating lexical scope leads to poorer performance!!!

Two ways to cheat lexical scope: `eval` and `with`.

#### `eval`

`eval(..)` function takes a string as an argument and treats it like an authored code. You can generate code and run it as if it had been there at author time. On subsequent lines after `eval` the *Engine* will not know (or care) if the previous code was dynamically interpreted and modified the lexical scope environment.

Example:

```javascript
function foo(str, a) {
    eval(str);
    console.log(a, b);
}

var b = 2;

foo('var b = 3;', 1);
```
The string `'var b = 3;'` is treated at the point of the `eval(..)` call, as if it was there all the time. The code declares a new variable `b` and modifies the existing lexical scope of `foo(..)`, shadows the `b` which is declared in the global scope, and we get `4` printed instead of `3`.

Note: in `strict mode` `eval` operates in its own lexical scope - declarations made inside don't modify the enclosing scope.

`setTimeout` and `setInterval` can take a string as their first argument the contents of which are evaluated as the code of a dynamically generated function. This behavior is long deprecated and should not be used!!!

`new Function(..)` similarly takes a string as the last argument and turns it into a dynamically generated function. It is safer than `eval` but still should not be used.

All of these have negative effects on performance, so it's almost never reasonable to use any of those.

#### `with`

This is now deprecated feature. It is typically used as a shorthand for making multiple property references against an object without specifying the object reference each time:

```javascript
var obj = {
    a: 1,
    b: 2,
    c: 3
};

with (obj) {
    a = 3;
    b = 4;
    c = 5;
}
```

In fact there is more going on here:

```javascript
function foo(obj) {
    with (obj) {
        a = 2;
    }
}

var o1 = { a: 3 };
var o2 = { b: 4 };

foo(o1);
console.log(o1.a); // 2

foo(o2);
console.log(o2.a); // undefined
console.log(a); // 2 -> leaked global variable
```
Inside `with` block we see a LHS reference to `a` and a value of `2` assigned to it. However, when we pass `o2` to our function, which doesn't have `a` property, no such property is created. `with` takes an object and treats it as if it was a wholly separate lexical scope, thus the object properties are treated as lexically defined identifiers. So the LHS look-up moves up a scope, reaches global scope which creates variable with such identifier in it (in non-strict mode).

Although an object is treated like a lexical scope by `with`, the `var` declaration inside will be scoped to the containing function scope.

#### Conclusion

`eval` can modify the existing lexical scope at runtime - if there are any declarations in it. `with` creates a whole new lexical scope at runtime. In `strict mode` `with` is completely disallowed, and various forms of indirect or unsafe `eval` are disallowed while retaining the core functionality.

#### Performance

There are quite a few performance optimizations performed by the JS engine during the compilation phase. Some of these are linked to predetermining where all the variables and function declarations are during statically analyzing the code as it lexes, so the identifiers can be resolved faster during the code execution.

Seeing either `with` or `eval` in the code, JS engine has to assume that anything in the lexical scope might be invalidated at runtime, so most optimizations are not performed **at all**.

Simply including `eval` or `with` in the code, will make it run slower!!!
