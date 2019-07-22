# Pitfalls

---

## Async / RxJs with Jest
In Jest errors which are thrown async are swallowed!
This means the test can be green but later when you run it in Karma/Jasmine
you see a failed test.

![alt text](./slides/img/after-all-fail.jpg)

---

## Async / RxJs with Jest
```typescript
import { Observable, Subject } from 'rxjs';

interface TestData {
    prop1: string;
    prop2: { prop3: string };
}
class TestComponent {
    myProperty1: string;
    myProperty2: string;

    doSomething(observable: Observable<TestData>): void {
        observable.subscribe((data: TestData) => {
            this.myProperty1 = data.prop1;
            this.myProperty2 = data.prop2.prop3;
        });
    }
}

describe('test rxjs error swallowed', () => {
    it('should do something', () => {
        const instance: TestComponent = new TestComponent();
        const subject: Subject<TestData> = new Subject();
        const prop1: string = 'testi';

        instance.doSomething(subject.asObservable());

        subject.next({ prop1, prop2: null });

        expect(instance.myProperty1).toBe(prop1);
    });

    it('should test something', () => {
        expect(true).toBe(true);
    });
});

```


---
## BrowserAnimationsModule and Snapshot-Tests
The animation modules generates some random classes like

```angular2html
  <hc-table
    class="ng-tns-c1-0 hc-table--card ng-star-inserted"
```

which can change independent from your own changes in another component. 
To detect that changes you have
to run the whole test suite and update the tests.

---
## `.toThrow`
doesn't do a deep equal check e.g. when testing against custom errors:

```typescript
class ServerContextException extends ResourceContextException<
    Server | ServerCreate
> {
    constructor(
        readonly resource: Server | ServerCreate,
        readonly userMessage: string,
        // tslint:disable-next-line
        public readonly error: any
    ) {
        super(resource, userMessage, error);
        Object.setPrototypeOf(this, ServerContextException.prototype);
    }
}

describe('test .toThrow', () => {
    function foo() {
        throw new ServerContextException(
            { id: 1 },
            'my message',
            new Error('testi')
        );
    }

    it('should throw test', () => {
        expect(() => foo()).toThrow(
            new ServerContextException(
                { id: 1 },
                'my message',
                new Error('testi')
            )
        );
    });

    it('should throw something different', () => {
        expect(() => foo()).toThrow(
            new ServerContextException(
                { id: 2 },
                'my message',
                new Error('testi')
            )
        ); // still green!!!
    });
});

```

---
## Not everything is implemented in Virtual DOM
and you have to mock it

```typescript
HTMLMediaElement.prototype.play = () => Promise.resolve()

Object.defineProperty(document.body.style, 'transform', {
    value: () => {
        return {
            enumerable: true,
            configurable: true,
        };
    },
});

Object.defineProperty(document.body.style, 'animation', {
    value: () => {
        return {
            enumerable: true,
            configurable: true,
        };
    },
});

Object.defineProperty(window, 'matchMedia', {
    value: jest.fn(() => {
        return { matches: true };
    }),
});

//...
```

---
## Side effects
Always override and set your variables in a `beforeEach`. You can have random failing tests
if there are side effects!

Bad
```typescript
let myData;

it('test', () => {
	myData = {};
})


it('test2', () => {
	expect(myData).toBe(false);
})
```

---
## Side effects
Better
```typescript
let myMockData;

beforeEach(() => {
	myMockData = {...}
});
```


---
## Out of memory
If you have a large project it can be necessary to increase the memory limit of node:

```bash
node --max_old_space_size=4096 node_modules/karma/bin/karma start
```

---
## "Ever green tests"
Your tests can also have bugs. When you write your test after you finished developing the code
and not in the TDD way (red - green - refactor), then there is the possibility
that you have ever green tests without knowing it...
