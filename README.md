# angular2-jwt-session
[![Build Status](https://travis-ci.org/ratnam99/Angular2-JWTSession.svg?branch=master)]
[![npm version](https://img.shields.io/npm/v/angular2-jwt-session.svg)](https://www.npmjs.com/package/angular2-jwt-session) [![license](https://img.shields.io/npm/l/angular2-jwt-session.svg)]

**angular2-jwt-session** is a helper library for working with [JWTs](http://jwt.io/introduction) in your Angular 2 applications. Also it can be used to implement "Keep me logged in feature" with the help of local storage and session storage.


## Contents
 - [What is this Library for?](#what-is-this-library-for)
 - [Key Features](#key-features)
 - [How to install?](#how-to-install)
 - [Configurations Required](#configurations-required)
 - [Sending Authenticated Requests](#sending-authenticated-requests)
 - [Configuration Options](#configuration-options)
    - [Advanced Configuration](#advanced-configuration)
    - [Sending Per-Request Headers](#sending-per-request-headers)
    - [Using the Observable Token Stream](#using-the-observable-token-stream)
    - [Using JwtHelper in Components](#using-jwthelper-in-components)
 - [Checking Authentication to Hide/Show Elements and Handle Routing](#checking-authentication-to-hideshow-elements-and-handle-routing)
 - [Keep me logged in](#keep-me-logged-in)
 - [Issue Reporting](#issue-reporting)
 - [Author](#author)
 - [License](#license)

## What is this Library for?

**angular2-jwt-session** is a small and unopinionated library that is useful for automatically attaching a [JSON Web Token (JWT)](http://jwt.io/introduction) as an `Authorization` header when making HTTP requests from an Angular 2 app. It also has a number of helper methods that are useful for doing things like decoding JWTs.

This library does not have any functionality for (or opinion about) implementing user authentication and retrieving JWTs to begin with. Those details will vary depending on your setup, but in most cases, you will use a regular HTTP request to authenticate your users and then save their JWTs in local storage or session storage or in a cookie if successful.


The library comes with several helpers that are useful in your Angular 2 apps.

1. `AuthHttp` - allows for individual and explicit authenticated HTTP requests
2. `tokenNotExpired` - allows you to check whether there is a non-expired JWT in local storage or session storage. This can be used for conditionally showing/hiding elements and stopping navigation to certain routes if the user isn't authenticated


This library not only attaches a JWT but also can be easily used to implement "Keep me logged in", "Stay signed in" or "Remember me" feature in your web app.



## Key Features

* Send a JWT on a per-request basis using the **explicit `AuthHttp`** class
* **Decode a JWT** from your Angular 2 app
* Check the **expiration date** of the JWT
* Conditionally allow **route navigation** based on JWT status
* Implement "Keep me logged in" feature


## How to Install?

```bash
npm install angular2-jwt-session --save
```

## Configurations Required

Create a new `auth.module.ts` file with the following code:

```ts
import { NgModule } from '@angular/core';
import { Http, RequestOptions } from '@angular/http';
import { AuthHttp, AuthConfig } from 'angular2-jwt-session';

export function authHttpServiceFactory(http: Http, options: RequestOptions) {
  return new AuthHttp(new AuthConfig(), http, options);
}

@NgModule({
  providers: [
    {
      provide: AuthHttp,
      useFactory: authHttpServiceFactory,
      deps: [Http, RequestOptions]
    }
  ]
})
export class AuthModule {}
```

We added a factory function to use as a provider for `AuthHttp`. This will allow you to configure angular2-jwt-session in the `AuthConfig` instance later on.


## Sending Authenticated Requests

If you wish to only send a JWT on a specific HTTP request, you can use the `AuthHttp` class. This class is a wrapper for Angular 2's `Http` and thus supports all the same HTTP methods.

```ts
import { AuthHttp } from 'angular2-jwt-session';
// ...
class App {

  someThing: string;

  constructor(public authHttp: AuthHttp) {}

  getSomething() {
    this.authHttp.get('http://example.com/api/something')
      .subscribe(
        data => this.someThing = data,
        err => console.log(err),
        () => console.log('Request Complete')
      );
  }
}
```


## Configuration Options

`AUTH_PROVIDERS` gives a default configuration setup:

* Header Name: `Authorization`
* Header Prefix: `Bearer`
* Token Name: `token`
* Token Getter Function: `(() => localStorage.getItem(tokenName) or sessionStorage.getItem(tokenName))`
* Supress error and continue with regular HTTP request if no JWT is saved: `false`
* Global Headers: none

If you wish to configure the `headerName`, `headerPrefix`, `tokenName`, `tokenGetter` function, `noTokenScheme`, `globalHeaders`, or `noJwtError` boolean, you can using `provideAuth` or the factory pattern (see below).

#### Errors

By default, if there is no valid JWT saved, `AuthHttp` will return an Observable `error` with 'Invalid JWT'. If you would like to continue with an unauthenticated request instead, you can set `noJwtError` to `true`.

#### Token Scheme

The default scheme for the `Authorization` header is `Bearer`, but you may either provide your own by specifying a `headerPrefix`, or you may remove the prefix altogether by setting `noTokenScheme` to `true`.

#### Global Headers

You may set as many global headers as you like by passing an array of header-shaped objects to `globalHeaders`.

### Advanced Configuration

You may customize any of the above options using a factory which returns an `AuthHttp` instance with the options you would like to change.

```ts
import { NgModule } from '@angular/core';
import { Http, RequestOptions } from '@angular/http';
import { AuthHttp, AuthConfig } from 'angular2-jwt-session';

export function authHttpServiceFactory(http: Http, options: RequestOptions) {
  return new AuthHttp(new AuthConfig({
    tokenName: 'token',
		tokenGetter: (() => sessionStorage.getItem('token')),
		globalHeaders: [{'Content-Type':'application/json'}],
	}), http, options);
}

@NgModule({
  providers: [
    {
      provide: AuthHttp,
      useFactory: authHttpServiceFactory,
      deps: [Http, RequestOptions]
    }
  ]
})
export class AuthModule {}
```

### Sending Per-Request Headers

You may also send custom headers on a per-request basis with your `authHttp` request by passing them in an options object.

```ts
getSomeThing() {
  let myHeader = new Headers();
  myHeader.append('Content-Type', 'application/json');

  this.authHttp.get('http://example.com/api/something', { headers: myHeader })
    .subscribe(
      data => this.someThing = data,
      err => console.log(error),
      () => console.log('Request Complete')
    );

  // Pass it after the body in a POST request
  this.authHttp.post('http://example.com/api/something', 'post body', { headers: myHeader })
    .subscribe(
      data => this.someThing = data,
      err => console.log(err),
      () => console.log('Request Complete')
    );
}
```

### Using the Observable Token Stream

If you wish to use the JWT as an observable stream, you can call `tokenStream` from `AuthHttp`.

```ts
tokenSubscription() {
  this.authHttp.tokenStream.subscribe(
      data => console.log(data),
      err => console.log(err),
      () => console.log('Complete')
    );
}
```

This can be useful for cases where you want to make HTTP requests out of observable streams. The `tokenStream` can be mapped and combined with other streams at will.


## Using JwtHelper in Components

The `JwtHelper` class has several useful methods that can be utilized in your components:

* `decodeToken`
* `getTokenExpirationDate`
* `isTokenExpired`

You can use these methods by passing in the token to be evaluated.

```ts
jwtHelper: JwtHelper = new JwtHelper();

useJwtHelper() {
  var token = localStorage.getItem('token');

  console.log(
    this.jwtHelper.decodeToken(token),
    this.jwtHelper.getTokenExpirationDate(token),
    this.jwtHelper.isTokenExpired(token)
  );
}
```


## Checking Authentication to Hide/Show Elements and Handle Routing

The `tokenNotExpired` function can be used to check whether a JWT exists in local storage or session storage, and if it does, whether it has expired or not. If the token is valid, `tokenNotExpired` returns `true`, otherwise it returns `false`.

> **Note:** `tokenNotExpired` will by default assume the token name is `token` unless a token name is passed to it, ex: `tokenNotExpired('token_name')`. This will be changed in a future release to automatically use the token name that is set in `AuthConfig`.

```ts
// auth.service.ts

import { tokenNotExpired } from 'angular2-jwt-session';

loggedIn() {
  return tokenNotExpired();
}
```

The `loggedIn` method can now be used in views to conditionally hide and show elements.

```html
 <button id="login" *ngIf="!authenticate.loggedIn()">Log In</button>
 <button id="logout" *ngIf="authenticate.loggedIn()">Log Out</button>
```

To guard routes that should be limited to authenticated users, set up an `AuthGuard`.

```ts
// auth-guard.service.ts

import { Injectable } from '@angular/core';
import { Router } from '@angular/router';
import { CanActivate } from '@angular/router';
import { Auth } from './auth.service';

@Injectable()
export class AuthGuard implements CanActivate {

  constructor(private authenticate: Auth, private router: Router) {}

  canActivate() {
    if(this.authenticate.loggedIn()) {
      return true;
    } else {
      this.router.navigate(['unauthorized']);
      return false;
    }
  }
}
```

With the guard in place, you can use it in your route configuration.

```ts
import { AuthGuard } from './auth.guard';

export const routes: RouterConfig = [
  { path: 'admin', component: AdminComponent, canActivate: [AuthGuard] },
  { path: 'unauthorized', component: UnauthorizedComponent }
];
```


## Keep me logged in

Once you become aware of the working of this library and how to use it, it becomes very simple to implement the "Keep me logged in" or "stay signed in" feature in your web app.

For this, I assume you have done all the above configurations and have generated JWTs. To implement this feature, you need to add a checkbox to your login page and model it with a boolean "rememberMe" which is by default false. On checking the checkbox this value changes to true and on unchecking the checkbox it becomes false. This boolean is to be stored in local storage when login button is clicked.

Now comes the logic, when the login button is pressed and rememberMe is true (i.e. checkbox is checked) the JWT is stored in the local storage using `localStorage.setItem(JWT)`. And if the login button is pressed and rememberMe is false (i.e. checkbox is not checked or unchecked) the JWT is stored in the sessionStorage using `sessionStorage.setItem(JWT)`.

All the other authentications, comparisons during naviagtion or during sending requests are handled by JWTHelper, AuthHttp and AuthConfig.

The "Keep me logged in" feature becomes very simple to implement using this library. 




## Issue Reporting

If you have found a bug or if you have a feature request, please report them at this [repository](https://github.com/ratnam99/Angular2-JWTSession.git) issues section. Please do not report security vulnerabilities on the public GitHub issue tracker.


## Author

[Kumar Ratnam Pandey](https://github.com/ratnam99) (Software Developer at [Geekyants](https://geekyants.com/))


## License

This project is licensed under the MIT license.
