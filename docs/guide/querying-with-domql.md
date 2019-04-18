---
layout: default
title: Querying with domQL
parent: Guide
nav_order: 4
---

<!-- markdownlint-disable MD022 -->
# Querying with domQL
{: .no_toc }
<!-- markdownlint-enable MD022 -->

In this section we will see how to define props and also learn the complete syntax of domQL.  
If you haven't read it already, take a minute to check the short [tour](/docs/guide/tour.html).

<!-- markdownlint-disable MD022 -->
## Table of contents
{: .no_toc .text-delta }
<!-- markdownlint-enable MD022 -->

* Operators
* Limit, skip and order
* Child queries
* Querying element attributes
* Querying data attributes
* Querying with style
{:toc}

## Operators

### and

```js
ayakashi
    .select()
    .where({
        and: [{
            class: {
                eq: "container"
            }
        }, {
            class: {
                neq: "content"
            }
        }]
    })
```

`and` works the same as most querying/programming languages.  
If **all** of the expressions evaluate to `true`, then the `and` expression is
`true` and we have a match.  
It can contain any number of nested expressions (the example has 2).

### or

```js
ayakashi
    .select()
    .where({
        or: [{
            class: {
                eq: "container"
            }
        }, {
            class: {
                eq: "content"
            }
        }]
    })
```

`or` works the same as most querying/programming languages.  
If **at least one** of the expressions evaluates to `true`, then the `or` expression
is `true` and we have a match.  
It can contain any number of nested expressions (the example has 2).

### eq

```js
ayakashi
    .select()
    .where({
        id: {
            eq: "main"
        }
    })
```

The most basic matching operator.  
The specified attribute on the left must equal the value on the right to have a match.

### neq

```js
ayakashi
    .select()
    .where({
        id: {
            neq: "main"
        }
    })
```

The reverse of `eq`.
The specified attribute on the left must **not** equal the value on the right
to have a match.

### $neq (strict)

```js
ayakashi
    .select()
    .where({
        id: {
            $neq: "main"
        }
    })
```

The strict version of `neq`.  
The difference is that the specified attribute **must exist** but not be equal
to our right value.  
In the above example, the query will match all elements that have an `id`
and it's not equal to `main`.  
The non-strict version would have also matched any element that doesn't have an `id`
at all.

### like

```js
ayakashi
    .select()
    .where({
        href: {
            like: "github.com"
        }
    })
```

`like` will match if the specified attribute **contains** the value on the right.

### nlike

```js
ayakashi
    .select()
    .where({
        href: {
            nlike: "github.com"
        }
    })
```

The reverse of `like`.  
The specified attribute on the left **must not** contain the value on the right.

### $nlike (strict)

```js
ayakashi
    .select()
    .where({
        href: {
            $nlike: "github.com"
        }
    })
```

The strict version of `nlike`.  
The difference is that the specified attribute **must exist** but not contain
the value on the right.  
In the above example, the query will match all elements that have a `href`
but it does not contain `github.com`.  
The non-strict version would have also matched any element that doesn't have a `href`
at all.

### in

```js
ayakashi
    .select()
    .where({
        class: {
            in: ["header", "footer"]
        }
    })
```

Matches any element that has the specified attribute equal to
**at least one** value of the list on the right.  
It's a more compact version of an `or` with a series of `eq`s.

### nin

```js
ayakashi
    .select()
    .where({
        class: {
            nin: ["header", "footer"]
        }
    })
```

The reverse of `in`.  
Matches any element that has the specified attribute **not equal** to
**any** value of the list on the right.  
It's a more compact version of an `and` with a series of `neq`s.

### $nin (strict)

```js
ayakashi
    .select()
    .where({
        class: {
            $nin: ["header", "footer"]
        }
    })
```

The strict version of `nin`.  
The difference is that the specified attribute **must exist** but not be equal
to any value of the list on the right.  
In the above example, the query will match all elements that have a `class`
attribute but not the `header` and `footer` classes.  
The non-strict version would have also matched any element that doesn't have a `class`
attribute at all.

## Limit, skip and order

We can control the amount and ordering of the matches with
`limit`, `skip` and `order`.  

```js
ayakashi
    .select()
    .where({
        class: {
            eq: "container"
        }
    })
    .limit(1)
```

The above example will only have 1 match (if any).  
Instead of `select()` and `limit(1)` you can use the convenience methods
`selectOne()` and `selectFirst()`
(both do the same thing but each can be more readable in different contexts).  

```js
ayakashi
    .selectFirst()
    .where({
        class: {
            eq: "container"
        }
    })
```

To get only the second match, use `skip(1)`  

```js
ayakashi
    .selectOne()
    .where({
        class: {
            eq: "container"
        }
    })
    .skip(1)
```

To match in a reverse order, use `order("desc")`.  
This will return the last element (if any)

```js
ayakashi
    .select()
    .where({
        class: {
            eq: "container"
        }
    })
    .order("desc")
    .limit(1)
```

The `selectLast()` method will also expand to the exact same query

```js
ayakashi
    .selectLast()
    .where({
        class: {
            eq: "container"
        }
    })
```

## Child queries

Sometimes we need to limit the scope of a match. Child queries accomplice just that

```js
ayakashi
    .selectOne("listContainer")
    .where({
        class: {
            eq: "repo-list"
        }
    })

ayakashi
    .select("githubLinks")
    .where({
        href: {
            like: "github.com"
        }
    })
    .from("listContainer")
```

The second query will only search for links inside our first prop
`listContainer`.  

Instead of using `from()` you can also chain the child queries

```js
ayakashi
    .selectOne("listContainer")
    .where({
        class: {
            eq: "repo-list"
        }
    })
    .selectChildren("githubLinks")
        .where({
            href: {
                like: "github.com"
            }
        })
```

Like the normal `select()`, convenience methods also exist for child selects

* `selectChild()` (`limit(1)`)
* `selectFirstChild()` (same as `selectChild()`)
* `selectLastChild()` (`limit(1)` and `order("desc")`)

You can nest child queries indefinitely, using either `from()`
or the convenience methods.

## Querying element attributes

```js
ayakashi
    .select()
    .where({
        and: [{
            href: {
                like: "github.com"
            }
        }, {
            text: {
                eq: "Repo"
            }
        }]
    })
```

Any standard element attribute will work as expected.

## Querying data attributes

The special attributes `dataKey` and `dataValue` make querying data attributes
easy and readable.

```js
ayakashi
    .select()
    .where({
        and: [{
            datakey: {
                eq: "content"
            }
        }, {
            dataValue: {
                like: "Here is my content:"
            }
        }]
    })
```

For `dataKey`, use only the part of the name after the `data-`.  
So `data-content` becomes just `content`.  
They behave like any normal attribute so all of the operators apply here
as well.

## Querying with style

```js
ayakashi
    .select()
    .where({
        "style-background-color": {
            eq: "rgb(40, 167, 69)"
        }
    })
```

We can also query by style.  
Just prepend `style-` to the full qualified
css property and you can match it like any other attribute.  
The query will search the actual evaluated css properties of the element,
so that means external, inline or styles in a `<style>`
element can be matched.  
**Note 1**: Make sure to use the full name of the css property, so instead of
`font`, use `font-size` and instead of `border` or `border-width`,
use `border-bottom-width` etc.  
**Note 2**: For color properties, like `background-color`, the value must be specified
in rgb. Using hex will not work.