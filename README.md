go-xslate
=========

Attempt to port Perl5's Text::Xslate to Go

[![Build Status](https://travis-ci.org/lestrrat/go-xslate.png?branch=master)](https://travis-ci.org/lestrrat/go-xslate)

Description
===========

This is an attempt to port [Text::Xslate](https://github.com/xslate/p5-Text-Xslate) from Perl5 to Go.
HOWEVER, although the author has 10+ yrs experience programming, he has absolutely no experience developing virtual machines, compilers, et al. Your help is much, much, much needed (note: "appreciated" is an understatement. it's "needed")

Current Status
=======

Currently:

* I'm aiming for port of most of TTerse syntax
* I'm working on the Virtual Machine portion
* VM currently supports: print\_raw, variable subtitution, arithmetic (add, subtract, multiply, divide), if/else conditionals, "for x in list" loop, simple method calls, function calls
* VM TODO: loops, macros, stuff involving external templates
* Parser is currently not finished.


Caveats
=======

Functions
---------

In Go, functions that are not part of current package namespace must be
qualified with a package name, e.g.:

    time.Now()

This works fine because you can specify this at compile time, but you can't
resolve this at runtime... which is a problem for templates. The way to solve
this is to register these functions as variables:

    template = `
      [% now() %]
    `
    tx.RenderString(template, xslate.Vars { "now": time.Now })

But this forces you to register these functions every time, as well as
having to take the extra care to make names globally unique.

    tx := xslate.New(
      functions: map[string]FuncDepot {
        // TODO: create pre-built "bundle" of these FuncDepot's
        "time": FuncDepot { "Now": time.Now }
      }
    )
    template := `
      [% time.Now() %]
    `
    tx.RenderString(template, ...)


Comparison Operators
--------------------

The original xslate, written for Perl5, has comparison operators for both
numeric and string ("eq" vs "==", "ne" vs "!=", etc). In go-xslate, there's
no distinction. Both are translated to the same opcode (XXX "we plan to", that is)

So these are the same:

    [% IF x == 1 %]...[% END %]
    [% IF x eq 1 %]...[% END %]


Accessing Fields
----------------

Only public struc fields are accessible from templates. This is a limitation of the Go language itself.
However, in order to allow smooth(er) migration from p5-Text-Xslate to go-xslate, go-xslate automatically changes the field name's first character to uppercase.

So given a struct like this:

```go
  x struct { Value int }
```

You can access `Value` via `value`, which is common in p5-Text-Xslate

```
  [% x.value # same as x.Value %]
```
