# Gambit Scheme JavaScript Notes

[Gambit Scheme](https://gambitscheme.org) has in my opinion pretty amazing mechanism for embedding C code into Scheme projects. Gambit is also able to compile to JavaScript and it provides tools for combining Scheme and JavaScript code. Documentation for using C (and thus interoperability with a host environment) is documented pretty well, but resources for JavaScript are a bit sparse. This is a shame, because Gambit is a very well implemented Scheme and in my opinion has lots of potential for developing browser-based applications. The reason for the state of the documenation seems to be planned refactoring of all the FFI backend stuff.

Looking at examples, there seems to be at least two different ways of using JavaScript. This document tries to describe one of the ways as I have understood things. So, there might be mistakes. Read and use at your own risk.

## Tools available for JavaScript interoperability

First we have to inform the compiler that the following functions are available in the runtime system even though they cannot be found during the compilation stage.

```
(declare (extended-bindings ##inline-host-statement
                            ##inline-host-expression))
```

Calling any JavaScript code from Scheme is done using ##inline-host-* primitives.

- ##inline-host-statement is used when we want to call JavaScript code and we are not interested in of any return values.
- ##inline-host-expression is used when we want to receive a return value.

When we want to pass or receive objects between Scheme code and JavaScript code, we have to use the following statements:

- @scm2host@() - converts Scheme simple type value to JavaScript simple type value
- @host2scm@() - converts JavaScript simple type value to Scheme simple type value
- @host2foreign@() - passes through a JavaScript object as a foreign object that can be stored in Scheme
- @foreign2host@() - passes through a foreign object back to JavaScript, note that it's also OK to use @scm2host@, which is able to handle foreign objects correctly

To inject arguments to the JavaScript code, use @1@, @2@, @3@, ... to refer to the arguments - in order - passed to ##inline-host-* primitives.

In short, the syntax for ##inline-host-statement (and -expression) is: `(##inline-host-statement "javascript code" args ...)`

The whole JavaScript code snippet is enclosed in the string. Inside the string the above data conversion statemants and references to arguments can be used.

## Example project

### `Makefile`

```
test.js: test.scm
	~/Gambit/bin/gsc -warnings -target js -o $@ -exe -nopreload $<
```

### `index.html`

Loading the compiler-generated JavaScript is easy. Just refer to the file in a script tag:

```
<!DOCTYPE HTML>
<html>
  <body>
    <script src="test.js"></script>
  </body>
</html>
```

### `test.scm`

This example prints output to the web browser developer console and calls alert() function at the end:

```
(declare (extended-bindings ##inline-host-statement
                            ##inline-host-expression))
(define c (##inline-host-expression "@host2foreign@(class Test{static a = 1234;})"))
(display c) ; -> #<foreign #2 0x0>
(newline)
(define b (##inline-host-expression "@host2scm@(@foreign2host@(@1@).a)" c))
(display b) ; -> 1234
(newline)
(##inline-host-statement "alert(@scm2host@(@1@));" b)
```

## Sources

- Gambit Scheme source code and code examples in the repository: https://github.com/gambit/gambit
- https://mailman.iro.umontreal.ca/pipermail/gambit-list/2022-January/009608.html
- http://www.iro.umontreal.ca/~feeley/papers/BelangerFeeleyELS21.pdf

