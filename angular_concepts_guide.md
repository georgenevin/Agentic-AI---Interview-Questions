# Angular Q&A Reference Guide

## 1. Fundamentals & Core Architecture

### Q: What is Angular?
**A:** Angular is an open-source, component-based frontend framework used for building Single Page Applications (SPAs).

### Q: What is a component?
**A:** A component is a reusable building block that contains the logic, view (HTML), and styling (CSS) used to create a user interface in an Angular application.

### Q: What is a Directive?
**A:** A Directive is a class annotated with `@Directive` that adds custom behavior, extends functionality, or transforms elements in the Document Object Model (DOM). There are three types of directives in Angular:
* **Component Directives:** Directives with a template (`@Component`).
* **Structural Directives:** Change the DOM layout by adding or removing elements (prefixed with `*`, e.g., `*ngIf`, `*ngFor`, `ngSwitch`).
* **Attribute Directives:** Change the appearance or behavior of an existing element (e.g., `ngClass`, `ngStyle`, or custom hover directives).

### Q: What is a Single Page Application (SPA)?
**A:** A Single Page Application loads a single HTML page and dynamically updates content as the user interacts with the app, without requiring a full page reload from the server.

### Q: What are the main advantages of Angular?
**A:**
* **Component-based architecture** for modular, reusable code
* **Two-way data binding** for real-time synchronization between logic and view
* **Directives** to extend HTML capabilities
* **Powerful CLI (Command Line Interface)** for generation and builds
* **Built-in Dependency Injection (DI)** for clean architecture
* **Built-in Routing** for SPA navigation
* **Open source** with strong community support

---

## 2. Project Structure & Build Process

### Q: What is npm and what are `package.json` and `node_modules`?
**A:** 
* **npm (Node Package Manager):** A tool used to install and manage project dependencies. It connects to the npm registry, an online repository containing thousands of packages.
* **`package.json`:** A file that contains metadata about the project, scripts to run/build the app, and lists the required package dependencies.
* **`node_modules/`:** The directory where npm installs all the package dependencies listed in `package.json`.

### Q: What are the `public` and `src` folders?
**A:** 
* **`public/`:** Contains static files (e.g., images, icons) served directly without processing.
* **`src/`:** The main working directory for developers, containing the core TypeScript, HTML, and CSS source code.

### Q: What is `angular.json`?
**A:** The configuration file that tells the Angular CLI how to structure, run, test, and build your project.

### Q: How does an Angular application load and boot up?
**A:**
1. **`index.html`:** The primary HTML page hosted by the server. It contains the `<app-root>` custom tag.
2. **`main.ts`:** The entry-point script that bootstraps (initializes) the application by launching the root component (`AppComponent`).
3. **`ng serve` process:** The Angular CLI compiles `main.ts` into a JavaScript file (`main.js`) and dynamically injects it into `index.html`.
4. **App Rendering:** `AppComponent` loads into the `<app-root>` tag, replacing it with the actual HTML/CSS defined in `app.component.html` and `app.component.css`. All other components act as children to this root component.

### Q: What are common Angular CLI commands?
**A:**
* **`ng serve`:** Compiles the application and starts a local development server with live-reload enabled.
* **`ng generate component <name>`:** Generates a new component along with its HTML, CSS, TypeScript, and test files.

---

## 3. Decorators & Modules

### Q: What is a decorator in Angular?
**A:** A decorator is a special function prefixed with `@` that attaches metadata to a class, property, method, or parameter, telling Angular how to process it.

### Q: What are the main types of decorators?
**A:**
* **Class Decorators:** `@Component`, `@NgModule`, `@Directive`, `@Pipe`
* **Property Decorators:** `@Input`, `@Output`
* **Method Decorators:** `@HostListener`
* **Parameter Decorators:** `@Inject`, `@Self`

### Q: What is an NgModule?
**A:** An `@NgModule` is a class annotated with `@NgModule` that groups related components, directives, pipes, and services into a cohesive functional unit.

---

## 4. Data Binding & Directives Usage

### Q: What is data binding in Angular?
**A:** Data binding is the mechanism for synchronization and communication between the component's TypeScript logic and its HTML template view.

