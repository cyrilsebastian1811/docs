# **:material-react: Angular**

## Setup

- create new project

```bash
ng new my-project
cd my-project
```

- create new component

```bash
ng generate component my-component
```

- create a new directive

```bash
ng generate directive highlight
```

- create a new service

```bash
ng generate service data
```

## Component

- Component are reusable.
- Naming convention
    - file: `name.component.ts`
    - class name: `NameComponent`
- basic definition includes:
    - selector(required): Can take many forms, tag(`app-example`), attribute(`[app-example]`), class(`.app-example`).
    - template(required): Can be defined as single (template, templateUrl), or multiple(templateUrls).
    - styles(optional): Can be defined as styles([]), styleUrls
    - logic & data: Class with the business logic

```ts title="hello.component.ts"
import { Component } from "@angular/core";

@Component({
  selector: "app-hello",
  // selector: '[appHello]'
  // selector: '.appHello'
  templateUrls: [`./hello.component.html`],
  // template: `<h2>Hello, {{ name }}!</h2>`,
  // templateUrl: `./hello.component.html`,
  styleUrls: ["./hello.component.css"],
  // styles: ['h3 { color: "blue"}'],
})
export class HelloComponent {
  title = "Developer";
  name = "Ron";
  imgUrl = "https://Ron.image.facebook.com";

  handleClick() {
    console.log("Clicked!");
  }
}
```

```html title="other.component.html"
<!-- selector: 'app-hello' -->
<app-hello></app-hello>

<!-- selector: '[appHello]' -->
<div appHello></div>

<!-- selector: '.appHello' -->
<div class="appHello"></div>
```

### Data binding

- Data binding connects your component class (logic) and template (view) so they stay in sync.

```html title="hello.component.html"
<div>
  <!-- Interpolation -->
  <h1>{{ title }}</h1>

  <!-- property binding -->
  <img [src]="imgUrl" />

  <!-- Event Binding -->
  <button (click)="handleClick()">Click me</button>

  <!-- Two-way Binding -->
  <!-- any changes made to name in ts file will update input.value and vice-versa -->
  <input [(ngModel)]="name" />
  <p>Hello, {{ name }}</p>
</div>
```

| Type             | Direction            | Syntax                | Use Case                                        |
| ---------------- | -------------------- | --------------------- | ----------------------------------------------- |
| Interpolation    | Component → Template | `{{ value }}`         | Display data                                    |
| Property binding | Component → Template | `[property]="value"`  | Bind DOM property (e.g., `[src]`, `[disabled]`) |
| Event binding    | Template → Component | `(event)="handler"`   | Listen to events like click                     |
| Two-way binding  | Bi-directional       | `[(ngModel)]="value"` | Sync input fields                               |

#### Custom Two-way binding

- Custom two-way binding allows a parent component to bind to a child component’s property and get notified when it changes — using the `[(...)]` syntax.
- For this, Angular looks for:
    - `@Input() value`
    - `@Output() valueChange`
    - Together, these make `[(value)]` valid!

```ts title="counter.component.ts"
@Component({
  selector: 'app-counter',
  template: `
    <button (click)="decrement()">−</button>
    <span>{{ count }}</span>
    <button (click)="increment()">+</button>
  `
})
export class CounterComponent {
    @Input() count: number = 0;
    @Output() countChange = new EventEmitter<number>();

    increment() {
        this.count++;
        this.countChange.emit(this.count); // emit change
    }

    decrement() {
        this.count--;
        this.countChange.emit(this.count);
    }
}
```

```ts title="app.component.ts"
export class AppComponent {
    currentCount = 5;
}
```

```html title="app.component.html"
<h3>Two-way binding with custom component</h3>
<app-counter [(count)]="currentCount"></app-counter>
<p>Parent count value: {{ currentCount }}</p>
```

### Data flow

| Direction       | Mechanism        | Decorator                    |
| --------------- | ---------------- | ---------------------------- |
| Parent ➡️ Child | Property binding | `@Input()`                   |
| Child ➡️ Parent | Event binding    | `@Output()` + `EventEmitter` |

| Decorator   | Parameter Type | Default Behavior              | With Alias                  |
| ----------- | -------------- | ----------------------------- | --------------------------- |
| `@Input()`  | `string`       | Uses property name as binding | Allows external alias name  |
| `@Output()` | `string`       | Uses event name as binding    | Allows external event alias |

