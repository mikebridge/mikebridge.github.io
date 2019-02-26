+++
title = "Testing Redux-Observable Epics"
categories = ["react", "typescript", "redux-observable", "epic", "test"]
description = "Creating Jest tests for Redux-ObservableAjax Epics"
date = "2017-04-26"
aliases = ["articles/typescript-redux-observable-epic-test"]
+++

Here's a way to test a [redux-observable](https://redux-observable.js.org/) epic that 
performs an ajax call.

This simplified TypeScript
example has three actions: `USER_LOAD_REQUEST` is the result of a request to load a user, 
`USER_LOAD_RESULT` indicates a successful response, and `USER_LOAD_ERROR` holds an 
ajax error.

{{< highlight typescript "linenos=table">}}
import {ActionsObservable} from "redux-observable";
import {AjaxError} from "rxjs/Rx";
import "rxjs/Rx";

export const USER_LOAD_REQUEST = "example/USER_LOAD_REQUEST";

export const USER_LOAD_RESULT = "example/USER_LOAD_RESULT";

export const USER_LOAD_ERROR = "example/USER_LOAD_ERROR";

export interface IUserResult {
    id: string;
    name: string;
}

export interface ICustomAjaxError {
    type: string;
    message: string;
}

export const loadUser = (userid: string) => ({
    type: USER_LOAD_REQUEST,
    userid
});

export const loadUserResult = (results: IUserResult) => ({
    type: USER_LOAD_RESULT,
    results
});

export const loadFailure = (message: string): ICustomAjaxError => ({
    type: USER_LOAD_ERROR,
    message
});


export const loadUserEpic = (action$: ActionsObservable<any>, store, {getJSON}) => {
    return action$.ofType(USER_LOAD_REQUEST)
        .do(() => console.log("Locating User ...")) // debugging
        .mergeMap(action =>
            getJSON(`%PUBLIC_URL%/api/users`)
                .map(response => loadUserResult(response as any))
                .catch((error: AjaxError): ActionsObservable<ICustomAjaxError> =>
                    ActionsObservable.of(loadFailure(
                        `An error occurred: ${error.message}`
                    )))
        );

};

export default loadUserEpic;
{{</ highlight >}}

Testing is straightforward with ReduxObservable's 
[dependency injection](https://redux-observable.js.org/docs/recipes/InjectingDependenciesIntoEpics.html) 
parameter.  In this case, I'm injecting the `ajax.getJSON` property into the epic.

Mocking the AJAX call involves either wrapping a plain JS object or a plain JS `Error` in an 
`Observable`.

```typescript
import { ActionsObservable } from "redux-observable";
import "rxjs/Rx";
import { Observable } from "rxjs/Observable";

import * as example from "./exampleEpic";

const successResult: example.IUserResult = {
    id: "123",
    name: "Test User"
};

describe("loadUserEpic", () => {

    // done: see https://facebook.github.io/jest/docs/asynchronous.html

    it("dispatches a result action when the user is loaded", (done) => {

        const dependencies = {
            getJSON: url => Observable.of(successResult)
        };

        const action$ = ActionsObservable.of(example.loadUser(successResult.id));
        const expectedOutputActions = example.loadUserResult(successResult);

        const result = example.loadUserEpic(action$, null, dependencies).subscribe(actionReceived => {
            expect((actionReceived as any).type).toBe(expectedOutputActions.type);
            done();
        });

    });

    it("dispatches an error action when ajax fails", (done) => {
        
        const errorMessage = "Failed Ajax Call";
        
        const dependencies = {
            getJSON: url => Observable.throw(new Error(errorMessage))
        };

        const action$ = ActionsObservable.of(example.loadUser(successResult.id));
        const expectedOutputActions = example.loadFailure(`An error occurred: ${errorMessage}`);

        const result = example.loadUserEpic(action$, null, dependencies).subscribe(actionReceived => {
            expect((actionReceived as any).type).toBe(expectedOutputActions.type);
            expect((actionReceived as any).message).toBe(
                expectedOutputActions.message);
            done();
        });

    });

});
```

Note the [`done`](https://facebook.github.io/jest/docs/asynchronous.html) parameter---this is a way of 
signaling to jest that an async call has completed.



