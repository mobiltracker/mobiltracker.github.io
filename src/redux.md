# Redux and Data Management

## Introduction

The goal of this documentation is to help you understand how to handle database information on Front-End. We'll mainly talk about [Redux](https://redux.js.org/introduction/getting-started), what it is, and good practices when using it.

To exemplify we'll use some aspects of the [Admin VanEscola](https://github.com/mobiltracker/admin-vanescola) implementation.

## Conceptual Basis

Imagine we have a set of data that is used in various parts of the application. We wouldn't want each page could modify this data arbitrarily because it could lead to hard-to-reproduce bugs.

That's why we use Redux. It is a library used to separate the data management logic from the screen logic. With it, to change something in our data, we need to dispatch an **Action** that will handle it and specify what happened.

We can think of **Actions** as a way to help us understand what's going on in the app. When something changes, we know why and where it changed.

Having our data as states and the actions to modify them, we also need a function called **Reducer**. It takes a state and an action as arguments and returns the next state of the application.

### React-Redux

For compatibility reasons, we use in our projects the [React-Redux](https://react-redux.js.org/introduction/getting-started) library. It gives us useful hooks to get the data (`useSelector`) and dispatch actions (`useDispatch`). For more informations on the differences bewtween Redux and React-Redux you can read the documentation linked above.

The usage of these hooks and the actions and reducers will be more detailed in the following tutorial session.

## Implementation Tutorial

As said before, we'll use the implementation in [Admin VanEscola](https://github.com/mobiltracker/admin-vanescola) as an example. We start by coding the roducer.

### Reducer

A reducer is a function that receives a state and an action and transforms it in another state. It has an initial state and uses the action type to decide what it needs to do.

The way we type the state helps us with the data management.

```javascript
export type ParentsState =
  | { tag: "UNLOADED" }
  | { tag: "LOADING" }
  | { tag: "ERROR" }
  | {
      tag: "LOADED",
      parents: Parent[],
    };
```

> The tags "UNLOADED", "LOADING" and "ERROR" can be easily treated in the component and we can be assured that the data we want is loaded and without errors.

Using this typing we have two different scenarios. One where the data isn't ready – either because it's loading or because an error occured with the get API request – and another where it is and can be modified.

In our reducer, we'll treat those scenarios in two separate `switch`. The initial state will be "UNLOADED". Then, after the GET API Request, it can result in an error or success. The state returned by the first switch wil make it clear.

```javascript
export default function parents(
  state: ParentsState = initialState,
  action: ParentsAction
): ParentsState {
  if (state.tag !== "LOADED") {
    switch (action.type) {
      case "GET_PARENTS_LOADING":
        return { tag: "LOADING" };

      case "GET_PARENTS_ERROR":
        return { tag: "ERROR" };

      case "GET_PARENTS_OK":
        return { tag: "LOADED", parents: action.data };

      default:
        return state;
    }
  }

  [...]
```

The second switch can assume that the data is loaded and ready to be manipulated, so it has actions like PUT and DELETE. It returns the already loaded state, with the modifications made within the application.

```javascript
  [...]

  switch (action.type) {
    case "PUT_PARENT":
      return {
        ...state,
        parents: state.parents
          .filter((p) => p.userId !== action.data.userId)
          .concat(action.data),
      };

    case "DELETE_PARENT":
      return {
        ...state,
        parents: state.parents.filter((p) => p.userId !== action.data),
      };

    default:
      return state;
  }
}
```

> Notice that every reducer receives all of the page messages (`GET_SCHOOLS`, `PUT_STUDENTS`, etc), these actions shouldn't modify the state of this reducer, so we need a `default` that returns the current state.

### Action

You can think of an action as an event that describes something that happens in our application. It always has a type property, which makes it clear what's going on, and it may have a data property.

We use them specially to enclose our API calls, and we define a type that has each of our routes or events.

```javascript
export type ParentsAction =
  | { type: "GET_PARENTS_ERROR" }
  | { type: "GET_PARENTS_LOADING" }
  | { type: "GET_PARENTS_OK", data: Parent[] }
  | { type: "PUT_PARENT", data: Parent }
  | { type: "DELETE_PARENT", data: string };
```

> Notice that the type makes explict what is happening in the application.

We then define functions that will make the api requests and **dispatch** one of the action types. The dispatch is used to trigger the reducer.

The get _something_ action is one of the most common, it is used to request the data from the API. When it starts, it makes a dispatch to modify the state to "LOADING". After the request is finished, it makes another dispatch that will depend if everything went right.

```javascript
export function getParentsAction(): ThunkAction<Promise<void>> {
  return async (dispatch) => {
    dispatch({ type: "GET_PARENTS_LOADING" });
    try {
      const response = await apiRequest<any[]>({ url: "owner/parents" });
      const data = normalizeParents(response.data);
      dispatch({ type: "GET_PARENTS_OK", data: data });
    } catch {
      dispatch({ type: "GET_PARENTS_ERROR" });
    }
  };
}
```

> We use ThunkAction as a way to turn dispatch into an async function.

We also have the actions that deal with PUT, POST and DELETE API Requests. It is similar to the GET action because it also waits for an API request and then makes a dispatch.

The main differences are: it will modify something in the data that is already loaded, so we don't have to dispatch the loading; and it usually has a return that indicates if it was succesful or not. This way the components can deal with errors.

```javascript
export function putParentAction(
  name: string,
  phoneNumber: string,
  parentId: string
): ThunkAction<Promise<Result>> {
  return async (dispatch, getState) => {
    const parentsState = getState().parents;

    if (parentsState.tag !== "LOADED") return err({});

    const editedParent = parentsState.parents.find(
      (p) => p.userId === parentId
    );

    if (!editedParent) return err({});

    editedParent.name = name;
    editedParent.phoneNumber = phoneNumber;

    try {
      await apiRequest({
        url: `owner/parent/${parentId}`,
        method: "PUT",
        data: {
          name,
          phoneNumber,
        },
      });

      dispatch({ type: "PUT_PARENT", data: editedParent });

      return ok({});
    } catch {
      return err({});
    }
  };
}
```

> Notice that with `ThunkAction` we can use `getState()` to access the current state.

### Store

The store is the combined state of all reducers and we use the `useSelector()` hook to access the specific state that we need.

```javascript
const studentsState = useSelector(selectStudentsState);
const parentsState = useSelector(selectParentsState);
const schoolsState = useSelector(selectSchools);
```

`selectParentsState()` is a simple function to return the state from the store.

```javascript
export const selectParentsState = (store: StoreState) => store.parents;
```

Since we have the tags in the state, after getting it with the `useSelector()`, it`s easy to treat loading states and error states.

The store is configured in the `configureStore.ts` file. and then we can go to `index.tsx` and have the `Provider` as the outermost component. This way the store is accessable in the entire application.

```javascript
ReactDOM.render(
  <Provider store={store}>
    <PersistGate loading={null} persistor={persistor}>
      <App />
    </PersistGate>
  </Provider>,
  document.getElementById("root")
);
```

### Implementing it in the page

If everything is configured appropriately, using the data in a screen or component should be easy. We only have to follow a few steps. Notice that the following component is a simplified version to ilustrate.

First, we use the `useSelector()` hook to get the current state of the data we need.

```javascript
export function ParentsScreen() {
  const parentsState = useSelector(selectParentsState);
  const studentsState = useSelector(selectStudentsState)

  [...]
```

Then, we must verify if the state is in loading or in error and treat it accordingly.

Sometimes we need to access more than a single state and the validations must be done to each one of them.

```javascript
  [...]

  if (isParentsLoading(parentsState) || isStudentsLoading(studentsState))
    return <div className="alert alert-info">Carregando...</div>;

  if (
    parentsState.tag === "ERROR" ||
    studentsState.tag === "ERROR"
  )
    return (
      <div className="alert alert-info">
        Ocorreu um erro, por favor tente novamente mais tarde.
      </div>
    );

  [...]
```

After that, we can be assured that the data is loaded and without errors and start using it.

```javascript
  [...]

  const students = studentsState.students;
  const parents = parentsState.parents;

  return (
    <div>
      <ParentComponent parents={parents} students={students}></ParentComponent>
    </div>
  );
}
```

The `useDispatch()` hook is used to call an action, for example in the `onClick` of the save button of a form.

```javascript
const dispatch = useDispatch()

[...]

<button
  onClick={async () => {
    const resp = await dispatch(
      putParentAction(currentName, currentPhone, myself.userId)
    );
    setSuccess(resp.success ? "success" : "error");
  }}
>
  Salvar
</button>
```

The return of the action is used to verify if everything went right or not and treat it properly.

## Data Fetching and Cache Maintenance

Ainda não temos um exemplo de boa implementação de "data fetching and cache maintenance", mas para fins didáticos o admin VanEscola funciona da seguinte maneira:

We don't have an example of good implementation for data fetching and cache maintenance. But, for didactic purposes, this is how we do it in VanEscola Admin: when the user logs in, **everything** is loaded.

```javascript
function Main() {
  const isAuthenticated = useSelector<StoreState, unknown>(
    (s) => s.login.authenticated
  );
  const dispatch = useDispatch();

  useEffect(() => {
    if (isAuthenticated) {
      dispatch(getStudentsAction());
      dispatch(getSchoolsAction());
      dispatch(getParentsAction());
      dispatch(getProfileAction());
    }
  }, [isAuthenticated, dispatch]);

  [...]
}
```

We also have a throttle to avoid receiving too many requests at the same time.

```javascript
export function refreshStateAction(): ThunkAction<Promise<void>> {
  const time = Date.now();
  const diff = time - lastTime;

  if (diff < throttleTimeout) {
    return () => Promise.resolve();
  } else {
    lastTime = time;
    return async (dispatch) => {
      dispatch(getParentsAction());
      dispatch(getStudentsAction());
      dispatch(getSchoolsAction());
    };
  }
}
```

Usually, when we make a `post`, `put` or `delete` request, we also call a `get` in order to update the data in the state. But sometimes we can update manually in the reducer.
