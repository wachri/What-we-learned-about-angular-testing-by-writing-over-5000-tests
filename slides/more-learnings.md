# More learnings

---

## Build your own schematics
We have build our own schematics which generate also a `my-component.component.mock.ts` file,
which has the same selector like the real component to mock it out in tests.

```typescript
import { Component , ChangeDetectionStrategy } from '@angular/core';

@Component({
    selector: 'hc-test',
    template: ``,
    changeDetection: ChangeDetectionStrategy.OnPush
})
export class TestMockComponent {
}
```

---
## Test Finished pattern
to close subscriptions.

```typescript
    const testFinished: Subject<void> = new Subject();

    afterEach(() => {
        testFinished.next();
    });

    it('should emit false if the page controller has no entries', () => {
        someObservable
            .pipe(takeUntil(testFinished))
            .subscribe(() => {});
    });

```

---
## Snapshots vs Expectations
Snapshots can be really easy to use, but when you have a huge component they can also be confusing.
For critical DOM changes it is better to explicitly test it like:

```typescript
expect(fixture.nativeElement.querySelector('[data-test=my-button]'))
    .toHaveCssClass('hc-tooltip--error');

// or
expect(fixture.nativeElement.querySelector('[data-test=my-button]'))
    .toMatchSnapshot();
```

---
## Dont' forget e2e testing

---
## Start directly with writing tests
They are an investment into the future of your project!
