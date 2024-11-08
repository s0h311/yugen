# Yugen: A framework agnostic, lightweight, robust and intuitive state manager

> 有限 / _Yūgen_: japanese for finite, as in _deterministic finite automaton_

Yugen models state management as a [deterministic finite automaton](https://en.wikipedia.org/wiki/Deterministic_finite_automaton). Below is the state machine for a `TodoStore`, which fetches todos.

![](assets/example-fetch-machine.png)

The code looks like this:

```TypeScript
// stores/todoStore.ts

import { createMachine } from '@yugen/machine'

const NOT_FETCHED = 'not fetched'
const FETCHING = 'fetching'
const FETCHED = 'fetched'
const FETCH_FAILED = 'fetch failed'

export const fetchTransition = 'fetch'

type TodoState = typeof NOT_FETCHED | typeof FETCHING | typeof FETCHED | typeof FETCH_FAILED

export const todoMachine = createMachine<TodoState, string[]>({
  state: NOT_FETCHED,
  value: [],
  transitions: {
    [fetchTransition]: {
      sourceState: NOT_FETCHED,
      targetState: FETCHING,
      effect: (value) => {
        return new Promise((resolve, reject) => {
          setTimeout(() => resolve([...value, 'shopping']), 2000)
        })
      },
      successState: FETCHED,
      failureState: FETCH_FAILED,
    },
  },
})
```

You can then use the `todoMachine` like this:

```TypeScript
// pages/TodoList.ts

import { todoMachine, fetchTransition } from '../stores/todoStore.ts'

console.log(todoMachine.state()) // not fetched
console.log(todoMachine.value()) // []

todoMachine.transitionTo(fetchTransition)

console.log(todoMachine.state()) // fetching
console.log(todoMachine.value()) // []

// after 2000ms

console.log(todoMachine.state()) // fetched
console.log(todoMachine.value()) // ['shopping']
```

## Why

Over the past years We observerd that state management in frontend can be quite complex and prone for error. This is why `Yugen` was born as part of my bachelors thesis. Beside the source code, you will also find my thesis here. It includes the design decisions and a comparison to other state management libraries.

## What it is

**Framework agnostic:** `@yugen/machine` does not rely on any UI framework. It is the core of `@yugen/signal` which on the other hand uses the [Stage 1 ECMAScript Signal implementation](https://github.com/tc39/proposal-signals).

**Lightweight:** `@yugen/machine` contains very little code that is easy to understand. We also prioritize ESM.

**Robust:** `Yugen` is well tested and abstracts away [DFAs](https://en.wikipedia.org/wiki/Deterministic_finite_automaton).

**Intuitive:** After experimenting with a lot of state managers, We tried to include the best aspects of each into `Yugen`. We tried to minimize boilerplate.

## What it's not

**Reactive:** `Yugen` is not reactive. It means that changes in state will not update the UI automatically. `@yugen/signal` is also **not** reactive, because it relies on [Stage 1 ECMAScript Signal implementation](https://github.com/tc39/proposal-signals) and not the ones used in the UI frameworks. We plan to add adapters for framework signals, such as `ref` and `reactive` in Vue, `signal` in Angular, and `@preact/signals-react` for React.

**Traceable:** `Yugen`, unlike `Redux` and `NgRx` does **not** have devtools yet. So you cannot trace changes in state.
