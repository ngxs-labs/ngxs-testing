<p align="center">
  <img src="https://raw.githubusercontent.com/ngxs-labs/emitter/master/docs/assets/logo.png">
</p>

---

## NGXS Testing

[![Build Status](https://travis-ci.org/ngxs-labs/testing.svg?branch=master)](https://travis-ci.org/ngxs-labs/testing)
[![NPM](https://badge.fury.io/js/%40ngxs-labs%2Ftesting.svg)](https://www.npmjs.com/package/@ngxs-labs/testing)

```
$ npm install @ngxs-labs/testing --save-dev
```

### Introduction

`@ngxs-labs/testing` is package for configures and initializes environment for ngxs unit testing and provides methods for creating states in unit tests.

### Simple example

Unit testing is easy with NGXS. 

```ts
import { NgxsTestBed } from '@ngxs-labs/testing';

describe('Zoo', () => {

  it('it toggles feed', async(() => {
    const { selectSnapshot, dispatch } = NgxsTestBed.configureTestingStates({ states: [ ZooState ] });
  
    dispatch(new FeedAnimals());
    const feed = selectSnapshot(state => state.zoo.feed);
    
    expect(feed).toBe(true);
  }));
  
});
```

### Unit testing actions

Use `getStateContextMocks` to mock and spy on the first argument of `@Actions` functions: the `StateContext`.

With this state: 
```ts
@State({ name: ZOO_STATE_NAME, defaults: { animals: 1, visitors: 10 } })
class ZooState {
    @Action(ResetAnimalAction)
    public reset(ctx: StateContext<any>) {
        ctx.setState({ animals: 1, visitors: 10 });
        ctx.dispatch(new AddAnimalAction());
    }

    @Action(AddAnimalAction)
    public add(ctx: StateContext<any>, { animalAmount }: AddAnimalAction) {
        const state = ctx.getState();
        ctx.patchState({ animals: state.animals + 1 });
    }
}
```

`getStateContextMocks` allows the following tests:

```ts
it('should set state and dispatch new action', () => {
    const { dispatch, getStateContextMocks } = NgxsTestBed.configureTestingStates({
        states: [ZooState]
    });
    dispatch(new ResetAnimalAction());

    expect(getStateContextMocks[ZOO_STATE_NAME].setState).toHaveBeenCalledWith({ animals: 1, visitors: 10 });

    expect(getStateContextMocks[ZOO_STATE_NAME].dispatch).toHaveBeenCalledWith(new AddAnimalAction());
    expect(getStateContextMocks[ZOO_STATE_NAME].dispatch).toHaveBeenCalledTimes(1);
});

it('should get state and patch to new value', () => {
    const { dispatch, getStateContextMocks } = NgxsTestBed.configureTestingStates({
        states: [ZooState]
    });
    dispatch(new AddAnimalAction());

    expect(getStateContextMocks[ZOO_STATE_NAME].getState).toHaveBeenCalled();
    expect(getStateContextMocks[ZOO_STATE_NAME].patchState).toHaveBeenCalledWith({ animals: 2 });
});
```


