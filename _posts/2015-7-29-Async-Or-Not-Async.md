---
layout: post
title: How to work with async & await without deadlocks
---

When working with async & await in code that executes within some sort of synchronization context
(WinForms / WPF UI, ASP.NET), you must be very careful. The well-known deadlock happens
if async-style code is mixed with blocking calls to Task.Wait / Task.Result etc.

There are some best practices for dealing with these situations. They are fully covered in an amazing article,
written by Stephen Cleary: [Async Programming - Brownfield Async Development](https://msdn.microsoft.com/en-us/magazine/mt238404.aspx)

Short summary from this article that relates to blocking on async calls (with my comments):

1. Use async / await all the way down - the most recommended approach. However, this is not always possible with legacy code
2. Use `ConfigureAwait(false)` in all calls with `await` if you don't need to resume on the same context. The key word here is **ALL**. If you use a third-party library which provides async methods and you want to use them in your UI code - be sure that this library follows the mentioned principle before you try to block on returned tasks :).
3. Use the ThreadPool hack - execute async methods (that do not require a specific context) in a separate task from the ThreadPool (using `Task.Run`),
then you can safely call `Task.Result` or `GetAwaiter().GetResult()`. In my opinion, this is the most useful and pragmatic way of dealing with async methods from
third-party libraries in legacy UI code when you have to block anyway.
4. To synchronously retrieve the result of an async call - use `GetAwaiter().GetResult()` instead of `Result` - the former nicely unwraps exceptions (if any)
