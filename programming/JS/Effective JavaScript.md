## Semicolon insertion
- Semicolons are only inferred before a `}`, at the end of a line, at the end of a program.
- Semicolons are only inferred when the net token cannot be parsed
- Omitting semicolons before a statement starting with `(`, `[`, `+`, `-` or `/` can be harmful.
- Semicolons should be put explicitly between scripts (minification concerns).
- Never put a newline after flow control words like `return` or `continue` or `++`/`--`.

## Named function expressions
- Named function expressions improve stack traces.
- Consider using named function expressions in development code only (due to concerns related to buggy ES3 environments).

## `new`-agnostic constructors
- Make constructors `new`-agnostic by reinvoking itself with `new` or `Object.create`.

```javascript
function User(name, passwordHash){
    var self = this instanceof User ?
        this :
        Object.create(User.prototype);
    self.name = name;
    self.passwordHash = passwordHash;
    return self;
}
```

## Avoid unnecessary state
- Prefer stateless APIs whenever possible.
- In case an API has to be stateful, document the relevant state that each operation depends on.

## Use duck typing for flexible interfaces
> If it looks like a duck, swims like a duck and quacks like a duck...

- Any object will do as a 'class' as long as it conveys the expected structure.
- Avoid inheritance in cases where structural interfaces are more flexible abd lightweight.
- Use duck typing for mock objects during testing.

## Chaining

- Use method chaining for combining stateless operations.

## Use recursion for asynchronous loops

- Use recursive functions to perform iterations in separate turns of the event queue.
- Recursion performed in separate turns of the event queue __does not__ overflow the call stack.

## Do not block the event queue on computation.

- Avoid expensive algorithms on the main event queue.
- Use the `Worker` API whenever available.
- If the `Worker` API is not available, consider breaking up the computations across multiple turns of the event loop. E.g.:

```js
Member.prototype.inNetwork = function(other, callback) {
    // ...
    function next(){
        for(var i = 0; i < 10; i += 1) {
            // ...
        }

        setTimeout(next, 0);    // scheduling the next set of computations
                                // to free up the main thread.
    }

    setTimeout(next, 0);    // scheduling the next set of computations
                            // to free up the main thread.
};
```

## Use a counter to perform concurrent operations

- The `forEach`, `map` etc. methods inject an `index` parameter into the callback - use it to avoid races while performing series of asynchronous operations.

## Never call asynchronous callbacks synchronously

- Do not do it even when the data is immediately available.
- Doing so disrupts the expected operation sequence and may lead to unwanted code interleaving.
- It can lead to stack overflows or mishandled exceptions.
- use `setTimeout(callback, 0)` to schedule an asynchronous callback.
