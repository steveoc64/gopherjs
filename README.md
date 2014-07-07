GopherJS - A compiler from Go to JavaScript
---------------------------------------------

[![Build Status](https://travis-ci.org/gopherjs/gopherjs.png?branch=master)](https://travis-ci.org/gopherjs/gopherjs)

GopherJS compiles Go code ([golang.org](http://golang.org/)) to pure JavaScript code. Its main purpose is to give you the opportunity to write front-end code in Go which will still run in all browsers. Give GopherJS a try on the [GopherJS Playground](http://gopherjs.github.io/playground/).

You can take advantage of Go's elegant type system and other compile-time checks that can have a huge impact on bug detection and the ability to refactor, especially for big projects. Just think of how often a JavaScript method has extra handling of some legacy parameter scheme, because you don't know exactly if some other code is still calling it in that old way or not. GopherJS will tell you and if it does not complain, you can be sure that this kind of bug is not present any more.

### Design Goals
- performance of generated code (see [HTML5 game engine benchmark](http://ajhager.github.io/enj/) by Joseph Hager)
- similarity between Go code and generated JavaScript code for easier debugging
- compatibility with existing libraries (see the list of [bindings to JavaScript APIs and libraries](https://github.com/gopherjs/gopherjs/wiki/bindings))
- small size of generated code

### What is supported?
Nearly everything, including Goroutines. See the [compatibility table](doc/packages.md) for details.

### Goroutines
JavaScript has no concept of concurrency (except web workers, but those are too strictly separated to be used for goroutines). Because of that, instructions in JavaScript are never blocking. A blocking call would effectively freeze the responsiveness of your web page, so calls with callback arguments are used instead.

GopherJS does some heavy lifting to work around this restriction: Whenever an instruction is blocking (e.g. receiving from an empty channel), the whole stack will unwind (= all functions return) and the goroutine will become asleep. Then another goroutine which is ready to resume gets picked and its stack with all local variables will be restored. This is done by preserving each stack frame inside a closure.

The performance of the code generated by this approach is quite good, but not as good as the simple nonblocking version. That's why GopherJS tries to be conservative about it. It will scan your code for functions that use blocking instructions and mark them as blocking accordingly. Then it will recursively mark all functions as blocking which have a call to some other function which is already know to be blocking. This works well for calls to package functions and for method calls on non-interface types. For calls to interface methods and to `func` values however it is not exactly known at compile time which function it will execute at runtime. In those cases you need to mark the call with the comment `//gopherjs:blocking`. Else, the call will cause panic at runtime.

Also, callbacks from external JavaScript code into Go code (see below) can never be blocking, but you may use the `go` statement to create a new goroutine that may have blocking behavior.

### Installation and Usage
Get or update GopherJS and dependencies with:
```
go get -u github.com/gopherjs/gopherjs
```
Now you can use  `./bin/gopherjs build [files]` or `./bin/gopherjs install [package]` which behave similar to the `go` tool. For `main` packages, these commands create a `.js` file and `.js.map` source map in the current directory or in `$GOPATH/bin`. The generated JavaScript file can be used as usual in a website. Use `./bin/gopherjs help [command]` to get a list of possible command line flags, e.g. for minification and automatically watching for changes. If you want to run the generated code with Node.js, see [this page](doc/syscalls.md).

*Note: GopherJS will try to write compiled object files of the core packages to your $GOROOT/pkg directory. If that fails, it will fall back to $GOPATH/pkg.*

### Getting started
#### 1. Interacting with the DOM
The package `github.com/gopherjs/gopherjs/js` (see [documentation](http://godoc.org/github.com/gopherjs/gopherjs/js)) provides functions for interacting with native JavaScript APIs. For example the line
```js
document.write("Hello world!");
```
would look like this in Go:
```go
js.Global.Get("document").Call("write", "Hello world!")
```
You may also want use the [DOM bindings](http://dominik.honnef.co/go/js/dom), the [jQuery bindings](https://github.com/gopherjs/jquery) (see [TodoMVC Example](https://github.com/gopherjs/todomvc)) or the [AngularJS bindings](https://github.com/gopherjs/go-angularjs). Those are some of the [bindings to JavaScript APIs and libraries](https://github.com/gopherjs/gopherjs/wiki/bindings) by community members.

#### 2. Providing library functions for use in other JavaScript code
Set a global variable to a map that contains the functions:
```go
package main

import "github.com/gopherjs/gopherjs/js"

func main() {
  js.Global.Set("myLibrary", map[string]interface{}{
    "someFunction": someFunction,
  })
}

func someFunction() {
  [...]
}
```
For more details see [Jason Stone's blog post](http://legacytotheedge.blogspot.de/2014/03/gopherjs-go-to-javascript-transpiler.html) about GopherJS.

### Community
- Get help in the [Google Group](https://groups.google.com/d/forum/gopherjs)
- See the list of [bindings to JavaScript APIs and libraries](https://github.com/gopherjs/gopherjs/wiki/bindings) by community members
- Follow [GopherJS on Twitter](https://twitter.com/GopherJS)
