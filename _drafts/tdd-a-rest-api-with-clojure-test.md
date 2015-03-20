---
layout: post
title: TDD a REST Api with Clojure.Test
---

# TDD a REST Api with Clojure.Test

## Goal

This post will demonstrate how to use test driven development (TDD) to build
a RESTful API in Clojure. The server will utilize the defacto standard Clojure
libraries for building web applications: [Ring][1] and [Compojure][2]. For testing, the
standard Clojure testing library `clojure.test` will be used.

## Our Rest Api

The API we will be designing will be the backend to a slightly less than trivial TODO
List application. The overall goal for the application will be for a *user* to be able
to create multiple *lists* that each contain different *todo* items. For now, we will
focus our efforts on just building *todos* and their owning *lists*. The implementation
of *users* and their associated authentication and authorization will be handled in
later posts.

### The Endpoints

The backend that we will be designing will respond in a typical REST fashion.
Our endpoints will be structured as follows:

```
/lists
    GET:    Get all lists
    POST:   Create a new list
/lists/:list-id
    GET:    Get    list with :list-id
    PUT:    Update list with :list-id
    DELETE: Delete list with :list-id
/lists/:list-id/todos
    GET:    Get all todos     belonging to :list-id
    POST:   Create a new todo belonging to :list-id
/lists/:list-id/todos/:todo-id
    GET:    Get todo    with :todo-id that belongs to :list-id
    PUT:    Update todo with :todo-id that belongs to :list-id
    DELETE: Delete todo with :todo-id that belongs to :list-id
```

## References

[1]: https://github.com/ring-clojure/ring
[2]: https://github.com/weavejester/compojure
