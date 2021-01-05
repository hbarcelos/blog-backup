## A better approach for testing your Redux code


## TL;DR

When testing Redux, here are a few guidelines:

### Vanilla Redux

- The smallest standalone unit in Redux is the entire state slice. Unit tests should interact with it as a whole.
- There is no point in testing reducers, action creators and selectors in isolation. As they are tightly coupled with each other, isolation gives us little to no value.
- Tests should interact with your Redux slice same way your application will: through action creators and selectors.
- Avoid assertions like `toEqual`/`toDeepEqual` against the state object, as they create a coupling between your tests and the state structure.
    - Selectors give you the granularity you need to run spot on assertions.
- Selectors and action creators should be boring, so they won't require testing.
- Your slice is somewhat equivalent to a pure function, which means you don't need any mocking facilities in order to test it.

### Redux + `redux-thunk`

- Dispatching thunks doesn't have any direct effect. Only after the thunk is called we will have the side-effects we need to make our application work.
- Here you can use stubs, spies and sometimes mocks (but [don't abuse mocks](https://medium.com/javascript-scene/mocking-is-a-code-smell-944a70c90a6a)).
- Because of the way thunks are structured, the only way to test them is by testing their implementation details.
- The strategy when testing thunks is to setup the store, dispatch the thunk and then asserting whether it dispatched the actions you expected in the order you expected or not.

I have created a [repo](https://github.com/hbarcelos/better-redux-tests) implementing the ideas above.

---

## Intro

As a Software Engineer, I am always finding ways to get better at my craft. It is not easy. Not at all. Coding is hard enough. Writing good code is even harder.

Then there are tests. I think every single time I start a new project &mdash; professionally or just for fun &mdash; my ideas on how I should test my code change. Every. Single. Time. This is not necessarily a bad thing as different problems require different solutions, but this still intrigues me a little.

## The Problem with Tests

As a ~most of the time~ TDD practitioner, I have learned that the main reason we write tests it not to assert the correctness of our code &mdash; this is just a cool side effect. The biggest win when writing tests first is that it guides you through the design of the code you will write next. If something is hard to test, there is *probably* a better way to implement it.

However, after if you have done this for some time, you realize that writing good tests are as hard as writing production code. Sometimes is even harder. Writing tests takes time. And extra time is something that your clients or the business people in your company will not give you so easily.

![Hourglass](https://cdn.hashnode.com/res/hashnode/image/upload/v1574465138087/M7XYT_x1t.jpeg)
> Testing?! Ain't nobody got time for that! (Photo by Aron Visuals on Unsplash)

And it gets worse. Even if you are able to write proper tests, throughout the lifespan of the product/project you are working on, requirements will change, new scenarios will appear. Write too many tests, make them very entangled and any minor change in your application will take a lot of effort to make all tests pass again. 

Flaky tests are yet another problem. When it fails, you have no idea were to start fixing it. You will probably just re-run the test suite and if it passes, you are good to go.

![Schrödinger's Paradox](https://cdn.hashnode.com/res/hashnode/image/upload/v1574466460701/jG8dcUyjJ.jpeg)  
> Schrödinger's tests: sometimes they fail, sometimes they pass, but you cannot know for sure (Picture by Jie Qi on Flickr)

But how do you know if you are writing good tests? What the hell is a good test in the first place?

## Schools of Testing

There is an long debate between two different currents of thoughts known as London School and Detroit School of Testing.

Summarizing their differences, while Detroit defends that software should be built bottom-up, with emphasis on design patterns and the [tests should have as little knowledge as possible about the implementation](https://en.wikipedia.org/wiki/Black-box_testing) and have little to no stubbing/mocking at all, London advocates that the design should be top-down, using external constraints as starting point, ensuring maximum isolation between test suites through extensive use of stubs/mocks, which has a side effect of  [having to know how the subject under test is implemented](https://en.wikipedia.org/wiki/White-box_testing).

This is a very brief summary &mdash; even risking being wrong because of terseness &mdash; but you can find more good references about this two decades old conundrum  [here](https://github.com/testdouble/contributing-tests/wiki/Detroit-school-TDD),  [here](https://github.com/testdouble/contributing-tests/wiki/London-school-TDD) and [here](https://medium.com/@adrianbooth/test-driven-development-wars-detroit-vs-london-classicist-vs-mockist-9956c78ae95f).

## Testing in the Real World

So which one is right, Londoners or Detrotians?  [Both of them and neither of them at the same time](https://blog.ncrunch.net/post/london-tdd-vs-detroit-tdd.aspx) . As I learnt throughout the almost five years I have been a professional Software Engineer, dogmatism will not take you very far in the real world, where projects should be delivered, product expectations are to be matched and you have bills to pay.

What you really need is to be able to take the  [best of both worlds](https://blog.ncrunch.net/post/london-tdd-vs-detroit-tdd.aspx) and use it in your favor. Use it wisely.

We live in a world where everybody seems obsessed with ~almost~ perfect code coverage, while the problem of [Redundant Coverage](https://github.com/testdouble/contributing-tests/wiki/Redundant-Coverage) is rarely mentioned &mdash; it is not very easy to find online references discussing this. If you abuse tests, you may end up having a hard time when your requirements suddenly change.

In the end we are not paid to write tests, we are paid to solve other people's problems through code. Writing tests is expensive and does not add **perceivable** value to the clients/users. One can argue that there is value added by tests, but in my personal experience it is very hard to make non-technical people buy that.

What we as Software Engineers should strive for is to write the minimum amount of tests that yields enough confidence in code quality and correctness &mdash; and "enough" is  [highly dependent on context](https://medium.com/javascript-scene/why-cutting-costs-is-expensive-how-9-hour-software-engineers-cost-boeing-billions-b76dbe571957).

## Redux Testing According to the Docs

Redux is known to have an outstandingly good documentation. In fact this is true. There is not only API docs and some quick examples, as there are also some valuable best practices advice and even links to more in depth discussions regarding Redux and its ecosystem.

However, I believe that the ["Writing Tests"](https://redux.js.org/recipes/writing-tests) section leaves something to be desired.

### Testing Action Creators

That section in the docs start with action creators.

```js
export function addTodo(text) {
  return {
    type: 'ADD_TODO',
    text
  }
}
``` 

Then we can test it like:

```js
import * as actions from '../../actions/TodoActions'
import * as types from '../../constants/ActionTypes'

describe('actions', () => {
  it('should create an action to add a todo', () => {
    const text = 'Finish docs'
    const expectedAction = {
      type: types.ADD_TODO,
      text
    }
    expect(actions.addTodo(text)).toEqual(expectedAction)
  })
})
```

While the test is correct and passes just fine, the fundamental problem here is that **it does not add much value**. Your regular action creators should be **very boring**, almost declarative code. You do not need tests for that.

Furthermore, if you use helper libraries like  [`redux-act`](https://github.com/pauldijou/redux-act) or Redux's own [`@reduxjs/toolkit`](https://github.com/reduxjs/redux-toolkit) &mdash; which you **should** &mdash; then there is absolutely no reason at all to write tests for them, as you would be testing the helper libs themselves, which are already tested and, more important, are not even owned by you.

And since action creators can be very prolific in a real app, the amount of test they would require is huge.

> But how can we know for sure our plain-old action creators do not contain silly errors like typos on them?

Bear with me. More on that later.

### Testing reducers

In Redux, a reducers is a function which given a state and an action, should produce an entirely new state, without mutating the original one. Reducers are pure functions. Pure functions are like heaven to testers. It should be pretty straightforward, right?

The docs gives us the following example:

```js
import { ADD_TODO } from '../constants/ActionTypes'

const initialState = [
  {
    text: 'Use Redux',
    completed: false,
    id: 0
  }
]

export default function todos(state = initialState, action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        {
          id: state.reduce((maxId, todo) => Math.max(todo.id, maxId), -1) + 1,
          completed: false,
          text: action.text
        },
        ...state
      ]

    default:
      return state
  }
}
```

Then the test:

```js
describe('todos reducer', () => {
  it('should return the initial state', () => {
    expect(reducer(undefined, {})).toEqual([
      {
        text: 'Use Redux',
        completed: false,
        id: 0
      }
    ])
  })

  it('should handle ADD_TODO', () => {
    expect(
      reducer([], {
        type: types.ADD_TODO,
        text: 'Run the tests'
      })
    ).toEqual([
      {
        text: 'Run the tests',
        completed: false,
        id: 0
      }
    ])

    expect(
      reducer(
        [
          {
            text: 'Use Redux',
            completed: false,
            id: 0
          }
        ],
        {
          type: types.ADD_TODO,
          text: 'Run the tests'
        }
      )
    ).toEqual([
      {
        text: 'Run the tests',
        completed: false,
        id: 1
      },
      {
        text: 'Use Redux',
        completed: false,
        id: 0
      }
    ])
  })
})
```

Let's just ignore the fact that the suggested test case *"should handle ADD\_TODO"* is actually two tests bundled together &mdash; with might  freakout some testing zealots. Even though in this case I believe it would be best to have different test cases &mdash; one for an empty list and the other for a list with some initial values &mdash; sometimes this is just fine.

The  real issue with those tests is that **they are tightly coupled with the internal structure of the reducer**. More precisely, the tests above are coupled to the state object structure through those `.toEqual()` assertions. 

While this example is rather simple, it is very common for the state of a given slice in Redux to change over time, as new requirements arrive and some unforeseen interactions need to occur. If we write tests like the ones above, they will soon become a maintenance nightmare. Any minimal change in the state structure would demand updating several test cases.

> So how exactly are we supposed to write those tests?

## Testing Redux the right way

**Disclaimer:** I am not saying this is the best or the only way of testing your Redux application, however I recently came to the conclusion that doing it the way I suggest bellow yields the best cost-benefit that I know of. If you happen to know a better way, please reach out to me through the comments, Twitter, e-mail or smoke signs.

Here is a popular folder structure for Redux applications that is very similar to the ones that can be found in many tutorials and even the official docs:

```
src
└── store
    ├── auth
    │   ├── actions.js
    │   ├── actionTypes.js
    │   └── reducer.js
    └── documents
        ├── actions.js
        ├── actionTypes.js
        └── reducer.js
```

If you are like me and like to have test files colocated with the source code, this structure encourages you to have the following:

```
src
└── store
    ├── auth
    │   ├── actions.js
    │   ├── actions.test.js
    │   ├── actionTypes.js
    │   ├── reducer.js
    │   └── reducer.test.js
    └── documents
        ├── actions.js
        ├── actions.test.js
        ├── actionTypes.js
        ├── reducer.js
        └── reducer.test.js
```

I have already left `actionTypes` tests out as those files are purely declarative. However, I already explained why action creators should be purely declarative, and therefore should not be tested as well. That leaves us with testing the only reducer itself, but that does not seem quite right.

The problem here is what we understand as being a _"unit"_ in Redux. Most people tend to consider each of the individual files above as being themselves a unit. I believe this is a misconception. Actions, action types and reducers **must** be tightly coupled to each other in order to function properly. To me, it does not make sense to test those "components" in isolation. They all need to come together to form a slice (e.g.: `auth` and `documents` above), which I consider to be the smallest standalone piece in Redux architecture.

For that reason, I am found of the [Ducks](https://github.com/erikras/ducks-modular-redux) pattern, even though  [it has some caveats](https://twitter.com/dan_abramov/status/738405796770353152). Ducks authors advocates everything regarding a single slice (which they call a _"duck"_) should be placed in a single file and follow a well-defined export structure.

I usually have a structure that looks like more this:

```
src
└── modules
    ├── auth
    │   ├── authSlice.js
    │   └── authSlice.test.js
    └── documents
        ├── documentsSlice.js
        └── documentsSlice.test.js
```

The idea now is to write the least amount of test possible, while having a good degree of confidence that a particular slice works as expected. The reason why Redux exists in the first place is to help us manipulate state, providing a single place for our application state to lie in.

In other words, the value Redux provides us is the ability to write and read state from a centralized place, called the store. Since Redux is based on the  [Flux Architecture](https://facebook.github.io/flux/), its regular flow is more or less like this:

![Flux Architecture](https://cdn.hashnode.com/res/hashnode/image/upload/v1574562511467/uPZvznF1P.png)
> The Flux Architecture by Eric Eliott on Medium

### Redux Testing Strategy

In the end of the day, what we want to test is that we are correctly writing to &mdash; through dispatching actions &mdash; and reading from the store. The way we do that is by given an initial state, we dispatch some action to the store, let the reducer to its work and then after that we check the state to see if the changes we expect were made.

However, how can we do that while avoiding the pitfall of having the tests coupled with the state object structure? Simple.  [Always use selectors](https://medium.com/javascript-scene/10-tips-for-better-redux-architecture-69250425af44#975e). Even those that would seem dumb.

Selectors are you slice public API for reading data. They can encapsulate your state internal structure and expose only the data your application needs, at the granularity it needs.  [You can also have computed data and optimize it through memoization](https://github.com/reduxjs/reselect).

Similarly, action creators are its public API for writing data.

Still confused? Let's try with some code using  [`@reduxjs/toolkit`](https://redux-toolkit.js.org/):

Here is my auth slice:

```js
import { createSlice, createSelector } from '@reduxjs/toolkit';

export const initialState = {
  userName: '',
  token: '',
};

const authSlice = createSlice({
  name: 'auth',
  initialState,
  reducers: {
    signIn(state, action) {
      const { token, userName } = action.payload;

      state.token = token;
      state.userName = userName;
    },
  },
});

export const { signIn } = authSlice.actions;

export default authSlice.reducer;

export const selectToken = state => state.auth.token;
export const selectUserName = state => state.auth.userName;
export const selectIsAuthenticated = createSelector([selectToken], token => token !== '');
``` 

Nothing really special about this file. I am using the `createSlice` helper, which saves me a lot of boilerplate code. The exports structure follows more or less the Ducks Pattern, the main difference being that  I don't explicitly export the action types, as they are defined in the `type` property of the action creators (e.g.: `signIn.type` returns `'auth/signIn'`).

Now the test suite implemented using [`jest`](https://jestjs.io/):

```js
import reducer, { initialState, signIn, selectToken, selectName, selectIsAuthenticated } from './authSlice';

describe('auth slice', () => {
  describe('reducer, actions and selectors', () => {
    it('should return the initial state on first run', () => {
      // Arrange
      const nextState = initialState;

      // Act
      const result = reducer(undefined, {});

      // Assert
      expect(result).toEqual(nextState);
    });

    it('should properly set the state when sign in is made', () => {
      // Arrange
      const data = {
        userName: 'John Doe',
        token: 'This is a valid token. Trust me!',
      };

      // Act
      const nextState = reducer(initialState, signIn(data));

      // Assert
      const rootState = { auth: nextState };
      expect(selectIsAuthenticated(rootState)).toEqual(true);
      expect(selectUserName(rootState)).toEqual(data.userName);
      expect(selectToken(rootState)).toEqual(data.token);
    });
  });
});
``` 

The first test case (`'should return the initial state on first run'`) is only there to ensure there is no problem in the definition of the slice file. Notice that I am using the `.toEqual()` assertion I said you should not. However, in this case, since  the assertion is against the constant `initialState` and there are no mutations, whenever the state shape changes, `initialState` changes together, so this test would automatically be "fixed".

The second test case is what we are interested in here. From the initial state, we "dispatch" a `signIn` action with the expected payload. Then we check if the produced state is what we expected. However we do that exclusively using selectors. This way our test is more decoupled from the implementation

If your slice grows bigger, by using selectors when testing state transitions, you gain yet another advantage: you could use only those selectors that are affected by the action you dispatched and can ignore everything else. Were you asserting against the full slice state tree, you would still need to declare those unrelated state properties in the assertion.

An observant reader might have noticed that this style of testing resembles more the one derived from  [Detroit School](https://github.com/testdouble/contributing-tests/wiki/Detroit-school-TDD). There are no mocks, stubs, spies or whatever. Since reducers are simply pure functions, there is no  point in using those.

However, this slice is rather too simple. Authentication is usually tied to some back-end service, which means we have to manage the communication between the latter and our application, that is, we have do handle side-effects as well as the loading state. Things start to get more complicated.

### Testing a More Realistic Slice

The first step is to split our `signIn` action into three new: `signInStart`, `signInSuccess` and `signInFailure`. The names should be self-explanatory. After that, our state needs to handle the loading state and an eventual error.

Here is some code with those changes:

```js
import { createSlice, createSelector } from '@reduxjs/toolkit';

export const initialState = {
  isLoading: false,
  user: {
    userName: '',
    token: '',
  },
  error: null,
};

const authSlice = createSlice({
  name: 'auth',
  initialState,
  reducers: {
    signInStart(state, action) {
      state.isLoading = true;
      state.error = null;
    },
    signInSuccess(state, action) {
      const { token, userName } = action.payload;

      state.user = { token, userName };
      state.isLoading = false;
      state.error = null;
    },
    signInFailure(state, action) {
      const { error } = action.payload;

      state.error = error;
      state.user = {
        userName: '',
        token: '',
      };
      state.isLoading = false;
    },
  },
});

export const { signInStart, signInSuccess, signInFailure } = authSlice.actions;

export default authSlice.reducer;

export const selectToken = state => state.auth.user.token;
export const selectUserName = state => state.auth.user.userName;
export const selectError = state => state.auth.error;
export const selectIsLoading = state => state.auth.isLoading;
export const selectIsAuthenticated = createSelector([selectToken], token => token !== '');
```

The first thing you might notice is that our state shape changed. We nested `userName` and `token` in a `user` property. Had we not created selectors, this would break all the tests and code that depends on this slice. However, since we did have the selectors, the only changes we need to do are in the `selectToken` and `selectUserName`.

Notice that our test suite is still broken, but that is because we fundamentally changed the slice. It is not hard to get it fixed though:

```js
describe('auth slice', () => {
  describe('reducer, actions and selectors', () => {
    it('should return the initial state on first run', () => {
      // Arrange
      const nextState = initialState;

      // Act
      const result = reducer(undefined, {});

      // Assert
      expect(result).toEqual(nextState);
    });

    it('should properly set loading and error state when a sign in request is made', () => {
      // Arrange

      // Act
      const nextState = reducer(initialState, signInStart());

      // Assert
      const rootState = { auth: nextState };
      expect(selectIsAuthenticated(rootState)).toEqual(false);
      expect(selectIsLoading(rootState)).toEqual(true);
      expect(selectError(rootState)).toEqual(null);
    });

    it('should properly set loading, error and user information when a sign in request succeeds', () => {
      // Arrange
      const payload = { token: 'this is a token', userName: 'John Doe' };

      // Act
      const nextState = reducer(initialState, signInSuccess(payload));

      // Assert
      const rootState = { auth: nextState };
      expect(selectIsAuthenticated(rootState)).toEqual(true);
      expect(selectToken(rootState)).toEqual(payload.token);
      expect(selectUserName(rootState)).toEqual(payload.userName);
      expect(selectIsLoading(rootState)).toEqual(false);
      expect(selectError(rootState)).toEqual(null);
    });

    it('should properly set loading, error and remove user information when sign in request fails', () => {
      // Arrange
      const error = new Error('Incorrect password');

      // Act
      const nextState = reducer(initialState, signInFailure({ error: error.message }));

      // Assert
      const rootState = { auth: nextState };
      expect(selectIsAuthenticated(rootState)).toEqual(false);
      expect(selectToken(rootState)).toEqual('');
      expect(selectUserName(rootState)).toEqual('');
      expect(selectIsLoading(rootState)).toEqual(false);
      expect(selectError(rootState)).toEqual(error.message);
    });
  });
});
```

Notice that `signInStart` has less assertions regarding the new state, because current `userName` and `token` do not matter to it. Everything else is much in line with what we have discussed so far.

There is another subtlety that might go unnoticed. Even though the main focus of the tests is the reducer, they end up testing the action creators as well. Those silly errors like typos will get caught here, so we do not need to write a separate suite of tests to prevent them from happening.

The same thing goes for selectors too. Plain selectors are purely declarative code. Memoized selectors for derived data created with `createSelector` from  [reselect](https://github.com/reduxjs/reselect) should not be tested as well. Errors will get caught in the reducer test.

For example, if we had forgotten to change `selectUserName` and `selectToken` after refactoring the state shape and left them like this:

```
// should be state.auth.user.token
export const selectToken = state => state.auth.token;

// should be state.auth.user.userName
export const selectUserName = state => state.auth.userName; 
```

In that case, all test cases above would fail.

### Testing Side-Effects

We are getting there, but our slice is not complete yet. It lacks the part that orchestrates the sign in flow and communicates with the back-end service API.

Redux itself deliberately does not handle side-effects. In order to be able to do that, you need a Redux Middleware that will handle that for you. While you can  [pick your own poison](https://redux.js.org/introduction/ecosystem#side-effects), `@reduxjs/toolkit` already ships with `redux-thunk`, so that is what we are going to use.

In this case, the Redux docs actually has a  [really good example](https://redux.js.org/recipes/writing-tests#async-action-creators), so I basically took it and adapted to our use case.

In our `authSlice.js`, we simply add:

```js
// ...
import api from '../../api';

// ...
export const signIn = ({ email, password }) => async dispatch => {
  try {
    dispatch(signInStart());
    const { token, userName } = await api.signIn({
      email,
      password,
    });
    dispatch(signInSuccess({ token, userName }));
  } catch (error) {
    dispatch(signInFailure({ error }));
  }
};
```

Notice that the `signIn` function is almost like an action creator, however, instead of returning the action object, it returns a function which receives the dispatch function as parameter. This is the "action" that will be triggered when the user clicks the "Sign In" button in our application.

This means that functions like `signIn` are very important to the application, therefore, they should be tested. However, how can we test this in isolation from the `api` module? Enter Mocks and Stubs.

Since this is basically an orchestration component, we are not interested in the visible effets it has. Instead, we are interested in the actions that were dispatched from within the thunk according to the response from the API.

So we can change the test file like this:

```js
import configureMockStore from 'redux-mock-store';
import thunk from 'redux-thunk';
// ...
import api from '../../api';

jest.mock('../../api');

const mockStore = configureMockStore([thunk]);

describe('thunks', () => {
    it('creates both signInStart and signInSuccess when sign in succeeds', async () => {
      // Arrange
      const requestPayload = {
        email: 'john.doe@example.com',
        password: 'very secret',
      };
      const responsePayload = {
        token: 'this is a token',
        userName: 'John Doe',
      };
      const store = mockStore(initialState);
      api.signIn.mockResolvedValueOnce(responsePayload);

      // Act
      await store.dispatch(signIn(requestPayload));

      // Assert
      const expectedActions = [signInStart(), signInSuccess(responsePayload)];
      expect(store.getActions()).toEqual(expectedActions);
    });

    it('creates both signInStart and signInFailure when sign in fails', async () => {
      // Arrange
      const requestPayload = {
        email: 'john.doe@example.com',
        password: 'wrong passoword',
      };
      const responseError = new Error('Invalid credentials');
      const store = mockStore(initialState);
      api.signIn.mockRejectedValueOnce(responseError);

      // Act
      await store.dispatch(signIn(requestPayload));

      // Assert
      const expectedActions = [signInStart(), signInFailure({ error: responseError })];
      expect(store.getActions()).toEqual(expectedActions);
    });
  });
```

So unlike reducers, which are easier to test with Detroit School methodology, we leverage London School style to test our thunks, because that is what makes sense.

Because we are testing implementation details, whenever code changes, our tests must reflect that. In a real world app, after a successful sign-in, you probably want to redirect the user somewhere. If we were using something like  [connected-react-router](https://github.com/supasate/connected-react-router), we would end up with a code like this:

```diff
+import { push } from 'connected-react-router';
 // ...
 import api from '../../api';
 
 // ...
     const { token, userName } = await api.signIn({
       email,
       password,
     });
     dispatch(signInSuccess({ token, userName }));
+    dispatch(push('/'));
   } catch (error) {
     dispatch(signInFailure({ error }));
   }
 // ...
```

Then we update the assert part of our test case:

```diff
+import { push } from 'connected-react-router';
 // ...

 // Assert
 const expectedActions = [
   signInStart(),
   signInSuccess(responsePayload),
+  push('/')
 ];
 expect(store.getActions()).toEqual(expectedActions);
 // ...
```

This is often a criticism against `redux-thunk`, but if you even so decided to use it, that is a trade-off you have to deal with.

## Conclusion

When it comes to the real world, there is no single best approach for writing tests. We can and should leverage both Detroit and London styles to effectively test your applications.

For components which behave like pure functions, that is, given some input, produce some deterministic output, Detroit style shines. Our tests can be a little bit more coarse-grained, as having perfect isolation does not add much value to them. Where exactly we should draw the line? Like most good questions, the answer is "It depends".

In Redux, I have come to the conclusion that a slice is the smallest standalone unit that exists. It makes little to no sense writing isolated tests for their sub-components, like reducers, action creators and selectors. We test them together. If any of them is broken, the tests will show us and it will be easy to find out which one.

On the other hand, when our components exists solely for orchestration purposes, then London style tests are the way to go. Since we are testing implementation details, tests should be as fine-grained as they get, leveraging mocks, stubs, spies and whatever else we need. However, this comes with a burden of harder maintainability.

When using `redux-thunk`, what we should test is that our thunk is dispatching the appropriate actions in the same sequence we would expect. Helpers like [`redux-mock-store`](https://github.com/dmitry-zaets/redux-mock-store) eases the task for us, as it expose more of the internal state of the store than Redux native store.

T-th-tha-that's a-all f-fo-fo-folks!