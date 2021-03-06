redux-await
=============

[![NPM version][npm-image]][npm-url]
[![Build status][travis-image]][travis-url]
[![Test coverage][coveralls-image]][coveralls-url]
[![Downloads][downloads-image]][downloads-url]

Manage async redux actions sanely

## Install

```js
npm install --save redux-await
```

## Usage

This module exposes a middleware and higher order reducer to take care of async state in a redux app. You'll need to:

1. Apply the middleware:

```js
import { middleware as awaitMiddleware } from 'redux-await';
let createStoreWithMiddleware = applyMiddleware(
  awaitMiddleware,
)(createStore);
```

2. Wrap your reducers

```js
const reducer = (state = [], action = {}) => {
  if (action.type === GET_USERS) {
    return { ...state, users: action.payload.users };
  }
  if (action.type === ADD_USER) {
    return { ...state, users: state.users.concat(action.payload.user) };
  }
  return state;
}

// old code
// export default reducer;

// new code
import { createReducer } from 'redux-await';
export default createReducer(reducer);
```

Note, if you are using `combineReducers` then you need to wrap each reducer that you are combining independently and not the master reducer that `combineReducers` returns

Now your action creators can contain promises, you just need to add `AWAIT_MARKER` to the action like this:

```js
// old code
//export const getUsers = users => ({
//  type: ADD_USER,
//  payload: {
//    users: users,
//  },
//});
//export const addUser = user => ({
//  type: ADD_USER,
//  payload: {
//    user: user,
//  },
//});

// new code
import { AWAIT_MARKER } from 'redux-await';
export const getUsers = users => ({
 type: ADD_USER,
 AWAIT_MARKER,
 payload: {
   users: api.getUsers(), // returns promise
 },
});
export const addUser = user => ({
 type: ADD_USER,
 AWAIT_MARKER,
 payload: {
   user: api.generateUser(), // returns promise
 },
});
```

Now your containers can hardly need to change at all:

```js
import { getInfo } from 'redux-await'

class Container extends Component {
  render() {
    const { users, user } = this.props;

    // old code
    //return <div>
    //  <MyTable data={users} />
    //</div>;

    // new code
    return <div>
      { getInfo(users).status === 'pending' && <div>Loading...</div> }
      { getInfo(users).status === 'success' && <MyTable data={users} /> }
      { getInfo(users).status === 'failure' && <div>Oops: {getInfo(users).error.message}</div> }
      { getInfo(user).status === 'pending' && <div>Saving new user</div> }
      { getInfo(user).status === 'failure' && <div>There was an error saving</div> }
    </div>;
  }
}
```

## Advanced Stuff

By default your reducer is called with the type specified in the action only on the success stage, you can listen to pending an fail events too by listening for `getPendingActionType(type)` and `getFailureActionType(type)` types, also your reducer is called every time (pending, success, failure) after the higher order reducer does it's thing, but you can still get the old state as the third parameter (not sure why you would ever need to though)

It's a little weird using the prop of the action creator to inject it's status info into state, but I couldn't really think of a better way to do it (WeakMaps somehow?)

[npm-image]: https://img.shields.io/npm/v/redux-await.svg?style=flat-square
[npm-url]: https://npmjs.org/package/redux-await
[travis-image]: https://img.shields.io/travis/kolodny/redux-await.svg?style=flat-square
[travis-url]: https://travis-ci.org/kolodny/redux-await
[coveralls-image]: https://img.shields.io/coveralls/kolodny/redux-await.svg?style=flat-square
[coveralls-url]: https://coveralls.io/r/kolodny/redux-await
[downloads-image]: http://img.shields.io/npm/dm/redux-await.svg?style=flat-square
[downloads-url]: https://npmjs.org/package/redux-await
