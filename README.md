Recitation 1: CodeQL
====================

[CodeQL](https://semmle.com/codeql) is a domain-specific language for program
analysis. CodeQL works by compiling code and building a database of information
that can be queried (e.g., variables, types, functions, dataflow) and combined
to build up an analysis. GitHub currently attempts to automatically generate a
CodeQL database for open-source projects on its platform, and you can use the
query console at [LGTM](https://lgtm.com/) to run analyses on projects.

Recitation Goals:
-----------------
In this recitation we will:

1. Get set up to run queries using GitHub and LGTM.
2. Get an overview of how a query works and how to navigate the interface.
3. Learn how to write simple analyses targeting JavaScript.

Setup:
------
1. Log in to your GitHub account.
2. Go to [LGTM](https://lgtm.com/) and sign in using your GitHub account.
3. Open the LGTM query console.
4. Select "JavaScript" as the target language and the `meteor/meteor` demo
   project.

Example Query 1, "Functions with many parameters":
--------------------------------------------------
Click the "View some example queries" button and scroll down to the "Functions
with many parameters" query. The query console should populate with the
following code:

```
import javascript

from Function f
where f.getNumParameter() > 10
select f
```

Here is an explanation of each part of the code:

| Query                            | Purpose                                                       |
|----------------------------------|---------------------------------------------------------------|
| `import javascript`              | Tells CodeQL to use the JavaScript library                    |
| `from Function f`                | Defines a variable `f` that ranges over Functions             |
| `where f.getNumParameter() > 10` | Constrains `f` to only functions with more than 10 parameters |
| `select f`                       | Tells CodeQL to display all matching `f`                      |

Run this example query on `meteor/meteor`. You should find one result. You can
click the result to jump to it in the code.


Example Query 2, "If statements with empty then branch":
--------------------------------------------------------
Load the "If statements with empty then branch" example code. You should get the
following:

```
import javascript

from IfStmt i
where i.getThen().(BlockStmt).getNumStmt() = 0
select i
```

This query finds `if` statements guarding an empty block. Note that this query
uses a cast (the `(BlockStmt)`). The return type of `getThen()` is `Stmt`, but
we are only interested in empty blocks, and the cast adds this constraint. See
the [JavaScript AST class
reference](https://help.semmle.com/QL/learn-ql/javascript/ast-class-reference.html)
for more info on classes.

Another way of doing this is to introduce a new variable and constrain it to be
equal to `getThen()`. This is useful if you'd like to use the new variable in
several places. Try running the following query:

```
import javascript

from BlockStmt b, IfStmt i
where
  i.getThen() = b and
  b.getNumStmt() = 0
select i, b.getContainer()
```

Note that `and` and `or` can be used to combine queries, and `select` can take a
comma-delimited list of values you'd like to inspect.

Example Query 3, "Functions without return statements":
-------------------------------------------------------
Load the "Functions without return statements" example. You should get this
code:

```
import javascript

from Function f
where
  exists(f.getABodyStmt()) and
  not exists(ReturnStmt r | r.getContainer() = f)
select f
```

This query uses the [`exists`
quantifier](https://help.semmle.com/QL/ql-handbook/formulas.html#exists). `exists`
is true if there is at least one set of variables that can make it true. For
example `exists(f.getABodyStmt())` is true if there is at least one statement in
the body of the function `f`. You can also define local variables in
`exists`. The syntax for this is `exists(<variables> | <formula>)`.

### Exercise 1:
Try extending this query to also filter out functions containing `if` statements.

Predicates:
-----------
Quantifiers can also be abstracted into predicates which can be used as
quantifiers themselves. For example, we can refactor the "If statement with
empty then branch" example using predicates:

```
import javascript

predicate empty_block(BlockStmt b) {
  b.getNumStmt() = 0
}

predicate if_empty_then(IfStmt i) {
  empty_block(i.getThen())
}

from Stmt s
where if_empty_then(s)
select s
```

### Exercise 2: 
Try extending this query to also match both empty `then` blocks and  empty `while` statements.

Resources:
----------
These resources will help when writing your own CodeQL queries:

- [The QL Language Handbook](https://help.semmle.com/QL/ql-handbook/)
- [JavaScript AST class
  reference](https://help.semmle.com/QL/learn-ql/javascript/ast-class-reference.html)