=== "Parent to Child "

    ```ts title="child.component.ts" hl_lines="5"
    import { Component, Input } from '@angular/core';

    @Component({
        selector: 'app-child',
        template: `<p>Received: {{ message }}</p>`
    })
    export class ChildComponent {
        @Input('alias') message: string = '';
    }
    ```

    ```html title="parent.component.html" hl_lines="1"
    <app-child [alias]="'Hello from Parent!'"></app-child>
    ```

=== "Child to Parent"

    ```ts title="child.component.ts" hl_lines="8 11"
    import { Component, Output, EventEmitter } from '@angular/core';

    @Component({
        selector: 'app-child',
        template: `<button (click)="sendData()">Send to Parent</button>`
    })
    export class ChildComponent {
        @Output('notifyParent') messageEvent = new EventEmitter<string>();

        sendData() {
            this.messageEvent.emit('Hello from Child!');
        }
    }
    ```

    ```html title="parent.component.html" hl_lines="1"
    <app-child (notifyParent)="receiveData($event)"></app-child>
    <p>{{ childMessage }}</p>
    ```

    ```ts title="parent.component.ts"
    export class ParentComponent {
        childMessage = '';

        receiveData(msg: string) {
            this.childMessage = msg;
        }
    }
    ```

=== "Sibling-to-Sibling"

    Fill stuff here

### Lifecycles

- When a component is created and managed by Angular, it goes through these key phases:
  Create → Render → Bind → Update → Destroy

| Hook | Trigger | Use |
| --- | --- | --- |
| `constructor()` | When the component class is instantiated | Inject dependencies only |
| `ngOnChanges()` | When `@Input()` properties change | React to `@Input()` changes |
| `ngOnInit()` | Once, after first `ngOnChanges()` | Init logic, API calls |
| `ngDoCheck()` | During every change detection cycle | Custom change detection logic |
| `ngAfterContentInit()` | After content `<ng-content>` is projected | Access projected content |
| `ngAfterContentChecked()` | Every check of projected content | Monitor `<ng-content>` changes |
| `ngAfterViewInit()` | After component's view and child views are initialized | Access child DOM or template refs |
| `ngAfterViewChecked()` | After every check of component's views | Handle post-render DOM logic |
| `ngOnDestroy()` | Right before the component is destroyed | Cleanup: unsubscribe, detach events, timers |

```ts title="lifecycle-demo.component.ts"
import { Component, OnInit, OnChanges, DoCheck, AfterContentInit, AfterContentChecked, AfterViewInit, AfterViewChecked, OnDestroy, Input, SimpleChanges } from "@angular/core";

@Component({
    selector: "app-lifecycle-demo",
    template: `
        <div class="child">
            <h3 #headerRef>Lifecycle Demo Component</h3>
            <p>Input: {{ inputData }}</p>
            <ng-content></ng-content>
        </div>
    `,
    styles: [`.child { border: 2px dashed #444; padding: 10px; margin-top: 10px; }`]
})
export class LifecycleDemoComponent implements OnInit, OnChanges, DoCheck, AfterContentInit, AfterContentChecked, AfterViewInit, AfterViewChecked, OnDestroy {
    @Input() inputData: string = "";

    @ContentChild('projectedContent', { static: false }) projectedParagraph!: ElementRef;
    @ViewChild('headerRef', { static: false }) header!: ElementRef;

    private oldValue = '';
    private data;

    constructor() {
        console.log('[constructor] Component instance created');
    }

    ngOnChanges(changes: SimpleChanges) {
        if (changes['inputData']) {
            const prev = changes['inputData'].previousValue;
            const curr = changes['inputData'].currentValue;
            console.log(`[ngOnChanges] inputData changed from "${prev}" to "${curr}"`);
        }
    }

    ngOnInit() {
        console.log('[ngOnInit] Component initialized');
        this.oldValue = this.inputData;
        fetch('http://example.com/data')
            .then((data: any): any => { this.data = data; });
    }

    ngDoCheck() {
        if (this.oldValue !== this.inputData) {
            console.log(`[ngDoCheck] Detected manual change from "${this.oldValue}" to "${this.inputData}"`);
            this.oldValue = this.inputData;
        }
    }

