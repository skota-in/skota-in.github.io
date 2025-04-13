# Integrating Angular with NgRx: Step-by-Step Guide

*Published on June 10, 2023*

## Introduction

NgRx is a powerful state management library for Angular applications, inspired by Redux. It helps you manage application state in a predictable way by enforcing a unidirectional data flow and immutable state updates. This guide will walk you through the process of integrating NgRx into an Angular application, from installation to implementation of core concepts.

## Table of Contents

1. [Understanding NgRx and State Management](#understanding-ngrx-and-state-management)
2. [Prerequisites](#prerequisites)
3. [Setting Up a New Angular Project](#setting-up-a-new-angular-project)
4. [Installing NgRx](#installing-ngrx)
5. [Core NgRx Concepts](#core-ngrx-concepts)
6. [Implementing NgRx in an Angular Application](#implementing-ngrx-in-an-angular-application)
    - [Step 1: Define the State Interface](#step-1-define-the-state-interface)
    - [Step 2: Create Actions](#step-2-create-actions)
    - [Step 3: Create Reducers](#step-3-create-reducers)
    - [Step 4: Create Effects (Optional)](#step-4-create-effects-optional)
    - [Step 5: Create Selectors](#step-5-create-selectors)
    - [Step 6: Set Up the Store Module](#step-6-set-up-the-store-module)
    - [Step 7: Use the Store in Components](#step-7-use-the-store-in-components)
7. [Advanced NgRx Features](#advanced-ngrx-features)
8. [Best Practices](#best-practices)
9. [Common Pitfalls and Solutions](#common-pitfalls-and-solutions)
10. [Performance Considerations](#performance-considerations)
11. [Testing NgRx](#testing-ngrx)
12. [Conclusion](#conclusion)

## Understanding NgRx and State Management

Before diving into implementation, it's important to understand why state management is necessary and how NgRx helps:

- **State Management**: As applications grow, managing state across components becomes challenging. NgRx provides a centralized store for all application state.
- **Predictability**: NgRx enforces a unidirectional data flow, making state changes predictable and easier to debug.
- **Immutability**: State is never directly modified, ensuring consistency and enabling features like time-travel debugging.
- **Performance**: NgRx helps optimize performance through features like memoized selectors.

## Prerequisites

Before starting, ensure you have:

- Node.js (v14 or later) and npm installed
- Angular CLI installed (`npm install -g @angular/cli`)
- Basic knowledge of Angular and TypeScript
- Understanding of reactive programming concepts (RxJS)

## Setting Up a New Angular Project

If you don't have an existing project, create a new Angular application:

```bash
ng new my-ngrx-app
cd my-ngrx-app
```

## Installing NgRx

Install the required NgRx packages:

```bash
ng add @ngrx/store
ng add @ngrx/effects
ng add @ngrx/entity
ng add @ngrx/store-devtools
ng add @ngrx/router-store
```

Alternatively, you can install them all at once:

```bash
npm install @ngrx/store @ngrx/effects @ngrx/entity @ngrx/store-devtools @ngrx/router-store --save
```

## Core NgRx Concepts

Before implementation, let's understand the core NgRx concepts:

1. **Store**: A centralized state container that holds the application state.
2. **Actions**: Events that describe state changes. They have a type and optional payload.
3. **Reducers**: Pure functions that take the current state and an action, and return a new state.
4. **Selectors**: Functions that extract specific pieces of state from the store.
5. **Effects**: Handle side effects like API calls, and can dispatch new actions based on the results.
6. **Entities**: A pattern for managing collections of entities in the store.

## Implementing NgRx in an Angular Application

Let's implement a simple todo list application with NgRx:

### Step 1: Define the State Interface

First, define what your application state will look like:

```typescript
// src/app/store/state/todo.state.ts
export interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

export interface TodoState {
  todos: Todo[];
  loading: boolean;
  error: string | null;
}

export const initialTodoState: TodoState = {
  todos: [],
  loading: false,
  error: null
};
```

### Step 2: Create Actions

Define actions that can modify the state:

```typescript
// src/app/store/actions/todo.actions.ts
import { createAction, props } from '@ngrx/store';
import { Todo } from '../state/todo.state';

export const loadTodos = createAction('[Todo] Load Todos');
export const loadTodosSuccess = createAction(
  '[Todo] Load Todos Success',
  props<{ todos: Todo[] }>()
);
export const loadTodosFailure = createAction(
  '[Todo] Load Todos Failure',
  props<{ error: string }>()
);

export const addTodo = createAction(
  '[Todo] Add Todo',
  props<{ text: string }>()
);
export const addTodoSuccess = createAction(
  '[Todo] Add Todo Success',
  props<{ todo: Todo }>()
);
export const addTodoFailure = createAction(
  '[Todo] Add Todo Failure',
  props<{ error: string }>()
);

export const toggleTodo = createAction(
  '[Todo] Toggle Todo',
  props<{ id: number }>()
);
```

### Step 3: Create Reducers

Create reducers to handle state changes based on actions:

```typescript
// src/app/store/reducers/todo.reducer.ts
import { createReducer, on } from '@ngrx/store';
import { initialTodoState } from '../state/todo.state';
import * as TodoActions from '../actions/todo.actions';

export const todoReducer = createReducer(
  initialTodoState,
  on(TodoActions.loadTodos, state => ({
    ...state,
    loading: true,
    error: null
  })),
  on(TodoActions.loadTodosSuccess, (state, { todos }) => ({
    ...state,
    todos,
    loading: false
  })),
  on(TodoActions.loadTodosFailure, (state, { error }) => ({
    ...state,
    error,
    loading: false
  })),
  on(TodoActions.addTodoSuccess, (state, { todo }) => ({
    ...state,
    todos: [...state.todos, todo]
  })),
  on(TodoActions.toggleTodo, (state, { id }) => ({
    ...state,
    todos: state.todos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    )
  }))
);
```

### Step 4: Create Effects (Optional)

Effects handle side effects like API calls:

```typescript
// src/app/store/effects/todo.effects.ts
import { Injectable } from '@angular/core';
import { Actions, createEffect, ofType } from '@ngrx/effects';
import { of } from 'rxjs';
import { catchError, map, mergeMap } from 'rxjs/operators';
import { TodoService } from '../../services/todo.service';
import * as TodoActions from '../actions/todo.actions';

@Injectable()
export class TodoEffects {
  loadTodos$ = createEffect(() =>
    this.actions$.pipe(
      ofType(TodoActions.loadTodos),
      mergeMap(() =>
        this.todoService.getTodos().pipe(
          map(todos => TodoActions.loadTodosSuccess({ todos })),
          catchError(error =>
            of(TodoActions.loadTodosFailure({ error: error.message }))
          )
        )
      )
    )
  );

  addTodo$ = createEffect(() =>
    this.actions$.pipe(
      ofType(TodoActions.addTodo),
      mergeMap(({ text }) =>
        this.todoService.addTodo(text).pipe(
          map(todo => TodoActions.addTodoSuccess({ todo })),
          catchError(error =>
            of(TodoActions.addTodoFailure({ error: error.message }))
          )
        )
      )
    )
  );

  constructor(
    private actions$: Actions,
    private todoService: TodoService
  ) {}
}
```

For this to work, you'll need a TodoService:

```typescript
// src/app/services/todo.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { Todo } from '../store/state/todo.state';

@Injectable({
  providedIn: 'root'
})
export class TodoService {
  private apiUrl = 'api/todos'; // Replace with your API URL

  constructor(private http: HttpClient) {}

  getTodos(): Observable<Todo[]> {
    return this.http.get<Todo[]>(this.apiUrl);
  }

  addTodo(text: string): Observable<Todo> {
    const todo: Partial<Todo> = {
      text,
      completed: false
    };
    return this.http.post<Todo>(this.apiUrl, todo);
  }

  toggleTodo(id: number): Observable<Todo> {
    return this.http.patch<Todo>(`${this.apiUrl}/${id}`, {});
  }
}
```

### Step 5: Create Selectors

Create selectors to extract specific pieces of state:

```typescript
// src/app/store/selectors/todo.selectors.ts
import { createFeatureSelector, createSelector } from '@ngrx/store';
import { TodoState } from '../state/todo.state';

export const selectTodoState = createFeatureSelector<TodoState>('todos');

export const selectAllTodos = createSelector(
  selectTodoState,
  state => state.todos
);

export const selectTodosLoading = createSelector(
  selectTodoState,
  state => state.loading
);

export const selectTodosError = createSelector(
  selectTodoState,
  state => state.error
);

export const selectCompletedTodos = createSelector(
  selectAllTodos,
  todos => todos.filter(todo => todo.completed)
);

export const selectIncompleteTodos = createSelector(
  selectAllTodos,
  todos => todos.filter(todo => !todo.completed)
);
```

### Step 6: Set Up the Store Module

Register the reducers and effects in your app module:

```typescript
// src/app/app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { HttpClientModule } from '@angular/common/http';
import { StoreModule } from '@ngrx/store';
import { EffectsModule } from '@ngrx/effects';
import { StoreDevtoolsModule } from '@ngrx/store-devtools';
import { environment } from '../environments/environment';

import { AppComponent } from './app.component';
import { todoReducer } from './store/reducers/todo.reducer';
import { TodoEffects } from './store/effects/todo.effects';

@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule,
    HttpClientModule,
    StoreModule.forRoot({ todos: todoReducer }),
    EffectsModule.forRoot([TodoEffects]),
    StoreDevtoolsModule.instrument({
      maxAge: 25, // Retains last 25 states
      logOnly: environment.production
    })
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

### Step 7: Use the Store in Components

Now you can use the store in your components:

```typescript
// src/app/todo-list/todo-list.component.ts
import { Component, OnInit } from '@angular/core';
import { Store } from '@ngrx/store';
import { Observable } from 'rxjs';
import { Todo } from '../store/state/todo.state';
import * as TodoActions from '../store/actions/todo.actions';
import * as TodoSelectors from '../store/selectors/todo.selectors';

@Component({
  selector: 'app-todo-list',
  template: `
    <div *ngIf="loading$ | async">Loading...</div>
    <div *ngIf="error$ | async as error" class="error">{{ error }}</div>
    
    <form (ngSubmit)="addTodo()">
      <input [(ngModel)]="newTodoText" name="newTodo" placeholder="Add a new todo">
      <button type="submit">Add</button>
    </form>
    
    <ul>
      <li *ngFor="let todo of todos$ | async" 
          [class.completed]="todo.completed"
          (click)="toggleTodo(todo.id)">
        {{ todo.text }}
      </li>
    </ul>
    
    <div>
      <p>Completed: {{ (completedTodos$ | async)?.length || 0 }}</p>
      <p>Incomplete: {{ (incompleteTodos$ | async)?.length || 0 }}</p>
    </div>
  `,
  styles: [`
    .completed {
      text-decoration: line-through;
      color: gray;
    }
    .error {
      color: red;
    }
  `]
})
export class TodoListComponent implements OnInit {
  todos$: Observable<Todo[]>;
  loading$: Observable<boolean>;
  error$: Observable<string | null>;
  completedTodos$: Observable<Todo[]>;
  incompleteTodos$: Observable<Todo[]>;
  
  newTodoText = '';

  constructor(private store: Store) {
    this.todos$ = this.store.select(TodoSelectors.selectAllTodos);
    this.loading$ = this.store.select(TodoSelectors.selectTodosLoading);
    this.error$ = this.store.select(TodoSelectors.selectTodosError);
    this.completedTodos$ = this.store.select(TodoSelectors.selectCompletedTodos);
    this.incompleteTodos$ = this.store.select(TodoSelectors.selectIncompleteTodos);
  }

  ngOnInit(): void {
    this.store.dispatch(TodoActions.loadTodos());
  }

  addTodo(): void {
    if (this.newTodoText.trim()) {
      this.store.dispatch(TodoActions.addTodo({ text: this.newTodoText }));
      this.newTodoText = '';
    }
  }

  toggleTodo(id: number): void {
    this.store.dispatch(TodoActions.toggleTodo({ id }));
  }
}
```

Don't forget to add the component to your app module:

```typescript
// src/app/app.module.ts
import { FormsModule } from '@angular/forms';
import { TodoListComponent } from './todo-list/todo-list.component';

@NgModule({
  declarations: [
    AppComponent,
    TodoListComponent
  ],
  imports: [
    // ... other imports
    FormsModule
  ],
  // ...
})
export class AppModule {}
```

## Advanced NgRx Features

### Using NgRx Entity

NgRx Entity helps manage collections of entities with less boilerplate:

```typescript
// src/app/store/state/todo.state.ts
import { EntityState, createEntityAdapter } from '@ngrx/entity';

export interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

export const todoAdapter = createEntityAdapter<Todo>();

export interface TodoState extends EntityState<Todo> {
  loading: boolean;
  error: string | null;
}

export const initialTodoState: TodoState = todoAdapter.getInitialState({
  loading: false,
  error: null
});
```

```typescript
// src/app/store/reducers/todo.reducer.ts
import { createReducer, on } from '@ngrx/store';
import { todoAdapter, initialTodoState } from '../state/todo.state';
import * as TodoActions from '../actions/todo.actions';

export const todoReducer = createReducer(
  initialTodoState,
  on(TodoActions.loadTodos, state => ({
    ...state,
    loading: true,
    error: null
  })),
  on(TodoActions.loadTodosSuccess, (state, { todos }) => 
    todoAdapter.setAll(todos, { ...state, loading: false })
  ),
  on(TodoActions.loadTodosFailure, (state, { error }) => ({
    ...state,
    error,
    loading: false
  })),
  on(TodoActions.addTodoSuccess, (state, { todo }) => 
    todoAdapter.addOne(todo, state)
  ),
  on(TodoActions.toggleTodo, (state, { id }) => {
    const todo = state.entities[id];
    if (!todo) return state;
    return todoAdapter.updateOne(
      { id, changes: { completed: !todo.completed } },
      state
    );
  })
);
```

```typescript
// src/app/store/selectors/todo.selectors.ts
import { createFeatureSelector, createSelector } from '@ngrx/store';
import { todoAdapter, TodoState } from '../state/todo.state';

export const selectTodoState = createFeatureSelector<TodoState>('todos');

export const {
  selectAll: selectAllTodos,
  selectEntities: selectTodoEntities,
  selectIds: selectTodoIds,
  selectTotal: selectTotalTodos
} = todoAdapter.getSelectors(selectTodoState);

export const selectTodosLoading = createSelector(
  selectTodoState,
  state => state.loading
);

export const selectTodosError = createSelector(
  selectTodoState,
  state => state.error
);

export const selectCompletedTodos = createSelector(
  selectAllTodos,
  todos => todos.filter(todo => todo.completed)
);

export const selectIncompleteTodos = createSelector(
  selectAllTodos,
  todos => todos.filter(todo => !todo.completed)
);
```

### Using NgRx Router Store

NgRx Router Store connects the Angular Router to the NgRx store:

```typescript
// src/app/app.module.ts
import { StoreRouterConnectingModule, routerReducer } from '@ngrx/router-store';

@NgModule({
  imports: [
    // ... other imports
    StoreModule.forRoot({
      todos: todoReducer,
      router: routerReducer
    }),
    StoreRouterConnectingModule.forRoot()
  ],
  // ...
})
export class AppModule {}
```

You can then create custom router serializers and selectors to access route information from the store.

## Best Practices

1. **State Structure**: Keep your state normalized and flat. Use NgRx Entity for collections.
2. **Action Naming**: Use a consistent naming convention for actions, like `[Source] Event`.
3. **Selectors**: Create reusable selectors and use memoization for performance.
4. **Effects**: Keep components free of side effects by handling them in NgRx Effects.
5. **Immutability**: Never mutate state directly. Use the spread operator or libraries like Immer.
6. **State Access**: Access state only through selectors, not directly from the store.
7. **Action Creators**: Use the `createAction` function to create type-safe actions.
8. **Feature Modules**: Use feature modules to organize state by feature.

## Common Pitfalls and Solutions

1. **Over-engineering**: Don't use NgRx for simple applications or components with local state.
2. **Boilerplate**: Use NgRx creator functions and Entity to reduce boilerplate.
3. **Performance**: Use memoized selectors and avoid unnecessary state updates.
4. **Debugging**: Use Redux DevTools to debug state changes.
5. **Side Effects**: Handle all side effects in Effects, not in components or services.
6. **State Updates**: Remember that state updates are immutable. Create a new state object.

## Performance Considerations

1. **Memoized Selectors**: Use createSelector to create memoized selectors that only recompute when inputs change.
2. **OnPush Change Detection**: Use OnPush change detection with NgRx for better performance.
3. **Normalized State**: Keep your state normalized to avoid duplication and improve performance.
4. **Lazy Loading**: Use NgRx with lazy-loaded modules for better initial load times.

## Testing NgRx

### Testing Reducers

```typescript
// src/app/store/reducers/todo.reducer.spec.ts
import { todoReducer } from './todo.reducer';
import { initialTodoState } from '../state/todo.state';
import * as TodoActions from '../actions/todo.actions';

describe('Todo Reducer', () => {
  it('should return the default state', () => {
    const action = { type: 'NOOP' } as any;
    const state = todoReducer(undefined, action);
    
    expect(state).toBe(initialTodoState);
  });
  
  it('should toggle a todo', () => {
    const todo = { id: 1, text: 'Test', completed: false };
    const initialState = {
      ...initialTodoState,
      todos: [todo]
    };
    
    const action = TodoActions.toggleTodo({ id: 1 });
    const state = todoReducer(initialState, action);
    
    expect(state.todos[0].completed).toBe(true);
  });
});
```

### Testing Selectors

```typescript
// src/app/store/selectors/todo.selectors.spec.ts
import * as fromSelectors from './todo.selectors';
import { TodoState } from '../state/todo.state';

describe('Todo Selectors', () => {
  const initialState: TodoState = {
    todos: [
      { id: 1, text: 'Test 1', completed: false },
      { id: 2, text: 'Test 2', completed: true }
    ],
    loading: false,
    error: null
  };
  
  it('should select all todos', () => {
    const result = fromSelectors.selectAllTodos.projector(initialState);
    expect(result.length).toBe(2);
  });
  
  it('should select completed todos', () => {
    const todos = initialState.todos;
    const result = fromSelectors.selectCompletedTodos.projector(todos);
    expect(result.length).toBe(1);
    expect(result[0].id).toBe(2);
  });
});
```

### Testing Effects

```typescript
// src/app/store/effects/todo.effects.spec.ts
import { TestBed } from '@angular/core/testing';
import { provideMockActions } from '@ngrx/effects/testing';
import { Observable, of, throwError } from 'rxjs';
import { TodoEffects } from './todo.effects';
import { TodoService } from '../../services/todo.service';
import * as TodoActions from '../actions/todo.actions';

describe('TodoEffects', () => {
  let actions$: Observable<any>;
  let effects: TodoEffects;
  let todoService: jasmine.SpyObj<TodoService>;
  
  beforeEach(() => {
    const spy = jasmine.createSpyObj('TodoService', ['getTodos', 'addTodo']);
    
    TestBed.configureTestingModule({
      providers: [
        TodoEffects,
        provideMockActions(() => actions$),
        { provide: TodoService, useValue: spy }
      ]
    });
    
    effects = TestBed.inject(TodoEffects);
    todoService = TestBed.inject(TodoService) as jasmine.SpyObj<TodoService>;
  });
  
  it('should load todos successfully', () => {
    const todos = [{ id: 1, text: 'Test', completed: false }];
    actions$ = of(TodoActions.loadTodos());
    todoService.getTodos.and.returnValue(of(todos));
    
    effects.loadTodos$.subscribe(action => {
      expect(action).toEqual(TodoActions.loadTodosSuccess({ todos }));
    });
  });
  
  it('should handle errors when loading todos', () => {
    const error = 'Error loading todos';
    actions$ = of(TodoActions.loadTodos());
    todoService.getTodos.and.returnValue(throwError(() => new Error(error)));
    
    effects.loadTodos$.subscribe(action => {
      expect(action).toEqual(TodoActions.loadTodosFailure({ error }));
    });
  });
});
```

## Conclusion

Integrating NgRx with Angular provides a robust solution for state management in complex applications. By following the steps outlined in this guide, you can implement a predictable, maintainable state management system that scales with your application.

Remember that NgRx introduces complexity, so evaluate whether your application needs it. For simpler applications, consider alternatives like RxJS with services or the Angular Component Store.

As you become more familiar with NgRx, explore advanced features like Entity, Router Store, and Meta-Reducers to further enhance your state management capabilities.

## References

- [NgRx Documentation](https://ngrx.io/)
- [Angular Documentation](https://angular.io/docs)
- [RxJS Documentation](https://rxjs.dev/)
- [Redux DevTools](https://github.com/reduxjs/redux-devtools)
- [NgRx Example Application](https://github.com/ngrx/platform/tree/master/projects/example-app)
