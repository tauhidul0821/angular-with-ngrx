# Implementing NgRx with Angular for Users API

- [youtube link](https://www.youtube.com/watch?v=zM6pUAaJZQM)

## **Step 1: Install NgRx Packages**
Run the following command in your Angular project:
```sh
ng add @ngrx/store @ngrx/effects @ngrx/entity @ngrx/store-devtools
```

---

## **Step 2: Create User Model**
Create a model for your user entity in `src/app/models/user.model.ts`:
```typescript
export interface User {
  id: string;
  name: string;
  email: string;
}
```

---

## **Step 3: Define Actions**
Create an actions file `src/app/store/users/users.actions.ts`:
```typescript
import { createAction, props } from '@ngrx/store';
import { User } from '../../models/user.model';

// Load Users
export const loadUsers = createAction('[Users] Load Users');
export const loadUsersSuccess = createAction(
  '[Users] Load Users Success',
  props<{ users: User[] }>()
);
export const loadUsersFailure = createAction(
  '[Users] Load Users Failure',
  props<{ error: string }>()
);

// Add User
export const addUser = createAction('[Users] Add User', props<{ user: User }>());
export const addUserSuccess = createAction(
  '[Users] Add User Success',
  props<{ user: User }>()
);
export const addUserFailure = createAction(
  '[Users] Add User Failure',
  props<{ error: string }>()
);

// Update User
export const updateUser = createAction(
  '[Users] Update User',
  props<{ user: User }>()
);
export const updateUserSuccess = createAction(
  '[Users] Update User Success',
  props<{ user: User }>()
);
export const updateUserFailure = createAction(
  '[Users] Update User Failure',
  props<{ error: string }>()
);

// Delete User
export const deleteUser = createAction('[Users] Delete User', props<{ id: string }>());
export const deleteUserSuccess = createAction(
  '[Users] Delete User Success',
  props<{ id: string }>()
);
export const deleteUserFailure = createAction(
  '[Users] Delete User Failure',
  props<{ error: string }>()
);
```

---

## **Step 4: Create Reducer**
Create a reducer in `src/app/store/users/users.reducer.ts`:
```typescript
import { createReducer, on } from '@ngrx/store';
import { User } from '../../models/user.model';
import * as UsersActions from './users.actions';

export interface UsersState {
  users: User[];
  loading: boolean;
  error: string | null;
}

export const initialState: UsersState = {
  users: [],
  loading: false,
  error: null,
};

export const usersReducer = createReducer(
  initialState,
  on(UsersActions.loadUsers, (state) => ({ ...state, loading: true })),
  on(UsersActions.loadUsersSuccess, (state, { users }) => ({
    ...state,
    users,
    loading: false,
  })),
  on(UsersActions.loadUsersFailure, (state, { error }) => ({
    ...state,
    loading: false,
    error,
  })),
  on(UsersActions.addUserSuccess, (state, { user }) => ({
    ...state,
    users: [...state.users, user],
  })),
  on(UsersActions.updateUserSuccess, (state, { user }) => ({
    ...state,
    users: state.users.map((u) => (u.id === user.id ? user : u)),
  })),
  on(UsersActions.deleteUserSuccess, (state, { id }) => ({
    ...state,
    users: state.users.filter((user) => user.id !== id),
  }))
);
```

---

## **Step 5: Create Effects**
Create effects to handle API calls in `src/app/store/users/users.effects.ts`:
```typescript
import { Injectable } from '@angular/core';
import { Actions, createEffect, ofType } from '@ngrx/effects';
import { catchError, map, mergeMap, of } from 'rxjs';
import { UsersService } from '../../services/users.service';
import * as UsersActions from './users.actions';

@Injectable()
export class UsersEffects {
  constructor(private actions$: Actions, private usersService: UsersService) {}

  loadUsers$ = createEffect(() =>
    this.actions$.pipe(
      ofType(UsersActions.loadUsers),
      mergeMap(() =>
        this.usersService.getUsers().pipe(
          map((users) => UsersActions.loadUsersSuccess({ users })),
          catchError((error) => of(UsersActions.loadUsersFailure({ error })))
        )
      )
    )
  );
}
```

---

## **Step 6: Create User Service**
Create `src/app/services/users.service.ts`:
```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { User } from '../models/user.model';

@Injectable({
  providedIn: 'root',
})
export class UsersService {
  private apiUrl = '/api/users';

  constructor(private http: HttpClient) {}

  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(this.apiUrl);
  }
}
```

---

## **Step 7: Register Store in App Module**
Update `src/app/app.module.ts`:
```typescript
import { StoreModule } from '@ngrx/store';
import { EffectsModule } from '@ngrx/effects';
import { usersReducer } from './store/users/users.reducer';
import { UsersEffects } from './store/users/users.effects';

@NgModule({
  imports: [
    StoreModule.forRoot({ users: usersReducer }),
    EffectsModule.forRoot([UsersEffects])
  ]
})
export class AppModule {}
```

---

## **Step 8: Select Data in a Component**
Use NgRx selectors in `src/app/users.component.ts`:
```typescript
import { Component } from '@angular/core';
import { Store } from '@ngrx/store';
import { Observable } from 'rxjs';
import { User } from './models/user.model';
import * as UsersActions from './store/users/users.actions';

@Component({
  selector: 'app-users',
  templateUrl: './users.component.html',
})
export class UsersComponent {
  users$: Observable<User[]> = this.store.select((state) => state.users.users);

  constructor(private store: Store) {
    this.store.dispatch(UsersActions.loadUsers());
  }
}
```

This completes the implementation of NgRx for CRUD operations with `/api/users`. 🚀



# **Dynamically Update Header User Data using NgRx**

## **1. Update User Model (`user.model.ts`)**
Ensure the user model includes `profileImage`:
```typescript
export interface User {
  id: string;
  name: string;
  email: string;
  profileImage: string;
}
```

---

## **2. Define Actions (`users.actions.ts`)**
Add actions for updating user profile data:
```typescript
import { createAction, props } from '@ngrx/store';
import { User } from '../../models/user.model';

export const updateUserProfile = createAction(
  '[User] Update Profile',
  props<{ user: Partial<User> }>()
);

export const updateUserProfileSuccess = createAction(
  '[User] Update Profile Success',
  props<{ user: User }>()
);

export const updateUserProfileFailure = createAction(
  '[User] Update Profile Failure',
  props<{ error: string }>()
);
```

---

## **3. Modify Reducer (`users.reducer.ts`)**
Handle the user profile updates:
```typescript
import { createReducer, on } from '@ngrx/store';
import { User } from '../../models/user.model';
import * as UsersActions from './users.actions';

export interface UsersState {
  user: User | null;
  loading: boolean;
  error: string | null;
}

export const initialState: UsersState = {
  user: null,
  loading: false,
  error: null,
};

export const usersReducer = createReducer(
  initialState,
  on(UsersActions.updateUserProfile, (state) => ({ ...state, loading: true })),
  on(UsersActions.updateUserProfileSuccess, (state, { user }) => ({
    ...state,
    user,
    loading: false,
  })),
  on(UsersActions.updateUserProfileFailure, (state, { error }) => ({
    ...state,
    loading: false,
    error,
  }))
);
```

---

## **4. Implement Effects (`users.effects.ts`)**
Handle API calls for updating user profiles:
```typescript
import { Injectable } from '@angular/core';
import { Actions, createEffect, ofType } from '@ngrx/effects';
import { catchError, map, mergeMap, of } from 'rxjs';
import { UsersService } from '../../services/users.service';
import * as UsersActions from './users.actions';

@Injectable()
export class UsersEffects {
  constructor(private actions$: Actions, private usersService: UsersService) {}

  updateUserProfile$ = createEffect(() =>
    this.actions$.pipe(
      ofType(UsersActions.updateUserProfile),
      mergeMap(({ user }) =>
        this.usersService.updateUserProfile(user).pipe(
          map((updatedUser) => UsersActions.updateUserProfileSuccess({ user: updatedUser })),
          catchError((error) => of(UsersActions.updateUserProfileFailure({ error })))
        )
      )
    )
  );
}
```

---

## **5. Update User Service (`users.service.ts`)**
Modify the service to handle profile updates:
```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { User } from '../models/user.model';

@Injectable({
  providedIn: 'root',
})
export class UsersService {
  private apiUrl = '/api/users';

  constructor(private http: HttpClient) {}

  updateUserProfile(user: Partial<User>): Observable<User> {
    return this.http.put<User>(`${this.apiUrl}/${user.id}`, user);
  }
}
```

---

## **6. Dispatch Action from Profile Component**
Trigger the update action in the Profile Component:
```typescript
import { Component } from '@angular/core';
import { Store } from '@ngrx/store';
import { User } from '../../models/user.model';
import * as UsersActions from '../../store/users/users.actions';

@Component({
  selector: 'app-profile',
  templateUrl: './profile.component.html',
})
export class ProfileComponent {
  constructor(private store: Store) {}

  updateProfile(updatedUser: Partial<User>) {
    this.store.dispatch(UsersActions.updateUserProfile({ user: updatedUser }));
  }
}
```

---

## **7. Select Data in Header Component**
Subscribe to user state in the header:
```typescript
import { Component } from '@angular/core';
import { Store } from '@ngrx/store';
import { Observable } from 'rxjs';
import { User } from '../../models/user.model';

@Component({
  selector: 'app-header',
  templateUrl: './header.component.html',
})
export class HeaderComponent {
  user$: Observable<User | null> = this.store.select((state) => state.users.user);

  constructor(private store: Store) {}
}
```

---

## **8. Display User Data in Header (`header.component.html`)**
```html
<div class="header">
  <img *ngIf="user$ | async as user" [src]="user.profileImage" alt="Profile Image" />
  <span *ngIf="user$ | async as user">{{ user.name }}</span>
</div>
```







---

## **Final Outcome**
- When a user updates their **name** or **profile picture** in the **Profile Component**, 
- The **Header Component** will automatically reflect the changes due to NgRx state management.

🚀 **Dynamic updates without refreshing!**




I'd be happy to teach you about Angular NgRx in detail! NgRx is a powerful state management library for Angular applications based on Redux principles. Let's break this down into manageable sections.

# Comprehensive Angular NgRx Guide

## Table of Contents
1. [Introduction to NgRx](#introduction-to-ngrx)
2. [Core Concepts](#core-concepts)
3. [Setting Up NgRx](#setting-up-ngrx)
4. [Actions](#actions)
5. [Reducers](#reducers)
6. [Selectors](#selectors)
7. [Effects](#effects)
8. [Store](#store)
9. [Entity State](#entity-state)
10. [Router Store](#router-store)
11. [DevTools](#devtools)
12. [Best Practices](#best-practices)
13. [Real-World Example](#real-world-example)

## Introduction to NgRx

NgRx is a framework for building reactive applications in Angular. It provides state management, isolation of side effects, entity collection management, router state management, code generation, and developer tools that enhance development experience.

### Why Use NgRx?
- **Predictable State Management**: Single source of truth for application state
- **Performance**: OnPush change detection strategy for better performance
- **Scalability**: Scales with your application
- **Testability**: Makes testing straightforward
- **Developer Tools**: Time-travel debugging, state inspection
- **Powerful Effects System**: Manage side effects like HTTP requests

## Core Concepts

NgRx follows the Redux pattern with Angular-specific adaptations:

1. **Store**: Single source of truth for state
2. **Actions**: Events that trigger state changes
3. **Reducers**: Pure functions that create new state
4. **Selectors**: Functions for selecting pieces of state
5. **Effects**: Handle side effects and async operations

### The NgRx Data Flow
1. Component dispatches an Action
2. Reducer processes the Action, creating new state
3. Store emits new state
4. Components receive updated state via Selectors

## Setting Up NgRx

### Installation

```bash
# Install the core NgRx packages
npm install @ngrx/store @ngrx/effects @ngrx/entity @ngrx/store-devtools @ngrx/router-store --save
```

### Basic Setup in AppModule

```typescript
import { StoreModule } from '@ngrx/store';
import { EffectsModule } from '@ngrx/effects';
import { StoreDevtoolsModule } from '@ngrx/store-devtools';
import { environment } from '../environments/environment';

@NgModule({
  imports: [
    BrowserModule,
    StoreModule.forRoot({}, {}),
    EffectsModule.forRoot([]),
    StoreDevtoolsModule.instrument({
      maxAge: 25, // Retains last 25 states
      logOnly: environment.production,
    }),
  ],
  declarations: [AppComponent],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

## Actions

Actions are events that describe something that happened in the application.

### Creating Actions

Using the modern createAction approach:

```typescript
// user.actions.ts
import { createAction, props } from '@ngrx/store';
import { User } from './user.model';

export const loadUsers = createAction('[User] Load Users');
export const loadUsersSuccess = createAction(
  '[User] Load Users Success',
  props<{ users: User[] }>()
);
export const loadUsersFailure = createAction(
  '[User] Load Users Failure',
  props<{ error: any }>()
);

export const addUser = createAction(
  '[User] Add User',
  props<{ user: User }>()
);
```

### Action Best Practices
- Use a descriptive, event-oriented name
- Include a clear source in square brackets
- Include relevant data in the payload
- Keep actions granular and specific

## Reducers

Reducers are pure functions that take the current state and an action and return a new state.

### Creating Reducers

```typescript
// user.reducer.ts
import { createReducer, on } from '@ngrx/store';
import * as UserActions from './user.actions';
import { User } from './user.model';

export interface UserState {
  users: User[];
  loading: boolean;
  error: any;
}

export const initialState: UserState = {
  users: [],
  loading: false,
  error: null
};

export const userReducer = createReducer(
  initialState,
  on(UserActions.loadUsers, state => ({
    ...state,
    loading: true,
    error: null
  })),
  on(UserActions.loadUsersSuccess, (state, { users }) => ({
    ...state,
    users,
    loading: false
  })),
  on(UserActions.loadUsersFailure, (state, { error }) => ({
    ...state,
    error,
    loading: false
  })),
  on(UserActions.addUser, (state, { user }) => ({
    ...state,
    users: [...state.users, user]
  }))
);
```

### Register Reducer in Module

```typescript
// user.module.ts
import { StoreModule } from '@ngrx/store';
import { userReducer } from './user.reducer';

@NgModule({
  imports: [
    StoreModule.forFeature('users', userReducer),
    // other imports
  ]
})
export class UserModule {}
```

### Reducer Best Practices
- Keep reducers pure - no side effects
- Create a new state object for each change
- Handle each action explicitly
- Use the spread operator for immutability
- Split reducers by domain

## Selectors

Selectors are pure functions used for obtaining slices of state.

### Creating Selectors

```typescript
// user.selectors.ts
import { createFeatureSelector, createSelector } from '@ngrx/store';
import { UserState } from './user.reducer';

export const selectUserState = createFeatureSelector<UserState>('users');

export const selectAllUsers = createSelector(
  selectUserState,
  (state: UserState) => state.users
);

export const selectUserLoading = createSelector(
  selectUserState,
  (state: UserState) => state.loading
);

export const selectUserError = createSelector(
  selectUserState,
  (state: UserState) => state.error
);

// More complex selectors can combine information
export const selectUserCount = createSelector(
  selectAllUsers,
  (users) => users.length
);
```

### Using Selectors in Components

```typescript
// user-list.component.ts
import { Component, OnInit } from '@angular/core';
import { Store } from '@ngrx/store';
import { Observable } from 'rxjs';
import * as fromUser from './user.selectors';
import * as UserActions from './user.actions';
import { User } from './user.model';

@Component({
  selector: 'app-user-list',
  templateUrl: './user-list.component.html'
})
export class UserListComponent implements OnInit {
  users$: Observable<User[]>;
  loading$: Observable<boolean>;
  error$: Observable<any>;

  constructor(private store: Store) {
    this.users$ = this.store.select(fromUser.selectAllUsers);
    this.loading$ = this.store.select(fromUser.selectUserLoading);
    this.error$ = this.store.select(fromUser.selectUserError);
  }

  ngOnInit() {
    this.store.dispatch(UserActions.loadUsers());
  }
}
```

### Selector Best Practices
- Memoize selectors for performance
- Keep selectors focused on specific data needs
- Compose selectors for complex data transformations
- Use selectors everywhere you access store state

## Effects

Effects isolate side effects from components, allowing for more pure components that select state and dispatch actions.

### Creating Effects

```typescript
// user.effects.ts
import { Injectable } from '@angular/core';
import { Actions, createEffect, ofType } from '@ngrx/effects';
import { of } from 'rxjs';
import { catchError, map, mergeMap } from 'rxjs/operators';
import * as UserActions from './user.actions';
import { UserService } from './user.service';

@Injectable()
export class UserEffects {
  loadUsers$ = createEffect(() => this.actions$.pipe(
    ofType(UserActions.loadUsers),
    mergeMap(() => this.userService.getUsers().pipe(
      map(users => UserActions.loadUsersSuccess({ users })),
      catchError(error => of(UserActions.loadUsersFailure({ error })))
    ))
  ));

  constructor(
    private actions$: Actions,
    private userService: UserService
  ) {}
}
```

### Register Effects in Module

```typescript
// user.module.ts
import { EffectsModule } from '@ngrx/effects';
import { UserEffects } from './user.effects';

@NgModule({
  imports: [
    // other imports
    EffectsModule.forFeature([UserEffects])
  ]
})
export class UserModule {}
```

### Effect Best Practices
- Keep effects focused on a single responsibility
- Handle errors properly in effects
- Use appropriate flattening operators (mergeMap, switchMap, concatMap)
- Avoid dispatching actions from services
- Test effects thoroughly

## Store

The Store is the central repository of state in NgRx.

### Accessing the Store

```typescript
// user-detail.component.ts
import { Component, OnInit } from '@angular/core';
import { Store } from '@ngrx/store';
import { Observable } from 'rxjs';
import * as fromUser from './user.selectors';
import * as UserActions from './user.actions';

@Component({
  selector: 'app-user-detail',
  templateUrl: './user-detail.component.html'
})
export class UserDetailComponent implements OnInit {
  selectedUser$: Observable<User>;

  constructor(private store: Store) {
    this.selectedUser$ = this.store.select(fromUser.selectSelectedUser);
  }

  ngOnInit() {
    // Dispatch an action to load a specific user
    this.store.dispatch(UserActions.loadUser({ id: this.route.snapshot.params.id }));
  }

  updateUser(user: User) {
    this.store.dispatch(UserActions.updateUser({ user }));
  }
}
```

### Store Best Practices
- Organize state by feature
- Never mutate state directly
- Keep components as pure as possible by delegating state management to NgRx
- Use OnPush change detection with Observable-based state

## Entity State

NgRx Entity provides a simple API for managing collections of entities.

### Setting up Entity State

```typescript
// user.reducer.ts
import { createEntityAdapter, EntityAdapter, EntityState } from '@ngrx/entity';
import { createReducer, on } from '@ngrx/store';
import * as UserActions from './user.actions';
import { User } from './user.model';

export interface UserState extends EntityState<User> {
  selectedUserId: string | null;
  loading: boolean;
  error: any;
}

export const adapter: EntityAdapter<User> = createEntityAdapter<User>({
  selectId: (user: User) => user.id,
  sortComparer: (a: User, b: User) => a.name.localeCompare(b.name),
});

export const initialState: UserState = adapter.getInitialState({
  selectedUserId: null,
  loading: false,
  error: null
});

export const userReducer = createReducer(
  initialState,
  on(UserActions.loadUsers, state => ({
    ...state,
    loading: true,
    error: null
  })),
  on(UserActions.loadUsersSuccess, (state, { users }) => 
    adapter.setAll(users, {
      ...state,
      loading: false
    })
  ),
  on(UserActions.loadUsersFailure, (state, { error }) => ({
    ...state,
    error,
    loading: false
  })),
  on(UserActions.addUser, (state, { user }) => 
    adapter.addOne(user, state)
  ),
  on(UserActions.updateUser, (state, { user }) => 
    adapter.updateOne({ id: user.id, changes: user }, state)
  ),
  on(UserActions.deleteUser, (state, { id }) => 
    adapter.removeOne(id, state)
  ),
  on(UserActions.selectUser, (state, { id }) => ({
    ...state,
    selectedUserId: id
  }))
);

// Create the default selectors
export const {
  selectIds,
  selectEntities,
  selectAll,
  selectTotal,
} = adapter.getSelectors();
```

### Entity Selectors

```typescript
// user.selectors.ts
import { createFeatureSelector, createSelector } from '@ngrx/store';
import * as fromUser from './user.reducer';

export const selectUserState = createFeatureSelector<fromUser.UserState>('users');

// Using the adapter's selectors
export const selectUserIds = createSelector(
  selectUserState,
  fromUser.selectIds
);

export const selectUserEntities = createSelector(
  selectUserState,
  fromUser.selectEntities
);

export const selectAllUsers = createSelector(
  selectUserState,
  fromUser.selectAll
);

export const selectUserTotal = createSelector(
  selectUserState,
  fromUser.selectTotal
);

export const selectCurrentUserId = createSelector(
  selectUserState,
  state => state.selectedUserId
);

export const selectCurrentUser = createSelector(
  selectUserEntities,
  selectCurrentUserId,
  (userEntities, userId) => userId ? userEntities[userId] : null
);
```

## Router Store

NgRx Router Store connects the Angular Router to the NgRx Store.

### Setting up Router Store

```typescript
// app.module.ts
import { StoreRouterConnectingModule, routerReducer } from '@ngrx/router-store';

@NgModule({
  imports: [
    // other imports
    StoreModule.forRoot({
      router: routerReducer,
    }),
    StoreRouterConnectingModule.forRoot()
  ]
})
export class AppModule {}
```

### Creating Custom Router Serializer

```typescript
// router.serializer.ts
import { Params, RouterStateSnapshot } from '@angular/router';
import { RouterStateSerializer } from '@ngrx/router-store';

export interface RouterStateUrl {
  url: string;
  params: Params;
  queryParams: Params;
}

export class CustomSerializer implements RouterStateSerializer<RouterStateUrl> {
  serialize(routerState: RouterStateSnapshot): RouterStateUrl {
    let route = routerState.root;

    while (route.firstChild) {
      route = route.firstChild;
    }

    const {
      url,
      root: { queryParams },
    } = routerState;
    const { params } = route;

    // Only return an object including the URL, params and query params
    // instead of the entire snapshot
    return { url, params, queryParams };
  }
}
```

### Configure Custom Serializer

```typescript
// app.module.ts
import { StoreRouterConnectingModule } from '@ngrx/router-store';
import { CustomSerializer } from './router.serializer';

@NgModule({
  imports: [
    // other imports
    StoreRouterConnectingModule.forRoot({
      serializer: CustomSerializer
    })
  ]
})
export class AppModule {}
```

### Creating Router Selectors

```typescript
// router.selectors.ts
import { createFeatureSelector, createSelector } from '@ngrx/store';
import { RouterReducerState, getSelectors } from '@ngrx/router-store';
import { RouterStateUrl } from './router.serializer';

export const selectRouter = createFeatureSelector<
  RouterReducerState<RouterStateUrl>
>('router');

export const {
  selectCurrentRoute,
  selectQueryParams,
  selectQueryParam,
  selectRouteParams,
  selectRouteParam,
  selectUrl,
  selectFragment,
  selectRouteData,
} = getSelectors(selectRouter);

// Custom selectors
export const selectRouteId = createSelector(
  selectRouteParams,
  params => params.id
);
```

## DevTools

NgRx DevTools provides debugging capabilities for NgRx applications.

### Setting up DevTools

```typescript
// app.module.ts
import { StoreDevtoolsModule } from '@ngrx/store-devtools';
import { environment } from '../environments/environment';

@NgModule({
  imports: [
    // other imports
    !environment.production ? StoreDevtoolsModule.instrument({
      maxAge: 25, // Retains last 25 states
      logOnly: environment.production,
      autoPause: true, // Pauses recording actions and state changes when the extension window is not open
      trace: false, // If set to true, will include stack trace for every dispatched action
      traceLimit: 75, // maximum stack trace frames to be stored (in case trace option was provided as true)
    }) : []
  ]
})
export class AppModule {}
```

### Using DevTools
Once installed, you can:
- Inspect state changes over time
- Time travel through state changes
- Dispatch actions directly
- Export/import state for debugging
- View action payloads

## Best Practices

### State Design
1. **Normalize Data**: Use flat state structures
2. **Minimal State**: Only store what you need
3. **Derive Data**: Use selectors for derived data
4. **Entity Pattern**: Use @ngrx/entity for collections

### Performance
1. **Memoized Selectors**: Use createSelector for performance
2. **OnPush Change Detection**: Use with observables
3. **Avoid Redundant Actions**: Don't dispatch unnecessarily
4. **Optimize Effect Streams**: Use appropriate operators

### Architecture
1. **Feature States**: Organize by domain/feature
2. **Facade Pattern**: Optionally use facades to abstract NgRx from components
3. **Strong Typing**: Use TypeScript interfaces for all state
4. **Testing**: Test actions, reducers, selectors, and effects

### Organization
1. **Feature Modules**: Align with Angular module structure
2. **Barrel Files**: Use index.ts for public API
3. **Clear Naming**: Follow consistent naming conventions
4. **Action Grouping**: Group related actions

## Real-World Example

Let's build a simple todo application with NgRx:

### Model

```typescript
// todo.model.ts
export interface Todo {
  id: string;
  text: string;
  completed: boolean;
  createdAt: Date;
}
```

### Actions

```typescript
// todo.actions.ts
import { createAction, props } from '@ngrx/store';
import { Todo } from './todo.model';

export const loadTodos = createAction('[Todo] Load Todos');
export const loadTodosSuccess = createAction(
  '[Todo] Load Todos Success',
  props<{ todos: Todo[] }>()
);
export const loadTodosFailure = createAction(
  '[Todo] Load Todos Failure',
  props<{ error: any }>()
);

export const addTodo = createAction(
  '[Todo] Add Todo',
  props<{ text: string }>()
);
export const addTodoSuccess = createAction(
  '[Todo] Add Todo Success',
  props<{ todo: Todo }>()
);

export const toggleTodo = createAction(
  '[Todo] Toggle Todo',
  props<{ id: string }>()
);

export const deleteTodo = createAction(
  '[Todo] Delete Todo',
  props<{ id: string }>()
);
```

### Reducer

```typescript
// todo.reducer.ts
import { createEntityAdapter, EntityAdapter, EntityState } from '@ngrx/entity';
import { createReducer, on } from '@ngrx/store';
import * as TodoActions from './todo.actions';
import { Todo } from './todo.model';

export interface TodoState extends EntityState<Todo> {
  loading: boolean;
  error: any;
}

export const adapter: EntityAdapter<Todo> = createEntityAdapter<Todo>({
  selectId: (todo: Todo) => todo.id,
  sortComparer: (a: Todo, b: Todo) => b.createdAt.getTime() - a.createdAt.getTime(),
});

export const initialState: TodoState = adapter.getInitialState({
  loading: false,
  error: null
});

export const todoReducer = createReducer(
  initialState,
  on(TodoActions.loadTodos, state => ({
    ...state,
    loading: true,
    error: null
  })),
  on(TodoActions.loadTodosSuccess, (state, { todos }) => 
    adapter.setAll(todos, {
      ...state,
      loading: false
    })
  ),
  on(TodoActions.loadTodosFailure, (state, { error }) => ({
    ...state,
    error,
    loading: false
  })),
  on(TodoActions.addTodoSuccess, (state, { todo }) => 
    adapter.addOne(todo, state)
  ),
  on(TodoActions.toggleTodo, (state, { id }) => {
    const todo = state.entities[id];
    if (!todo) return state;
    return adapter.updateOne(
      {
        id,
        changes: { completed: !todo.completed }
      },
      state
    );
  }),
  on(TodoActions.deleteTodo, (state, { id }) => 
    adapter.removeOne(id, state)
  )
);

export const {
  selectIds,
  selectEntities,
  selectAll,
  selectTotal,
} = adapter.getSelectors();
```

### Selectors

```typescript
// todo.selectors.ts
import { createFeatureSelector, createSelector } from '@ngrx/store';
import * as fromTodo from './todo.reducer';

export const selectTodoState = createFeatureSelector<fromTodo.TodoState>('todos');

export const selectTodoIds = createSelector(
  selectTodoState,
  fromTodo.selectIds
);

export const selectTodoEntities = createSelector(
  selectTodoState,
  fromTodo.selectEntities
);

export const selectAllTodos = createSelector(
  selectTodoState,
  fromTodo.selectAll
);

export const selectTodoTotal = createSelector(
  selectTodoState,
  fromTodo.selectTotal
);

export const selectTodoLoading = createSelector(
  selectTodoState,
  state => state.loading
);

export const selectTodoError = createSelector(
  selectTodoState,
  state => state.error
);

export const selectCompletedTodos = createSelector(
  selectAllTodos,
  todos => todos.filter(todo => todo.completed)
);

export const selectActiveTodos = createSelector(
  selectAllTodos,
  todos => todos.filter(todo => !todo.completed)
);
```

### Effects

```typescript
// todo.effects.ts
import { Injectable } from '@angular/core';
import { Actions, createEffect, ofType } from '@ngrx/effects';
import { of } from 'rxjs';
import { catchError, map, mergeMap, switchMap } from 'rxjs/operators';
import { v4 as uuidv4 } from 'uuid';
import * as TodoActions from './todo.actions';
import { TodoService } from './todo.service';

@Injectable()
export class TodoEffects {
  loadTodos$ = createEffect(() => this.actions$.pipe(
    ofType(TodoActions.loadTodos),
    switchMap(() => this.todoService.getTodos().pipe(
      map(todos => TodoActions.loadTodosSuccess({ todos })),
      catchError(error => of(TodoActions.loadTodosFailure({ error })))
    ))
  ));

  addTodo$ = createEffect(() => this.actions$.pipe(
    ofType(TodoActions.addTodo),
    mergeMap(({ text }) => {
      const newTodo = {
        id: uuidv4(),
        text,
        completed: false,
        createdAt: new Date()
      };
      return this.todoService.addTodo(newTodo).pipe(
        map(() => TodoActions.addTodoSuccess({ todo: newTodo })),
        catchError(error => of(TodoActions.loadTodosFailure({ error })))
      );
    })
  ));

  toggleTodo$ = createEffect(() => this.actions$.pipe(
    ofType(TodoActions.toggleTodo),
    mergeMap(({ id }) => this.todoService.toggleTodo(id).pipe(
      map(() => ({ type: '[Todo] Toggle Todo Success' })),
      catchError(error => of(TodoActions.loadTodosFailure({ error })))
    ))
  ));

  deleteTodo$ = createEffect(() => this.actions$.pipe(
    ofType(TodoActions.deleteTodo),
    mergeMap(({ id }) => this.todoService.deleteTodo(id).pipe(
      map(() => ({ type: '[Todo] Delete Todo Success' })),
      catchError(error => of(TodoActions.loadTodosFailure({ error })))
    ))
  ));

  constructor(
    private actions$: Actions,
    private todoService: TodoService
  ) {}
}
```

### Component

```typescript
// todo-list.component.ts
import { Component, OnInit } from '@angular/core';
import { Store } from '@ngrx/store';
import { Observable } from 'rxjs';
import * as TodoActions from './todo.actions';
import * as fromTodo from './todo.selectors';
import { Todo } from './todo.model';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';

@Component({
  selector: 'app-todo-list',
  template: `
    <div>
      <h2>Todo List</h2>
      <form [formGroup]="todoForm" (ngSubmit)="onSubmit()">
        <input formControlName="text" placeholder="Add a new todo...">
        <button type="submit" [disabled]="todoForm.invalid">Add</button>
      </form>

      <div *ngIf="loading$ | async">Loading...</div>
      <div *ngIf="error$ | async as error" class="error">{{ error }}</div>

      <h3>Active ({{ (activeTodos$ | async)?.length }})</h3>
      <ul>
        <li *ngFor="let todo of activeTodos$ | async">
          <input type="checkbox" [checked]="todo.completed" (change)="toggleTodo(todo.id)">
          {{ todo.text }}
          <button (click)="deleteTodo(todo.id)">Delete</button>
        </li>
      </ul>

      <h3>Completed ({{ (completedTodos$ | async)?.length }})</h3>
      <ul>
        <li *ngFor="let todo of completedTodos$ | async">
          <input type="checkbox" [checked]="todo.completed" (change)="toggleTodo(todo.id)">
          <span class="completed">{{ todo.text }}</span>
          <button (click)="deleteTodo(todo.id)">Delete</button>
        </li>
      </ul>
    </div>
  `,
  styles: [`
    .completed { text-decoration: line-through; }
    .error { color: red; }
  `]
})
export class TodoListComponent implements OnInit {
  todos$: Observable<Todo[]>;
  activeTodos$: Observable<Todo[]>;
  completedTodos$: Observable<Todo[]>;
  loading$: Observable<boolean>;
  error$: Observable<any>;
  todoForm: FormGroup;

  constructor(
    private store: Store,
    private fb: FormBuilder
  ) {
    this.todos$ = this.store.select(fromTodo.selectAllTodos);
    this.activeTodos$ = this.store.select(fromTodo.selectActiveTodos);
    this.completedTodos$ = this.store.select(fromTodo.selectCompletedTodos);
    this.loading$ = this.store.select(fromTodo.selectTodoLoading);
    this.error$ = this.store.select(fromTodo.selectTodoError);

    this.todoForm = this.fb.group({
      text: ['', [Validators.required, Validators.minLength(3)]]
    });
  }

  ngOnInit() {
    this.store.dispatch(TodoActions.loadTodos());
  }

  onSubmit() {
    if (this.todoForm.valid) {
      this.store.dispatch(TodoActions.addTodo({ text: this.todoForm.value.text }));
      this.todoForm.reset();
    }
  }

  toggleTodo(id: string) {
    this.store.dispatch(TodoActions.toggleTodo({ id }));
  }

  deleteTodo(id: string) {
    this.store.dispatch(TodoActions.deleteTodo({ id }));
  }
}
```

This comprehensive guide covers the core concepts and best practices for using NgRx in Angular applications. The real-world example demonstrates how all these pieces fit together to create a maintainable, scalable application.

I've created a comprehensive guide to Angular NgRx for you. Here's an overview of what's included:

## What You'll Learn

NgRx is a state management library for Angular based on Redux principles. It helps you manage application state in a predictable way, especially useful for medium to large applications with complex data flows.

The guide covers:

1. **Core Concepts**: Store, Actions, Reducers, Selectors, and Effects
2. **Setup and Configuration**: How to install and set up NgRx in your Angular app
3. **Actions**: Creating and dispatching events that trigger state changes
4. **Reducers**: Pure functions that create new state based on actions
5. **Selectors**: Functions for efficiently selecting and deriving state
6. **Effects**: Handling side effects like HTTP requests
7. **Entity State**: Managing collections of data efficiently
8. **Router Store**: Connecting Angular Router to NgRx
9. **DevTools**: Debugging capabilities for NgRx applications
10. **Best Practices**: Performance tips and architectural patterns

The guide finishes with a complete real-world example of a todo application using NgRx, showing how all the pieces fit together.

Would you like me to explain any specific part of NgRx in more detail?



