# Speed

---

## Bottlenecks
now we have improved the developer experience by using Jest, we found out that there are some more bottlenecks
in our tests....

---
## Bottlenecks
### Module loading from Testbed

The default snippet which is generated from the `angular-cli` for 
testing a component looks like:

```typescript
beforeEach(async(() => {
    TestBed.configureTestingModule({
      declarations: [ ComponentOfModuleComponent ]
    })
    .compileComponents();
  }));
```

but with this, the `TestBed` is resetted and all 
compiled stuff is disposed and recreated every time.

---
## Bottlenecks
### `fixture.detectChanges()`
every `fixture.detectChanges()` costs some extra time (depending how large and complex the template is)


---
## Bottlenecks
### Importing Modules
At the beginning we had a lot of large modules which where imported into our tests. Each component has 
to be compiled and loaded! Even if a test doesn't need the component the work is done...

---
## Bottlenecks
### Unit tests are fast by default?
...not really when the template is involved by calling `fixture.detectChanges()` in the `beforeEach`...


---
## Solutions

---
## Solutions
### Limiting `fixture.detectChanges();`
The default template generated with the `angular-cli` looks like 

```typescript
  beforeEach(() => {
    fixture = TestBed.createComponent(ComponentOfModuleComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });
```

---
## Solutions
### Limiting `fixture.detectChanges();`
but in the most cases you change some properties afterwards. So there is an unnecessary 
trigger of the change detection. Better:

```typescript
  beforeEach(() => {
    fixture = TestBed.createComponent(ComponentOfModuleComponent);
    component = fixture.componentInstance;
  });

  it('should create', () => {
    component.property = '123';
    fixture.detectChanges();
    expect(component).toBeTruthy();
  });
```

---
## Solutions
### Don't import modules, declare components 
If you import a module everything in the module is loaded and compiled.
Instead you can declare the component without the unneeded stuff.

---
## Solutions
### Don't import modules, declare components 
Even better is when you have a Mock of the component with an empty template, so you don't have to provide
all it's dependencies.

```typescript
const integrationConfig: TestModuleMetadata = {
    declarations: [
    	NetworkDetailsComponent,
        NetworkBarMockComponent,
        DetailsAdvancedMockComponent,
        NetworkSimpleMockComponent,
        NetworkSnippetModalMockComponent,
    ],
    providers: [],
};
```

---
## Solutions
### Eliminating the template in Unit-Tests
When you only run unit tests against the API of the component you can eliminate the template with

```typescript
TestBed.overrideTemplate(MyComponent, '');
```

so no need for `fixture.detectChanges();`. If you need the functionality which is triggered in
`ngOnInit()` you can call it manually e.g. in a `beforeEach`.


---
## Solutions
### @hetznercloud/ngx-prepare-test-environment
Provides the function `prepareTestEnvironment` which overrides `TestBed.resetTestingModule`.
So all the compiled stuff isn't disposed.

https://www.npmjs.com/package/@hetznercloud/ngx-prepare-test-environment

---
## Solutions
### @hetznercloud/ngx-prepare-test-environment

```typescript
import { ComponentFixture, TestBed, TestModuleMetadata } from '@angular/core/testing';
import { prepareTestEnvironment } from '@hetznercloud/ngx-prepare-test-environment';
import { MyTestComponent } from './my-test.component';

const sharedConfig: TestModuleMetadata = {
        imports: [],
        declarations: [MyTestComponent],
        providers: [],
};

describe('MyTestComponent #unit', () => {
    let comp: MyTestComponent;
    let fixture: ComponentFixture<MyTestComponent>;

    prepareTestEnvironment(sharedConfig, MyTestComponent);

    beforeEach(() => {
        fixture = TestBed.createComponent(MyTestComponent);
        comp = fixture.componentInstance;
    });

    it('should create', () => {
        expect(comp).toBeTruthy();
    });
});

const integrationConfig: TestModuleMetadata = {
        imports: [],
        declarations: [],
        providers: [],
};

describe('MyTestComponent #integration', () => {
    let comp: MyTestComponent;
    let fixture: ComponentFixture<MyTestComponent>;

    prepareTestEnvironment(sharedConfig, integrationConfig);

    beforeEach(() => {
        fixture = TestBed.createComponent(MyTestComponent);
        comp = fixture.componentInstance;
    });

    it('should create', () => {
        fixture.detectChanges();
        expect(comp).toBeTruthy();
    });
});


describe('MyTestComponent #snapshot', () => {
    // let comp: MyTestComponent;
    let fixture: ComponentFixture<MyTestComponent>;

    prepareTestEnvironment(sharedConfig, integrationConfig);

    beforeEach(() => {
        fixture = TestBed.createComponent(MyTestComponent);
        // comp = fixture.componentInstance;
    });

    it('should create', () => {
        fixture.detectChanges();
        expect(fixture).toMatchSnapshot();
    });
});

```

---
## Solutions
### @hetznercloud/ngx-prepare-test-environment
without

![alt text](./slides/img/test-run-with-reset.png)

---
## Solutions
### @hetznercloud/ngx-prepare-test-environment
with it

![alt text](./slides/img/test-run-without-reset.png)


---
### Analyse your own tests
and find slow tests. One slow test is in general no problem but in sum...

![alt text](./slides/img/analyse-tests.png)
