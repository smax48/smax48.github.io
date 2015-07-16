---
layout: post
title: How to asynchronously process multiple items in JavaScript with a fixed delay between executions
---

Assume you have an array of items and you must somehow process them sequentially and asynchronously, e.g. using a web service call.
In addition, you must insert a delay between executions, for instance if the web service implements request throttling.

You cannot just organise a for-loop and use setTimeout() directly, because in this case all your requests would start without  any delay between calls.

However, if you use promises (or similar constructs) there is one very simple solution.

```javascript
function processItemsWithDelay(items, process, context, delay) {
	var seq = Promise.resolve();

	if (items.length == 0)
		return seq;

	return items.reduce(function (s, item) {
		return s.then(function() {
			return new Promise(function(resolve) {
				setTimeout(resolve, delay);
			});
		}).then(function () {
			return process.call(context, item);
		});
	}, seq);
}
```

Here the "process" parameter is a function that processes items (it must accept one argument and doesn't have to return a promise).
Basically, we construct a chain of promises where real calls alternate with timeouts.
