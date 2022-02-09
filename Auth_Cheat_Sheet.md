# Authentication CheatSheet

### Auth service
> RUN ```ng g s auth/Auth```

```Javascript
import {Injectable} from '@angular/core';
import {HttpClient, HttpErrorResponse, HttpHeaders, HttpParams} from "@angular/common/http";
import {catchError, Observable, tap, throwError} from "rxjs";
import {TokenService} from "./token.service";
import * as endpoints from "./auth.endpoint";
import {environment} from "../../environments/environment";
import {Router} from "@angular/router";

const BASE_URL = environment.v1AuthEndpoint; // API Endpoint
const HTTP_OPTIONS = {
  headers: new HttpHeaders({
    'Content-Type': 'application/json'
  }),
  params: {}
} // HTTP Header

@Injectable({
  providedIn: 'root'
})
export class AuthService {
  refreshTokenInterval: any; // Interval to get refresh token
  private static handleError(error: HttpErrorResponse): any { // This function handles HTTP Response Errors
    if (error.error instanceof ErrorEvent) {
      console.error('An Error occurred: ', error.error.message)
    } else {
      console.error(
        `Error Code From Backend: ${error.status}`,
        `Body: ${error.error}`
      )
    }

    return throwError(
      'Internal server error!'
    )
  };

  private static log(message: string): any { // Simple function to print logs
    console.log(message);
  }

  constructor(
    private http: HttpClient,
    private tokenService: TokenService,
    private router: Router) {
  }

  login(loginPayload: any): Observable<any> {
    HTTP_OPTIONS.params = {
      grant_type: 'password',
      token_type: 'regular'
    } // Quaryparams 
    return this.http.post(BASE_URL + endpoints.LOGIN, loginPayload, HTTP_OPTIONS).pipe(
      tap((res: any) => {
        AuthService.log('login')
        if(res.data.access_token) {
         this.refreshTokenInterval =  setInterval(() => {
           console.log(res.data.access_token);
            this.refreshToken({refresh_token: res.data.access_token}).subscribe(res => {
              AuthService.log(res)
            })
          },300000); // TODO: This is not the right way but it will do for now
        }
      }),
      catchError(AuthService.handleError))
  }

  signUp(signUpPayload: any): Observable<any> {
    return this.http.post(BASE_URL + endpoints.REGISTER, signUpPayload, HTTP_OPTIONS).pipe(
      tap(_ => AuthService.log('register')),
      catchError(AuthService.handleError));
  }

  refreshToken(refreshTokenData: any): Observable<any> {
    HTTP_OPTIONS.params = {
      grant_type: 'refresh_token',
      token_type: 'regular'
    } // Quaryparams 
    return this.http.post(BASE_URL + endpoints.LOGIN, refreshTokenData, HTTP_OPTIONS).pipe(
      tap((res: any) => {
        this.tokenService.removeAccessToken(); // Cleaning localstorage before storing new value
        this.tokenService.removeRefreshToken();  // Cleaning localstorage before storing new value
        this.tokenService.saveAccessToken(res.access_token); // Storing new value
        this.tokenService.saveRefreshToken(res.refresh_token); // Storing new value
      }),
      catchError(AuthService.handleError));
  }

  logOut(): void { // Loging out
    this.tokenService.removeAccessToken();
    this.tokenService.removeRefreshToken();
    setTimeout(() => {
      clearInterval(this.refreshTokenInterval); // Clearing interval
      this.router.navigate(['/auth/login']); //  Redirection to login page
    }, 1000);
  }
}

```

### Requester service
> RUN ```ng g s auth/Requester```

```Javascript
import {Injectable} from '@angular/core';

const ACCESS_TOKEN = 'access_token';
const REFRESH_TOKEN = 'refresh_token';

@Injectable({
  providedIn: 'root'
})
export class TokenService {

  constructor() {
  }

  getAccessToken(): string | null { // Getting access token from local storage
    return localStorage.getItem(ACCESS_TOKEN);
  }

  getRefreshToken(): string | null { // Getting refresh token from local storage
    return localStorage.getItem(REFRESH_TOKEN);
  }

  saveAccessToken(accessToken: string): void { // Saving access token from local storage
    localStorage.setItem(ACCESS_TOKEN, accessToken);
  }

  saveRefreshToken(refreshToken: string): void { // Saving refresh token from local storage
    localStorage.setItem(REFRESH_TOKEN, refreshToken);
  }

  removeAccessToken(): void { // Removing access token from local storage
    localStorage.removeItem(ACCESS_TOKEN);
  }

  removeRefreshToken(): void { // Removing refresh token from local storage
    localStorage.removeItem(REFRESH_TOKEN);
  }
}

```

### HTTP Interceptor

> RUN ```ng g interceptor shared/interceptors/ApiCallInterceptor```

```Javascript
import { Injectable } from '@angular/core';
import {
  HttpRequest,
  HttpHandler,
  HttpEvent,
  HttpInterceptor, HttpResponse, HttpErrorResponse
} from '@angular/common/http';
import {catchError, map, Observable, throwError} from 'rxjs';
import {Router} from "@angular/router";
import {AuthService} from "../../auth/auth.service";
import {TokenService} from "../../auth/token.service";

@Injectable()
export class ApiCallInterceptor implements HttpInterceptor {

  constructor(
    private router: Router,
    private authService: AuthService,
    private tokenService: TokenService) {}

  intercept(request: HttpRequest<unknown>, next: HttpHandler): Observable<HttpEvent<unknown>> {
    const accessToken = this.tokenService.getAccessToken();
    const refreshToken = this.tokenService.getRefreshToken();

    if(accessToken) {
      request = request.clone({
          setHeaders: {
            Authorization: "Bearer " + accessToken
          }
        }
      ) // Adding Bearer tocken to the request
    }

    if(!request.headers.has('content-type')) {
      request = request.clone({
        setHeaders: {
          'content-type': "application/json"
        }
      })  // Adding content-type to the request
    }
    return next.handle(request).pipe( // Handleing request
      map((event: HttpEvent<any>) => {
        if(event instanceof HttpResponse) {
            console.log("event => ", event) // Loging out the event
        }
        return event;
      }),
      catchError((error: HttpErrorResponse) => {
        if(error.status === 401) {
          if(error.error.error === 'invalid_token') {
            this.authService.refreshToken({ // If tocken is not valid then trying for refresh token
              refresh_token: refreshToken
            }).subscribe(() => {
              location.reload();
            })
          } else {
            this.router.navigate(['login']).then( _=> {
              console.log('Redirecting to login page');
            })
          }
        }
        return throwError(error);
      })
    );
  }
}

```
> Don't Forget To Update AppModule 
> ```Javascript
> @NgModule({
>  declarations: [
>    AppComponent
>  ],
>  imports: [
>    BrowserModule,
>    AppRoutingModule,
>    BrowserAnimationsModule,
>    HttpClientModule
>  ],
>  providers: [{
>    provide: HTTP_INTERCEPTORS,
>    useClass: ApiCallInterceptor,
>    multi: true // To allow multiple interceptors support
>  }],
>  bootstrap: [AppComponent]
> })
> export class AppModule { }
> ```

>  _NOTE: There are too many ways to do this but i found it easy_
