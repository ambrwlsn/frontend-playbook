JavaScript Style Guide
======================

This document outlines the way we write JavaScript. It's a living styleguide – it will grow as our practices do.

- [General Principles](#general-principles)
  - [Write code for humans](#code-for-humans)
  - [Optimise for reading](#optimise-for-reading)
  - [Write modules over monoliths](#modules-over-monoliths)
  - [KISS](#keep-it-simple)
  - [Performant Code](#performant-code)
- [Code Style](#code-style)
  - [Linting](#linting)
  - [Indentation](#indentation)
  - [White Space](#white-space)
  - [Semi-Colons](#semi-colons)
  - [Variables](#variables)
  - [Functions](#functions)
  - [Classes](#classes)
  - [Operators](#operators)
  - [Blocks](#blocks)
    - [Conditionals](#conditionals)
    - [Loops](#loops)
    - [Error handling](#error-handling)
  - [Strict mode](#strict-mode)
- [Client-Side JavaScript Architecture](#client-side-javascript-architecture)
  - [Directory Structure](#directory-structure)
    - [Components Directory](#components-directory)
    - [Polyfills Directory](#polyfills-directory)
    - [Vendor Directory](#vendor-directory)
    - [Utils Directory](#utils-directory)


General Principles
------------------


### Code For Humans

Your JavaScript should always be optimised for readability first, someone is going to have to maintain your code. Most of the rules in this style guide are here to help enforce this.

Code should be self-documenting wherever possible. This means writing sensible method names and using variables which adequately describe the object in question. Assume no prior knowledge when somebody new starts to contribute to the codebase.

We _don't_ do this:

```js
function load(file, cb) {
    fs.readFile(file, function(e, config) {
        if (e) return cb(e);
        cb(null, process(config));
    });
}
```

We do this:

```js
function loadConfigFile(filePath, callback) {
    fs.readFile(filePath, function(error, fileContents) {
        if (error) {
            return callback(error);
        }
        var config = processConfig(fileContents);
        callback(null, config);
    });
}
```


### Optimise For Reading

Code is written once and read many times, using abbreviations or single-character variables saves time in the short term but there's an overhead every time somebody has to read that code.

Reading code is difficult enough at the best of times. Don't make it harder; it's better to have RSI in your over-worked fingers than for all your colleagues to hate you.


### Modules Over Monoliths

Wherever possible, you should try to think in smaller single-purpose modules and functions. This encourages reuse and helps to [keep complexity down].

If code is generic and reusable you should aim to break it into a separate module which can be managed through npm or included in a project's `vendor` directory. We advocate open sourcing of these kinds of modules, and it's a good idea to have open sourcing in mind no matter what you're writing.

Often you'll find that the code you're writing is catered for by a third-party module. Whenever possible, install a dependency rather than reinventing the wheel.

### Keep it simple

We adhere to the [KISS] principle. Unnecessarily complex/obtuse code completely goes against our rule of coding for humans. It's difficult to understand, slows us down, and reduces maintainability. Don't do it.

It is also Important to remember that [duplication is far cheaper than the wrong abstraction].

```js
// We don't do this
~~number;

// We do this
Math.floor(number);
```

```js
// We don't do this
function isOdd(number) {
    return !!(number & 1);
}

// We do this
function isOdd(number) {
    var mod = number % 2;
    return (mod !== 0);
}
```

```js
// We don't do this
foo ^= bar; bar ^= foo; foo ^= bar;

// We do this
var tempVar = foo;
foo = bar;
bar = tempVar;
```

### Performant Code

We should always try to write fast code, but not if it makes the code difficult to understand. It's inefficient to optimise in favour of performance unless evidence shows that you really _need_ to do that.

If you're writing a small library with a single simple purpose, and you really _really_ need to do it, then it makes sense; in a project with more than one contributor, there's frequently more "cost" than "benefit".


Code Style
---------

### Linting

JavaScript code should be linted with [XO]. XO uses [eslint] under the hood.

If you are using ES2015+ syntax, XO provides [configuration](https://github.com/sindresorhus/xo#esnext) to enforce relevant rules. Add this in your `package.json` to enable it:

```js
"xo": {
  "esnext": true
}
```

Consider the following scenario: Assume the `semicolon` rule is `on`, if you omit a semicolon from your JavaScript code, it is suggested the following actions occur:

1. Your code editor should inform you of this linting failure. There are [XO plugins](https://github.com/sindresorhus/xo#editor-plugins) for popular editors such as [Sublime Text](https://github.com/sindresorhus/SublimeLinter-contrib-xo) and [Atom](https://github.com/sindresorhus/atom-linter-xo).
2. Your local `watch` tasks, e.g. those run through Grunt, Gulp, Make etc. inform you of this error in your terminal. There are [XO plugins](https://github.com/sindresorhus/xo#build-system-plugins) for [Grunt](https://github.com/sindresorhus/grunt-xo), [Gulp](https://github.com/sindresorhus/gulp-xo) and the [CLI](https://github.com/sindresorhus/xo#install).
3. When you push code which fails to lint, your build pipeline should invoke XO as part of a build task. The job should then fail.

#### Automatically fixing code

You can use XO as a command line tool. Once you install it, XO can automatically fix _some_ linting failures. Given a file `file.js` which contains the following:

```js
const object = {
    foo:function(){}
}
```

You can run XO with the `fix` option: `xo --fix file.js`. The same file now contains this:

```js
const object = {
    foo: function () {}
};
```

#### Questions and Answers

Q: XO is complaining about the following code:

```js
adsProvider.load();
```

It says `adsProvider` is not defined, but it's a global variable already present in the page. What should I do?

A: This breaks the [no-undef](http://eslint.org/docs/rules/no-undef) rule of ESLint. You must specify to ESLint that this is an expected global variable. Do not define this project-wide, but rather, be explicit about which file or files need it.

```js
/* global adsProvider */
```

Q: The [default set of rules](https://github.com/sindresorhus/eslint-config-xo/blob/7511eed20ff7e21b9e32b5d240f3bcaca09c0f70/index.js#L16) XO is using is too strict, how should I handle this?

A: [Configure rules](https://github.com/sindresorhus/xo#rules) according to your use case, however, be wary of turning rules off. Strive for consistency.

Q: I just added XO into the build, now everything is failing the linting process.

A: If your codebase has never been linted before, you might find rules being broken in each file. Fixing large amounts of code to pass linting may not be possible to do in one go. As a team, consider adopting an approach to successfully lint each file you touch as part of regular feature development. During this period, alter your build task accordingly to only lint files which you know you have linted. E.g.

```js
gulp.task('lint', () =>
    gulp.src([
      'js/file1.js',
      'js/file2.js'
    ])
    .pipe(xo())
);
```

Q: XO is complaining about undefined variables in my JavaScript unit test code.

A: XO supports many [popular environment global variables](http://eslint.org/docs/user-guide/configuring#specifying-environments). Specify an environment of your choice to add the necessary global variables. Combine this with [Config Overrides](https://github.com/sindresorhus/xo#config-overrides) to ensure you don't apply one or many environments to your entire codebase accidentally.

### Indentation

We indent our JavaScript using single tabs, not spaces. You can convert characters automatically in most editors, and you're advised to do this.

Follow the [BSD-KNF] indentation style.

We do this:

```js
if (foo) {
    console.log(foo);
} else {
    console.log(bar);
}
```

We _don't_ do this:

```js
if (foo)
{
    console.log(foo);
}
else
{
    console.log(bar);
}

if (foo) {
        console.log(foo);
} else {
        console.log(bar);
}
```

### White Space

We use white space liberally to help keep code readable. You're encouraged to use newlines to break up long functions into logical chunks.

You should also remove trailing white space from lines of code. Your editor should be able to either remove this or highlight it when present.

### Semi-Colons

We require the use of semi-colons. The only statements that don't need to end with semi-colons are function declarations and blocks.

We do this:

```js
var foo = 1;

if (foo) {
    foo += 1;
}

function bar() {
    // ...
}

var baz = function() {
    // ...
};
```

We _don't_ do this:

```js
var foo = 1

if (foo) {
    foo += 1
};

function bar() {
    // ...
};

var baz = function() {
    // ...
}
```

### Variables

We define variables with multiple `var` statements and trust our developers to understand [hoisting]. We generally discourage defining variables inside loops or blocks.

We do this:

```js
var foo = 1;
var bar = 2;

var baz;
if (foo === bar) {
    baz = 3;
}
```

We _don't_ do this:

```js
var foo = 1,
    bar = 2;

if (foo === bar) {
    var baz = 3;
}
```

If the environment you're running in supports ES2015 (Node.js 4.x/[Babel](https://babeljs.io/)) then you should not use `var` statements at all – you should be using `let` and `const`:

In ES2015 we do this:

```js
const foo = 'bar';
```

We use `let` only when a variable _explicitly_ [needs to be mutable]:

```js
let foo = 'bar';
```

In ES2015 we _don't_ do this:

```js
var foo = 'bar';
```

You can also use destructuring assignment with objects and arrays. These don't have to be multi-line, but there must be spaces after the commas:

```js
let {foo, bar} = getFooAndBar();
let [baz, qux] = getBazAndQux();
```

### Functions

We use lowerCamelCase naming for functions, and functions should be named descriptively.

Functions are not allowed to be written on a single line – you must always follow our [indentation](#indentation) rules.

Whitespace in functions is different to in blocks: there must be no space between the function declaration and the opening arguments bracket.

We do this:

```js
var loadConfig = function(filePaths, callback) {
    // ...
}

function loadConfig(filePaths, callback) {
    // ...
}
```

We _don't_ do this:

```js
var loadConfig = function (filePaths, callback) {
    // ...
}

function loadConfig(filePaths, callback){ /* ... */ }
```

#### Arguments

Arguments must be separated from the preceding comma by a space, and there should be no padding whitespace at the start and end of the arguments.

We do this:

```js
var loadConfig = function(filePaths, callback) {
    // ...
}
```

We _don't_ do this:

```js
var loadConfig = function(filePaths,callback) {
    // ...
}
```

If the environment you're running in supports ES2015 (Node.js 4.x/[Babel](https://babeljs.io/)) then you can use rest and default arguments.

Default and rest arguments must follow the same spacing rules as other arguments and regular variable assignment.

We do this:

```js
var sayHello = function(person = 'World', ...others) {
    // ...
}
```

We _don't_ do this:

```js
var sayHello = function(person="world",...others) {
    // ...
}
```

### Classes

If the environment you're running in supports ES2015 (Node.js 4.x/[Babel](https://babeljs.io/)) then you can use classes. Classes and their methods must be indented in the same way as functions. Also methods should be spaced apart.

We do this:

```js
class Apple extends Fruit {
    constructor() {
        this.color = 'red';
    }

    peel() {
        // ...
    }
}
```

We _don't_ do this:

```js
class Apple extends Fruit{
    constructor(){ this.color = 'red'; }
    peel () {
        // ...
    }
}
```

### Operators

All operators must have padding whitespace on either side of them. The only exceptions are increment (`++`), decrement (`--`), unary negation (`-foo`), and unary plus (`+foo`).

We disallow the use of normal equal and not equal, requiring you to use the strict equivalents.

We do this:

```js
foo === bar;
foo !== bar;
foo += bar;
baz = 'hello ' + qux;
foo++;
```

We _don't_ do this:

```js
foo == bar;
foo != bar;
foo+=bar;
baz = 'hello '+qux;
foo ++;
```

### Blocks

All blocks must use curly braces and are not allowed to be written on a single line – see the [indentation](#indentation) rules.

Blocks must also have padding whitespace on both sides (unless it's at the start of a line).

If a block requires brackets, there must be padding whitespace between the brackets and the keyword as well as the opening curly brace.

#### Conditionals

In conditionals, `else` blocks must start on the same line as the end curly brace of the `if`.

We do this:

```js
if (foo) {
    foo += 1;
}

if (foo) {
    foo += 1;
} else {
    bar += 1;
}
```

We _don't_ do this:

```js
if(foo){
    foo += 1;
}

if (foo) { foo += 1; }

if (foo) return;

if (foo) {
    foo += 1;
}
else {
    bar += 1;
}
```

#### Loops

In `for` loops, there must be space after the semi-colons.

`for..in` loops must include a check with `hasOwnProperty`.

We do this:

```js
for (var i = 0; i < 10; i += 1) {
    // ...
}

for (var prop in obj) {
    if (obj.hasOwnProperty(prop)) {
        // ...
    }
}
```

We _don't_ do this:

```js
for (var i = 0;i < 10;i += 1) // ...

for (var prop in obj) {
    // ...
}
```

#### Error Handling

`try/catch` blocks must be indented in the same way as conditionals, with `catch` starting on the same line as the end curly brace of the `try`.

We do this:

```js
try {
    foo += 1;
} catch (error) {
    console.log(error);
}
```

We _don't_ do this:

```js
try {
    foo += 1;
}
catch (error) {
    console.log(error);
}

try { foo += 1; }
catch (error) { console.log(error); }
```

### Strict Mode

You should assume that your code will fail, and take steps to handle those failures.

Where [ES5 and above is available and strict mode is implemented](http://kangax.github.io/compat-table/es5/), use strict mode: `'use strict';` This causes JavaScript to error early rather than allow malformed or incorrect code to run.

In Node.js, you can add `'use strict';` to the top of each of your source files.

In-browser you should not add your `'use strict';` to the top of the file; instead you should add it to either the file's [IIFE](http://benalman.com/news/2010/11/immediately-invoked-function-expression/) or each defined function if an IIFE isn't present. This is because global strict mode in browsers can cause issues with third-party code.

Client-Side JavaScript Architecture
-----------------------------------


### Directory Structure

Client-Side JavaScript normally lives in `resources/js` in our projects, unless the back-end you're using dictates a different directory.  We should try to stick as closely to this as possible though.

Within this directory, you should always use the following structure. We'll go into more detail as to what these directories are for shortly:

```
<root>
  ├── components
  ├── polyfills
  ├── utils
  └── vendor
```

File names in these directories should be lower-case, with words separated by dashes.

#### Components Directory

The `components` directory is used to house JavaScript that couples to the DOM. There are several conditions that need to be met for a file to live in here:

  - It must bind to the DOM
  - It must add behaviour to and/or manipulates the DOM
  - It must expose itself on the global `SN.Component` object

An example component might be an auto-complete, tab group, or loading spinner.

#### Polyfills Directory

The `polyfills` directory contains JavaScript that polyfills browser behaviour for older clients. JavaScript in this directory must meet these conditions:

  - It must _not_ override native browser functionality
  - It must match native browser functionality exactly (where possible)
  - It can be either third-party, or written in-house

An example polyfill might add `requestAnimationFrame` for any grade-A browsers which don't support it.

#### Vendor Directory

The `vendor` directory is where third-party code lives. It's also where we add code which is written in-house but managed and versioned elsewhere, e.g. as an open source project.

If the code fits the requirements outlined by the `polyfills` directory, it should go there instead of in the `vendor` directory.

JavaScript in this directory must meet these conditions:

  - It must be maintained outside of the project, e.g. as an open-source library
  - It's file name must contain the version of the code
  - It must include a link to the original source code in a comment
  - It must _not_ be modified within the project

An example vendor library might be jQuery, which would live in a file named `jquery-2.1.4.js`.

#### Utils Directory

The `utils` directory is used to house JavaScript written for the project that doesn't fit into the rules for the `components` directory. Utilities still have some conditions of their own:

  - It must _not_ bind to the DOM
  - It must _not_ add behaviour to and/or manipulate the DOM
  - It must _not_ be destructive – arguments passed in cannot be modified
  - It must expose itself on the global `SN.util` object
  - It can expose functions and/or prototypal classes

An example utility might be a function to make a string title-case.


[KISS]: https://en.wikipedia.org/wiki/KISS_principle
[duplication is far cheaper than the wrong abstraction]: http://www.sandimetz.com/blog/2016/1/20/the-wrong-abstraction
[keep complexity down]: https://en.wikipedia.org/wiki/Cyclomatic_complexity
[eslint]: http://eslint.org/
[jshint]: http://jshint.com/
[hoisting]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/var#var_hoisting
[needs to be mutable]: https://ada.is/blog/2015/07/13/immutable/
[xo]: https://github.com/sindresorhus/xo