    ngAfterContentInit() {
        console.log('[ngAfterContentInit] Projected content initialized');
        if (this.projectedParagraph) {
            console.log(`Projected content: "${this.projectedParagraph.nativeElement.textContent.trim()}"`);
        }
    }

    ngAfterContentChecked() {
        console.log('[ngAfterContentChecked] Projected content checked');
    }

    ngAfterViewInit() {
        console.log('[ngAfterViewInit] View initialized');
        if (this.header) {
            console.log(`Header text: "${this.header.nativeElement.textContent.trim()}"`);
        }
    }

    ngAfterViewChecked() {
        console.log('[ngAfterViewChecked] View checked');
    }

    ngOnDestroy() {
        console.log('[ngOnDestroy] Component is about to be destroyed. Cleaning up...');
    }
}
```

``` html title="app.component.html"
<button (click)="toggle()">Toggle Component</button>
<button (click)="changeInput()">Change Input</button>

<app-lifecycle-demo *ngIf="showChild" [inputData]="parentData">
    <p #projectedContent>This is projected content</p>
</app-lifecycle-demo>

```

??? note "`ng-content`"

    - Insert whatever content the parent component provides here.
    - This is called content projection.

    ``` html 
    <!-- card.component.html -->
    <div class="card">
        <ng-content select="[card-title]"></ng-content>
        <ng-content select="[card-body]"></ng-content>
    </div>

    <!-- app.component.html -->
    <app-card>
        <h3 card-title>Title Here</h3>
        <p card-body>Body content here.</p>
    </app-card>
    ```



## Directives

- 2 types:
  - Structural: Change DOM structure. e.g `*ngIf`, `*ngFor`, `*ngSwitchCase`, custom ones
  - Attribute: Change appearance or behavior of elements. e.g `ngStyle`, `ngClass`, custom ones

=== "Structural"

    === "`*ngIf`"

        ``` html
        <div *ngIf="isLoggedIn; else loggedOutTemplate">Welcome back!</div>

        <ng-template #loggedOutTemplate>Please log in.</ng-template>

        <!-- Or explicit with then and else -->
        <ng-container *ngIf="isLoading; then loadingBlock; else contentBlock"></ng-container>

        <ng-template #loadingBlock><p>Loading...</p></ng-template>

        <ng-template #contentBlock><p>Data loaded!</p></ng-template>
        ```

    === "`*ngFor`"

        ``` html
        <ul>
            <li *ngFor="let item of items">{{ item }}</li>
        </ul>
        ```

    === "`*ngSwitch`"

        ``` html
        <div [ngSwitch]="status">
            <p *ngSwitchCase="'online'">Online</p>
            <p *ngSwitchCase="'offline'">Offline</p>
            <p *ngSwitchDefault>Unknown</p>
        </div>
        ```

    === "Custom"

        ``` ts title="unless.directive.ts"
        import { Directive, Input, TemplateRef, ViewContainerRef } from '@angular/core';

        @Directive({
            selector: '[appUnless]'
        })
        export class UnlessDirective {
            constructor(private tpl: TemplateRef<any>, private vcr: ViewContainerRef) {}

            @Input() set appUnless(condition: boolean) {
                if (!condition) {
                    this.vcr.createEmbeddedView(this.tpl);
                } else {
                    this.vcr.clear();
                }
            }
        }
        ```

        ```html title="example"
        <p *appUnless="isLoggedIn">You are not logged in.</p>
        ```

        | Feature            | Use Case                                    | Decorator         |
        | ------------------ | ------------------------------------------- | ----------------- |
        | `@Input()`         | Receive values from HTML                    | Property          |
        | `TemplateRef`      | Used in structural directives for templates | Constructor param |
        | `ViewContainerRef` | Add/remove templates dynamically            | Constructor param |

=== "Attribute"

    === "`ngClass`"

        ``` html
        <div [ngClass]="'class1 class2'"></div>

        <div [ngClass]="['class1', 'class2']"></div>

        <div [ngClass]="{ 'class1': isActive, 'class2': isPrimary }"></div>
        ```

    === "`ngStyle`"

        ``` html
        <p [ngStyle]="{
        'color': isError ? 'red' : 'black',
        'background-color': isHighlighted ? 'yellow' : 'transparent'
        }">Status</p>
        ```

    === "Custom"

        ``` ts title="highlight.directive.ts"
        import { Directive, ElementRef, HostListener, Input, Renderer2, HostBinding } from '@angular/core';

        @Directive({
            selector: '[appHighlight]'
        })
        export class HighlightDirective {
            // Accept input like [appHighlight]="'lightblue'"
            @Input('appHighlight') highlightColor: string = 'yellow';

            // Optional: bind class or style directly
            @HostBinding('style.border') border: string = '';

            constructor(private el: ElementRef, private renderer: Renderer2) {}

            @HostListener('mouseenter') onMouseEnter() {
                this.setBackground(this.highlightColor);
                this.border = '2px solid orange';
            }

            @HostListener('mouseleave') onMouseLeave() {
                this.setBackground('');
                this.border = '';
            }

            private setBackground(color: string) {
                this.renderer.setStyle(this.el.nativeElement, 'backgroundColor', color);
            }
        }
        ```

        ``` html title="example"
        <p [appHighlight]="'lightblue'">Custom highlight color</p>
        ```

        | Feature | Use Case | Decorator|
        | --- | --- | --- |
        | `@Input()` | Receive values from HTML | Property |
        | `ElementRef` | Direct access to DOM (Use: Read DOM properties) | Constructor param |
        | `Renderer2` | Safe DOM manipulation via Angular (Use: set style/class/attr/property) | Constructor param |
        | `@HostListener` | Listen to events on host element | Method |
        | `@HostBinding` | Bind class, style, or attribute | Property |


## Services

- An Angular service is a reusable class that holds logic/data that you want to share across components.
- Use cases:
    - Business logic (e.g. calculations)
    - API calls (e.g. HttpClient)
    - Shared state (e.g. a logged-in user)


```ts
// message.service.ts
import { Injectable } from '@angular/core';
import { BehaviorSubject } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class MessageService {
    private messageSource = new BehaviorSubject<string>('Initial message');
    message$ = this.messageSource.asObservable();

    updateMessage(newMessage: string) {
        this.messageSource.next(newMessage); // Push update to all subscribers
    }
}

