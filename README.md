# angular-8-role-based-authorization-example

Angular 8 Role Based Authorization Example with the Angular CLI

### AuthGuard

Angular guards are defined in `auth.guard.ts`. It relies upon the `authentication.service.ts` to retrive the user's JWT.

Guards only serve to create a clean and consistent user experience. Guards can be circumvented by malicious frontend users, however the browser will not have the data required to display the component.

All backend API calls must be enforced with server-side security.

### JWT Management

JWTs are kept in either local or session storage so that their payloads can be used for client-side guards. This demo uses local storage.

JWTs are decoded using the `@auth0/angular-jwt` Angular library and can be checked for expiration as well.

JWTs may be RSA signed by the server, however the client's browser cannot be trusted to be verify a signature. Thus there is no functionality for verifying RSA signatures via Angular (see <https://github.com/auth0/angular2-jwt/issues/206>).

#### Caveats on HttpOnly Cookies

JWTs stored in HttpOnly cookies cannot be accessed directly by client-side Javascript, and thus AuthGuard. However, if another backend call is made each time a route is accessed—presenting the HttpOnly cookie to the server to check for permission—it is possible to use AuthGuard with HttpOnly cookies. This however is a significant architectural decision with important tradeoffs.

### Demo Site Routes

The following routes are defined in `app.routing.ts` and are protected via the `canActivate` method.

```
const routes: Routes = [
    {
        path: '',
        component: HomeComponent,
        canActivate: [AuthGuard]
    },
    {
        path: 'admin',
        component: AdminComponent,
        canActivate: [AuthGuard],
        data: { roles: [Role.Admin] }
    },
    {
        path: 'login',
        component: LoginComponent
    },

    // otherwise redirect to home
    { path: '**', redirectTo: '' }
];
```

#### Login Page

`/login` is unsecured. If a user is not logged in, `auth.guard.ts` reroutes them to `/login`.

#### Home Page

`/` is secured.

#### Admin Page

`/admin` is restricted only to admin roles. If an unauthorized user attempts to access the `/admin` page directly, `auth.guard.ts` reroutes to the homepage.

In addition, the `Admin` navbar header is only displayed to admin roles as defined in `app.routing.ts`.

### Credential Storage

`authentication.service.ts` makes a REST call to obtain the JWT. `fake-backend.ts` stubs the calls, and provides two users as defined below.

The JWT header and payload are HMACSHA256 encoded using secret `lookman` using <https://jwt.io>. (The secret is not Base64 encoded.)


```
{
    "id": 1,
    "username": "admin",
    "firstName": "Admin",
    "lastName": "User",
    "role": "Admin",
    "token": "fake-jwt-token.1"
}
```
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MSwidXNlcm5hbWUiOiJhZG1pbiIsImZpcnN0TmFtZSI6IkFkbWluIiwibGFzdE5hbWUiOiJVc2VyIiwicm9sZSI6IkFkbWluIn0.h4erqv3evVuMosUoAG2zwxT5WvVAdy9CZtEjrk21xxg
```


```
{
    "id": 2,
    "username": "user",
    "firstName": "Normal",
    "lastName": "User",
    "role": "User",
    "token": "fake-jwt-token.2"
}
```
User's roles and permissions are saved to local storage in `authentication.service.ts` under the `currentUser` key. Upon logout, the `currentUser` key is deleted from local storage.

### References

* https://jasonwatmore.com/post/2019/08/06/angular-8-role-based-authorization-tutorial-with-example
* https://medium.com/@ryanchenkie_40935/angular-authentication-using-route-guards-bf7a4ca13ae3
* https://codinglatte.com/posts/angular/role-based-authorization-in-angular-route-guards/
* https://www.onooks.com/jwt-in-httponly-cookie-authguard-and-protected-routes-node-js-angular/
