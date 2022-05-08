# Phylosophy of design

The idea behind `Canyon-SQL` it's a really simple one. *Make the developer's life easier*.

This is the core concept of the full development. We wanted to create an *out of the box* solution to quickly
develop any application that interacts with a database, with the simplicity in mind and hiding the ugly
implementation details of handling database connections and connectors, writing repetive code every time for
every entity or model (concepts just for tell that a struct should map a database entity, usually a table).

This is done via the powerful concept of `write code that writes code`, and in `Rust`, this is done with macros.
In concrete, `Canyon's` most beautiful solutions are created through `procedural macros`, letting the user
achieve functionalities provided by the framework just writting attributes attached to some entities.

So, with this in mind, `Canyon` provides you the power of develop code that needs a persistence solution just thinking 
and focusing in your application design, not in the infrastructure or technical requirements, making it as scalable 
as you want (or at least, as far as your business requirements let you go).