### Q: What are the main types of data binding?
**A:**
* **Interpolation `{{ value }}`:** One-way binding from component logic to the template view.
* **Property Binding `[property]="value"`:** One-way binding from component logic to an HTML element property using square brackets.
* **Event Binding `(event)="handler()"`:** One-way binding from the view to the component logic using round brackets (listens for user actions like clicks or input).
* **Two-Way Data Binding `[(ngModel)]="value"`:** Keeps the view and component logic instantly synchronized in both directions.

### Q: What are structural directives and common examples?
**A:** Structural directives alter the layout of the DOM by adding or removing elements:
* **`*ngIf`:** Conditionally adds or removes DOM elements based on a boolean condition.
* **`*ngFor`:** Iterates through a collection/list and renders a template for each item.
* **`ngSwitch` (`*ngSwitchCase`):** Evaluates an expression and displays the matching conditional template.

---

## 5. Lifecycle Hooks

### Q: What is a component lifecycle hook?
**A:** A lifecycle hook is a predefined method that Angular executes at specific stages during a component's lifecycle—from creation to destruction.

### Q: What are the common lifecycle hooks and their execution order?
**A:**
1. **`constructor()`:** Default TypeScript class constructor executed when the class is instantiated. (Not an Angular lifecycle hook, but runs first).
2. **`ngOnChanges()`:** Called before `ngOnInit()` and whenever an `@Input()` bound property changes.
3. **`ngOnInit()`:** Executed once after Angular initializes the component's data-bound properties. Used for initialization logic and initial data fetching.
4. **`ngDoCheck()`:** Executed during every change detection run, allowing custom change detection.
5. **`ngOnDestroy()`:** Executed right before Angular destroys the component. Used for cleanup (e.g., unsubscribing from Observables).

---

## 6. Services & Dependency Injection

### Q: What is an Angular Service?
**A:** A Service is a reusable TypeScript class containing business logic, data management, or API calls that can be shared across multiple components.

### Q: What is Dependency Injection (DI)?
**A:** Dependency Injection is a design pattern in which Angular automatically provides instances of services to components or other services that request them, rather than creating them manually inside the class.

### Q: What is the `@Injectable()` decorator?
**A:** `@Injectable()` marks a class as available to be provided and injected via Angular's dependency injection system. It also enables other services to be injected into it.

---

## 7. Asynchronous Programming (RxJS, Observables, Promises)

### Q: How do Promises and Observables handle asynchronous data?
**A:**
* **Promise:** Resolves a single asynchronous value or error at a time. If fetching 1 million records, a Promise waits until all 1 million records are ready before returning them as a single payload.
* **Observable:** Streams chunks of data continuously over time. Data can be processed piece-by-piece as it arrives rather than waiting for the entire dataset.

### Q: What is RxJS?
**A:** RxJS (Reactive Extensions for JavaScript) is a library for composing asynchronous and event-based programs using observable sequences and operators.

### Q: Are Observables executed automatically?
**A:** No, Observables are lazy. They will not emit values or execute work until you explicitly subscribe to them using the `.subscribe()` method.

### Q: What is `HttpClient`?
**A:** `HttpClient` is an Angular service used to make HTTP requests (GET, POST, PUT, DELETE) to backend APIs, returning RxJS Observables as responses.

---

## 8. Miscellaneous & Utility Concepts

### Q: What are Pipes and Pipe Chaining?
**A:** 
* **Pipe:** A function that accepts input data, transforms it, and returns a formatted display value in the template (e.g., date formats, currency).
* **Pipe Chaining:** Passing the output of one pipe into another pipe sequentially (e.g., `{{ birthday | date | uppercase }}`).

### Q: What are `import` and `export` in TypeScript?
**A:**
* **`export`:** Makes a class, function, or variable accessible to other files.
* **`import`:** Loads and consumes an exported class, function, or variable from another file.

### Q: What is `any` in TypeScript?
**A:** `any` is a data type that disables static type-checking for a variable, allowing it to hold any type of value.

### Q: What is an arrow function?
**A:** An arrow function (`() => {}`) is a shorthand syntax for defining functions in JavaScript/TypeScript, preserving the lexical scope of `this`.
