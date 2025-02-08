# Implementing NgRx with Angular for Users API

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