// message-display.component.ts
@Component({
    selector: 'app-message-display',
    template: `<p>Message: {{ message }}</p>`
})
export class MessageDisplayComponent implements OnInit {
    message = '';

    constructor(private messageService: MessageService) {}

    ngOnInit() {
        this.messageService.message$.subscribe((msg) => {
            this.message = msg;
        });
    }
}

// message-control.component.ts
@Component({
    selector: 'app-message-control',
    template: `<button (click)="changeMessage()">Change Message</button>`
})
export class MessageControlComponent {
    constructor(private messageService: MessageService) {}

    changeMessage() {
        const newMsg = 'Updated at ' + new Date().toLocaleTimeString();
        this.messageService.updateMessage(newMsg);
    }
}
```

### Scopes

| Scope               | How You Provide It                    | Instance Shared Across           |
| ------------------- | ------------------------------------- | -------------------------------- |
| **App-wide**        | `@Injectable({ providedIn: 'root' })` | Entire application (singleton)   |
| **Module-level**    | `providers: [MyService]` in NgModule  | That specific feature module     |
| **Component-level** | `providers: [MyService]` in component | Only that component and children |


```ts
// App wide
@Injectable({ providedIn: 'root' })
export class GlobalService { ... }

// scope: specific Module
@NgModule({
    providers: [FeatureService]
})
export class SomeFeatureModule {}

// scope: Component
@Component({
    // ...
    providers: [UserService]
})
export class UserComponent { }
```


## NgModules

- An NgModule is a class decorated with @NgModule that defines:
    - Which components, directives, and pipes it owns
    - What other modules it depends on
    - Which components it exposes to other modules

```ts title="my-feature.module.ts"
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { MyComponent } from './my.component';

@NgModule({
  declarations: [MyComponent],     // Components, directives, pipes owned
  imports: [CommonModule],         // Other modules used inside
  exports: [MyComponent],          // Exposed to other modules
  providers: []                    // Services scoped to this module
})
export class MyFeatureModule {}
```