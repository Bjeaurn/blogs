# When to (not) use RxJS Subjects?

When using RxJS, it can be a bit difficult to grasp what operator does what. It's difficult to remember what every operator does and how to use them properly. The same goes for Subjects, an often discussed topic of discussion. Although a very powerful tool in itself, more then often you see Subjects abused creating dangerous environments in code.

##  What are Subjects?

A `Subject` is a RxJS concept that creates an `Observable` that you can "push" new data to. This is a very powerful concept, as it allows you to work in reactive way using RxJS; without sacrificing flexibility of sending data and messages like events between application components.

## So why shouldn't I use Subjects?!

The main purpose of working Reactively using a library like RxJS, is that you use Observables throughout your application. In a lot of cases, you do not need to manage this yourself by creating a Subject and pushing data throughout.

The main difference with Subjects is that they are, by nature, hot observables which need to be opened and closed properly. You will need to take into account subscription management, making sure you do not expose the Subject directly and so on. Whereas creation operators can achieve the same result in a safe and predictable way.

The main rule that I see for using Subjects is: 
>Do not use Subjects when you can use another `creation operator`.

So the key here is to know which `Creation operators` exist, and how to use them!

## Creation operators and their function

There are about 10 creation operators. The most commonly used ones are briefly described here:

#### of
Used to return a value or multiple values as an Observable. 

```ts
return of(1, 2, 3)
// returns 1, 2, 3 as separately emitted values.

return of([1, 2, 3])
// returns [1, 2, 3] as a single emitted value.
```

#### from
Turns an array, promise or iterable into an Observable. Emits all values separately.

```ts
return from([1, 2, 3])
// returns 1, 2, 3
```

#### fromEvent
Creates an Observable stream from event(s). For example a mouse click on a HTML element can be turned into a stream of click events using this operator.

```ts
fromEvent(document, 'click')
// returns every click on the document element as a hot observable.
```

#### ajax
Turns a URL or AjaxRequest into an Observable. 

```ts
ajax('https://pokeapi.co/api/v2/pokemon/ditto/')
// returns a single emission value of the response.
```
#### EMPTY
Returns an Observable that instantly completes. Used often in a `catchError()` situation where you want the "broken"/errored Observable to still complete normally.

#### throw
Throws an Observable that errors the given value. Used when you want to handle errors thrown by your business logic in a reactive way throughout your application.

#### interval / timer
Used to create Observables that emit a value after each "interval" or after a given "duration" every specified "duration".

```ts
interval(1000)
// emits every 1000ms.

timer(1000)
// emits once after a second, then completes.

timer(1000, 2000)
// emits for the first time after one second, then emits every two seconds.
```

#### range
Creates an Observable that emits a value for the specified range. 

```ts
range(1, 3)
// emits 1, 2, 3 as separate values.
```

And the less frequently used, more advanced creation operators left:

- `create`, to create an Observable from an Observer that you write yourself.
- `defer`, takes a function to execute upon subscription. Will create an Observable of the value captured on runtime.
- `generate`, generates an Observable sequence based on the initialState provided and generator functions passed.

## An example

So let's say an example that may trigger you to think: "I need a Subject here" is that you want a bit more complicated behavior from your Observable. Let's say you want an Observable that tries to get some data, stops after multiple attempts with a different message. Sounds like a Subject right? 

```ts
const subject: Subject = new Subject()
let i: number = 0
setInterval(() => {
    if(i < 3) {
        subject.next('Write your retrieval logic here')
        i++
    } else {
        subject.next('Retrieval failed')
        subject.complete() // Don't forget this! Else it'll keep running in the background.
    }
}, 1000)

const consume$ = subject.asObservable() // Don't want to expose the Subject's next() and complete() functions outside of it's scope.
consume$.subscribe(console.log)
```

No. This can be done much easier, readable and less complicated with the added benefit of being less risky to use!

```ts
const data = interval(1000).pipe(
    take(3),
    map(() => {
        return 'Write your retrieval logic here' // SwitchMapping directly to another Observable is of course no problem.
    }),
    finalize(() => {
        return 'Retrieval failed'
    })
)

data.subscribe(console.log)
```

// Work in progress: https://stackblitz.com/edit/rxjs-4ug4jb
// https://stackblitz.com/edit/rxjs-jghhpm - Subject for Hot shared observable
// https://stackblitz.com/edit/rxjs-1us9gk
// A better example is perhaps: Taking an ID from a route, and instead of putting that on a self managed ReplaySubject eventbus; you can create an
// observable stream from the Route and offer that as an Observable.
// Will require a bit more work to show the example in just Stackblitz I think?
// Here we go: https://stackblitz.com/edit/angular-ef9qxx?file=src%2Fapp%2Fdata.service.ts
// Works better locally, but shows the idea.


## Ok, so when do I use a Subject?

Whenever you need an Observable that has an unpredictable lifecycle length, and it's values cannot be predicted or gathered from an external source within your code. That's when you use a Subject.

// TODO: Examples voor beide oorzaken.
- Unpredictable lifecycle

An Observable that runs for the entire lifecycle of the application is still predictable in that sense. An `interval(1000)` will tick once every second for as long as it's subscribed to, but an Observable that is created upon the start of a component and should last till the component is removed and teared down for whatever reason is a tad more tricky. Creating a Subject that is contained within the component on initialization and emitting a value when it is marked for teardown. As long as it's contained within the component this is a great way to use a Subject. 
Although, if the teardown of the component is heavily relient on user actions, the argument can be made that a combination of Observables including a `fromEvent()` could give you the exact same Observable stream of user events that will trigger teardown.

The reason why you would need a Subject here is because you setup streams beforehand. If you need other Observable streams to `takeUntil()` on a `toBeDestroyed$` stream, you cannot initialize the `toBeDestroyed$` in the teardown logic; as it will not exist at the point in time when the listening stream is created. As long as you are able to create these streams by, for example, listening to user actions with `fromEvent`; you may not need a Subject.

- Unpredictable values or values cannot be gathered from an external source


