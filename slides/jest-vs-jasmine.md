# Jest vs. Jasmine or together?!

---

## Jasmine and Karma
* Are the default when you are using the angular-cli.
* Jasmine is the testing framework
* Karma is the test runner
* Tests are running in real browsers
* Bad start up time, as everything has to compile first with webpack.
* Even with a small change the whole project has to be recompiled first

---
## Jest
* Compiles only the stuff it needs to run the test
* By default it only runs tests from changed files
* Runs tests in parallel (scales with cores)
* Great cli
* Runs tests with "Virtual DOM" -> no real browser is involved
* API is nearly the same as from Jasmine
* Possibility of Snapshot-Testing

---
## Migrating from Jasmine to Jest
...easy in most cases, because the API is nearly the same.
Main difference is in the spying system.

---
## Migration Tools
> We made jest-codemods so you can try out Jest on your existing codebase. We strive to make the migration as smooth as possible, but some manual intervention and tweaks to your tests are to be expected.

https://github.com/skovhus/jest-codemods

---
## Angular support

> A preset of Jest configuration for Angular projects.

https://github.com/thymikee/jest-preset-angular 

---
## How we started
### Initial thoughts
* At the time we started migrating `jest-codemods` was not available
* There were already a lot of tests in the project (2000+)
* All these tests would need a refactoring
* But Jest was way faster than Karma/Jasmine
* We also liked the possibility to run the tests against real browsers like IE (to reduce manual testing) which isn't possible with Jest.

---
## But again Jest was so fast!
   
---
## How we started
### Solution
The Solution for all the problems was a "jasmine polyfill":
* Bridge the API from Jasmine to Jest
* Tests haven't to be refactored (or only a few)
* We are still able to run both Jest and Jasmine (with Karma)

---
## How we started
### How it looks like
```typescript
// Currently not available on npm but when 
// there is time we work in open sourcing this
export function addJasminePolyfill() {
    (window as any).spyOn = spyOn;
    (window as any).spyOnProperty = spyOnProperty;
    (window as any).jasmine.createSpy = createSpy;
}

function createSpy(name: string) {
    const spy = jest.fn();
    spy.calls = {
        argsFor(num: number) {
            return spy.mock.calls[num];
        },
        reset() {
            spy.mockClear();
        },
        first() {
            return { args: spy.mock.calls[0] };
        },
        mostRecent() {
            return { args: spy.mock.calls[spy.mock.calls.length - 1] };
        },
        allArgs() {
            return spy.mock.calls;
        },
    };
    spy.and = {
        returnValue(value: any) {
            spy.mockReturnValue(value);
            return spy;
        },
        callFake(fakeFn: () => any) {
            spy.mockImplementation(fakeFn);
            return spy;
        },
    };
    return spy;
}
// ...

```

---
## How we started
but then it worked so well that we now have both running
