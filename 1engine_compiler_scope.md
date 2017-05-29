## JavaScript is also a compiled language

Javascript is dynamic/interpreted language, but it's also a compiled one. It's not compiled well in advance like most other languages, but JS engine still performs many of the same steps:

1. **Tokenizing/Lexing**: breaking up a string of characters into meaningful chunks called tokens. For example:

```javascript
var a = 2;
```
would be broken into `var, a, =, 2, ;` and possibly whitespace tokens if the whitespace is meaningful. **Lexing** is when a tokenizer invokes stateful parsing rules to determine whether a chunk is a distinct token or a part of another token.

2. **Parsing**: taking a stream of tokens and turning it into a tree of nested elements that collectively represent the grammatical structure of the program - Abstract Syntax Tree (AST). For example:

```javascript
var a = 2;
```
tree might start with `VariableDeclaration`, with a child node `Identifier` (`a`), and another child `AssignmentLiteral`, which has a child `NumericLiteral` with value `2`.

3. **Code-Generation**: Taking AST and turning it into executable code. This can vary greatly depending on the language, platform, etc. 

These form a very simplified view on how JS engine works. JS engine also performs optimizations for the performance of execution during the parsing and code-generation steps (like removing redundant variables).

JS compilation usually happens milliseconds before the code is executed, so there isn't much time for optimizations. JS engine has to use any tricks possible to ensure the best performance optimizations. But it's important that compilation actually does happen, just before executing the code.

#### Three important characters of program execution

1. **Engine**: responsible for start-to-finish compilation and execution of JS program

2. **Compiler**: handles parsing and code-generation work

3. **Scope**: collects and maintains a look-up list of all the declared identifiers (variables) and enforces a strict set of rules as to how these are accessible to currently executing code

#### How the statement is approached

```javascript
var a = 2;
```
1. First *Compiler* performs lexing and breaks it down into tokens. Then *Compiler* gets to code generation and performs it in a slightly unexpected way:

* `var a` is encountered and *Scope* is asked whether a variable `a` already exists for that particular scope collection. If yes, *Compiler* ignores it and moves on. If not, *Compiler* asks the *Scope* to declare a new variable called `a` for that scope collection.

* *Compiler* produces the code for *Engine* to later execute to handle the `a = 2` assignment. As the *Engine* runs this code it will first ask the *Scope* to check if a variable called `a` exists in the current scope collection. If yes, it will use this variable. If not, *Engine* will look for the variable elsewhere. If it eventually finds one - the value `2` is assigned to it. If not, *Engine* will yell out an error.


Before the *Engine* executes the code produced by *Compiler* it looks up the variable consulting the scope. There are 2 types of look-ups: Left-hand Side and Right-hand Side (*RHS* and *LHS*). These are sides of an assignment operation.

An RHS look-up is indistinguishable for our purposes from the look-up of a value of some variable. LHS look-up is trying to find the variable container itself, so that it can assign to it. Due to this we mostly care about the LHS whereas RHS can be treated as simply non-LHS. In other words RHS can be thought of as "go get the value of..."

Example:

```javascript
console.log(a);
```
the reference to `a` is a RHS reference, because nothing is being assigned to `a` here, but we are trying to retrieve the value of `a`.
 
 By contrast:
 
 ```javascript
 a = 2;
 ```
reference to `a` is an LHS reference as we don't care what the current value is, instead we need to get the variable to use as a target for the `= 2` assignment.

It's worth noting that LHS and RHS are not necessarily left/right hand sides of the `=` operator, as there are other ways the assignment could happen. It's better to think of it as the *target of the assignment* (LHS) and the *source of the assignment* (RHS).

Example:
```javascript
function foo(a) {
    console.log(a); // 2
}

foo(2);
```

last line is a function call `foo(..)` that asks to look up the value of `foo`, therefore it's an RHS reference. `(..)` means that the value of `foo` should be executed.

There is also a `a = 2;` hidden in this example. It happens when the value `2` is passed as an argument to the function `foo(..)`, in other the value `2` is assigned to the parameter `a`. To implicitly assign to parameter `a` an LHS look-up is performed.

Finally, there is also an RHS reference for the value of `a` and the resulting value is passed to the `console.log(..)` statement, which needs a reference to execute. It's an RHS lookup for the `console` object after which a property-resolution occurs to check if a method called `log` exists. Inside there is some sort of exchange of LHS/RHS of passing the value `2` (by RHS look-up of variable `a`) into `log(..)`. The `log(..)` presumably has parameters the first of which has a LHS reference look-up before assigning `2` to it.

Important:

```javascript
function foo(a) {...}
```
is not really the same as

```javascript
var foo;
foo = function (a) {...}
```
because *Compiler* handles both the declaration and the value definition during the code-generation phase, such that when *Engine* is executing code, there is no processing necessary to "assign" a function value to foo. So it's inappropriate to think of a function declaration as an LHS look-up assignment. 

####Summary of the above as an *Engine/Scope* conversation:
E: RHS reference for foo. Have it?

S: Sure, Compiler declared it as a function.

E: Executing `foo`. LHS reference for `a`. Have it?

S: Sure, Compiler declared it as a formal parameter to foo.

E: Assignine `2` to `a`. RHS reference for `console`. Have it?

S: Sure, it's a built in object.

E: Looking up `log(..)`. It's a function. RHS reference for `a`. Have it?

S: Still do, it hasn't been changed.

E: Passing the value of `a`, which is `2` into `log(..)`.


#### Nested Scopes

Usually there is more than 1 scope. Scopes are nested inside each other just like the blocks of functions are. If a variable cannot be found inside the immediate scope, *Engine* consults next outer containing scope, and the next until the outermost - global - scope is reached.

Example:

```javascript
function foo(a) {
    console.log(a + b);
}

var b = 2;

foo(2);
```

Although the RHS reference for `b` can't be resolved inside the function `foo`, it can be found in the scope surrounding it.

LHS and RHS look-ups behave differently in a situation in which the variable has not been yet declared:

```javascript
function foo(a) {
    console.log(a + b);
    b = a;
}

foo(2)
```

The first RHS lookup for `b` will not be resolved. Such variable is called `undeclared`, as it's not found in the scope. Failed RHS look-up results in a `ReferenceError` being thrown by the *Engine*.

If an LHS lookup fails and the global scoped is reached, the global scope will create a new variable of that name in the global scope and hand it back to the *Engine* (if not running the program in a `strict mode`). Strict mode disallows the implicit global variable creation and in such case an *Engine* would throw the `ReferenceError` just like with the RHS case. If you try to do something with the RHS reference, which is illegal (trying to execute as a function a non-function value) the *Engine* would throw a `TypeError`.

`ReferenceError` is scope resolution-failure related, whereas `TypeError` means that the scope resolution was successful, but there was an illegal/impossible action attempt.
