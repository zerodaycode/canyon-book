# The Canyon-SQL Book

This repository holds the [Canyon-SQL](https://github.com/zerodaycode/Canyon-SQL) reference guide. It follows the current API from installation and datasource configuration through entities, CRUD, relationships, query building, direct SQL, errors, and repository adapters. Migrations are documented as experimental rather than presented as a supported production workflow.

Read the published book at [zerodaycode.github.io/canyon-book](https://zerodaycode.github.io/canyon-book/). The source is in [`src/SUMMARY.md`](src/SUMMARY.md) and the chapters it links to.

To preview a checkout locally, install [mdBook](https://rust-lang.github.io/mdBook/) and run:

```sh
mdbook serve --open
```

The hosted site updates when changes to the book are published. When changing an example, please compare it with the current Canyon-SQL source and its tests; the documentation should not promise an API that has not landed.
