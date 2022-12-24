# Introduction

Welcome to `The Canyon's SQL book`! This is a *work-in-progress* user guide for the `Canyon-SQL` ORM and querybuilder.

## Phylosophy of design

The idea behind `Canyon-SQL` it's a really simple one. *Make the developer's life easier*.

This is the core concept of the full development. We wanted to create an *out-of-the-box* solution to quickly
develop any application that interacts with a database, with the simplicity in mind and hiding the ugly
implementation details of handling database connections and connectors, writing repetive code every time for
every entity or model (concepts just for tell that a struct should map a database entity, usually a table).

### Write code that writes code

Most of the powerful concepts in `Canyon` are achieved through it's incredible macro system.
This is the perfect definition for those macros: `they write code that writes code`.

In concrete, `Canyon's` most beautiful solutions are created through `procedural and derive macros`,
letting the user achieve functionalities provided by the framework only needing to write attributes attached to some structs or fields.

### The advantages

With this in mind, `Canyon` provides you the power of develop code that needs a persistence solution just thinking
and focusing in your application design, not in the infrastructure or technical requirements, making it as scalable
as you want (or at least, as far as your business requirements let you go).

## The Rust programming language

If you're new to the `Rust` programming language, before getting started, it is highly recommended that you familiarize yourself with all of the concepts
outlined in the officially maintained Rust Book before you getting started with `Canyon-SQL`.

## Contributing to Canyon's source code or to the book documentation

We are really glad to meet new developers that push up this project to a new level, but, in order to maintain a certain order
and consistency during development, assuming you want to add new functionality, modify or improve some part of the source code,
or fix a bug, we would like any contributor to respect the following guidelines:

* Open an issue [here](https://github.com/zerodaycode/Canyon-SQL/issues) explaining what do you want to change or upgrade on the source code
* Locally clone the repo and work in the source code
* Open a `PR` to a new branch, and try to documentate as much as posible the changes made in the commits to your working branch.

So, if you want to *contribute* to the source code:

* `Canyon-SQL` it's a fully open-source project. You cand find it [here](https://github.com/zerodaycode/Canyon-SQL)

If you go ahead, and want to upgrade the documentation with things that you missed that must be explained, or with some changes
that have been accepted on a `PR`, and you want to document it:

* The source code for this book is [this one](https://github.com/zerodaycode/canyon-book).

## License

`Canyon-SQL`, and this user guide, are both licensed under the MIT license.
