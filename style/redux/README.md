# Redux Style Guide

## Table of Contents

- [Redux directory](#redux-directory)
- [Action creators](#action-creators)
- [Action types](#action-types)
- [Reducers](#reducers)
- [Connecting React components](#connecting-react-components)

## Redux directory

Redux related logic should all live under a separate directory `redux/` in
order to isolate it from app logic

## Action creators

- Group related action creators in one file.
- Use [redux-promise-middleware] for asynchronous actions, and/or [redux-thunk]
  if you need more flexibility.
- Expose each action creator as a named export.
- Don't use a default export.

```js
// actions/todo.js
import actionTypes from '../constants/actionTypes';
import * as api from '../../utils/api';

export function addTodoSync() {
  return {
    type: actionTypes.ADD_TODO,
    payload: { foo: 'bar' },
  };
}

export function addTodoAsync() {
  return {
    type: actionTypes.ADD_TODO_ASYNC,
    payload: api.addTodoAsync(),
  };
}
```

Action creators are functions which return an action object. An action object
contains a `type` and optionally a `payload`, `error` and `meta`data. Action
objects should follow the [Flux Standard Action] schema.

We could also use the `createAction` helper of [redux-actions], but other than
enforcing FSA it doesn't do much here in terms of reducing boilerplate. In fact
`createAction` is less readable and doesn't let us easily export named
functions. Note that only `type` is mandatory.

So far the common practice is to group files by type rather than by feature, and
this style guide follows that pattern. But there's an alternative pattern -
[ducks] - a proposal to bundle action creators with their reducers into a single
file. Such a bundle is called a 'duck'. Its goal is to move towards a modular
application structure.

[redux-promise-middleware]: https://github.com/pburtchaell/redux-promise-middleware
[redux-thunk]: https://github.com/gaearon/redux-thunk
[Flux Standard Action]: https://github.com/acdlite/flux-standard-action
[redux-actions]: https://github.com/acdlite/redux-actions
[ducks]: https://github.com/erikras/ducks-modular-redux

## Action types

- Put action types in one file of constants.
- Use a tool like [mirror-creator] for generating constant strings
- Expose all action types as the default export.

```js
// constants/actionTypes.js
import mirrorCreator from 'mirror-creator';

export default mirrorCreator([
  'ADD_TODO',
  'ADD_TODO_ASYNC',
]);

// actions/todo.js
import actionTypes from '../constants/actionTypes';

dispatch({ type: actionTypes.ADD_TODO });
```

Action types identify the nature of the action that has occurred. Two actions
with the same type MUST be strictly equivalent (using ===). By convention, type
is usually a string constant or a Symbol.

[mirror-creator]: https://github.com/shakacode/mirror-creator

## Reducers

- Expose reducers as the default export.
- Expose handlers as named exports for unit testing.
- Use constants instead of inline strings for action types.
- If applicable, define an `INITIAL_STATE` variable.

```js
// using redux-promise-middleware
// using type-to-reducer

// reducers/todo.js
import typeToReducer from 'type-to-reducer';
import actionTypes from '../constants/actionTypes';

const INITIAL_STATE = {
  data: {},
  error: false,
  isRequesting: false,
};

export function addTodo(state, action) {
  return { ...state, data: action.payload };
}

export const addTodoAsync = {
  PENDING: (state) => ({
    ...state,
    isRequesting: true,
  }),
  REJECTED: (state, action) => ({
    ...state,
    isRequesting: false,
    error: action.payload,
  }),
  FULFILLED: (state, action) => ({
    ...state,
    isRequesting: false,
    data: action.payload,
  }),
};

export default typeToReducer(
  {
    [actionTypes.ADD_TODO]: addTodo,
    [actionTypes.ADD_TODO_ASYNC]: addTodoAsync,
  },
  INITIAL_STATE
);
```

The final reducer should be exposed as the default export. Individual handler
functions should be exposed as named exports in order to simplify unit testing.

The simplest way to create a reducer is to use the `handleActions` method of
[redux-actions]. This avoids having to write a switch statement and a default
handler. Another benefit is that it enforces the use of FSA-compliant action
objects. Alternatively, use [type-to-reducer] for simplifying handling of async
actions, as it allows nesting of related actions.

A state subset which represents the result of an async action should have the
following format:

```js
{
  data: {},
  isRequesting: false,
  error: false,
}
```

[type-to-reducer]: https://github.com/tomatau/type-to-reducer
[redux-actions]: https://github.com/acdlite/redux-actions

## Connecting React components

- Use [react-redux]
- Use a state selector function `mapStateToProps` that returns the
  minimal state subset you need. If not applicable use `null`.
- Use a action dispatch selector function `mapDispatchToProps` that wraps action
  creators into a `dispatch` call so they may be invoked directly. If not
  applicable omit it.
- Expose the connected component as default export.
- Expose the unconnected component as named export for unit testing.

```js
import { bindActionCreators } from 'redux';
import { connect } from 'react-redux';
import { addTodoAsync } from '../../redux/actions/todo';

export class TodoList extends React.Component { ... }

function mapStateToProps(state) {
  return { todos: state.todos.data };
}

function mapDispatchToProps(dispatch) {
  return bindActionCreators({ addTodoAsync }, dispatch);
}

export default connect(mapStateToProps, mapDispatchToProps)(TodoList);
```

The [reselect] library works well for more complex selectors and to achieve
better performance.

[react-redux]: https://github.com/reactjs/react-redux
[reselect]: https://github.com/rackt/reselect
