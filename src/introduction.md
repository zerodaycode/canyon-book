# Introduction

Welcome to `The Canyon's SQL book`! This is a *work-in-progress* user guide for the `Canyon-SQL` ORM and querybuilder.

## Join our discord

For the latest announcements, frequently asked questions, and discussions about `Canyon` in general, join our [Discord](https://discord.gg/MhwSymk7F6).

## Phylosophy of design

The idea behind `Canyon-SQL` is a really simple one: *Make the developer's life easier*.

This is the core concept of the full development. We wanted to create an *out-of-the-box* solution to quickly
develop any application that interacts with a database. With simplicity in mind. Hiding the ugly
implementation details of handling database connections, connectors or repetitive code for every entity or model.

### Write code that writes code

Most of the powerful concepts in `Canyon.SQL` are achieved through the *Rust* incredible macro system.
This is the perfect definition for metaprogramming: `they write code that writes code`.

In concrete, `Canyon's` most beautiful solutions are created through `procedural` and `derive macros`. By writing attributes attached to a few structs or fields, the user achieve all functionalities provided by the framework.

### The advantages

With this in mind, `Canyon` enables the development of code that requires a persistence solution. Allowing the focus to be placed on application design rather than infrastructure or technical requirements. This facilitates the scalability of the project to meet business requirements.


## The Rust programming language

If you're new to the `Rust` programming language, it is highly recommended that you first familiarize yourself with all of the concepts outlined in the [officially maintained Rust Book](https://www.rust-lang.org/).


## Contributing to Canyon's source code or to the book documentation

We welcome the contributions of new developers in advancing this project. However, in order to maintain order and consistency during development, we request that all contributors adhere to the following guidelines when adding new functionality, modifying or improving the source code, or fixing a bug:

* Check with the community on [Discord](#join-our-discord) to keep updated with the latest announcements, issues and goals;
* Open an issue [here](https://github.com/zerodaycode/Canyon-SQL/issues) explaining what you wish to change or upgrade on the source code;
* Clone the repo locally and work in the source code;
* Open a `PR` to a new branch, and try to documentate as much as possible the changes made in the commits to your working branch.

So, if you wish to *contribute* to the source code:

* `Canyon-SQL` it's a fully open-source project. You can find it [here](https://github.com/zerodaycode/Canyon-SQL)

If you wish to update the documentation with any information that was previously omitted or with changes that have been accepted in a pull request, please proceed as follows:

* The source code for this book is [this one](https://github.com/zerodaycode/canyon-book).

## License

`Canyon-SQL`, and this user guide, are both licensed under the MIT license.
