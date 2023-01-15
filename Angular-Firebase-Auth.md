# VISIT https://developers.google.com/codelabs/building-a-web-app-with-angular-and-firebase#9

## Step 1 - Install the Firebase Package
      *Firebase team offers the package @angular/fire, which provides integration between the two technologies.
      *Run This Command- ng add @angular/fire
      *Choose from Different options

## Step 2 - add the Firebase configuration to your environment
      * You can find your project configuration in the Firebase Console
           --Click the Gear icon next to Project Overview.
           --Choose Project Settings.
           --register your application and make sure you enable "Firebase Hosting":
           --npm install firebase
           --After you click "Register app", you can copy your configuration into src/environments/environment.ts:
           --initialize Firebase and begin using the SDKs for the products you'd like to use.

## Step 3 - Install Firebase CLI
          --To host your site with Firebase Hosting, you need the Firebase CLI (a command line tool).
          --npm install -g firebase-tools

## Step 4 - Making Http Requests to Firebase i.e POST,GET,PUT,DELETE API's

           --import { HttpClientModule } from '@angular/common/http'; in imports[] of app.module.ts
           --INJECT httpClient to call the API inside the Services File OR any Other File.
           --Create a interface for RequestData and ResponseData
           --Inside Firebase Set Rules to True For Testing --    
                                     {
                                       "rules": {
                                            ".read": "true", 
                                             ".write": "true",  
                                          }
                                      } 
                
## Step 5 - Add Authentication with Firebase
           ----Inside Firebase Set Rules to auth != null For Authentication --    
                                     {
                                       "rules": {
                                            ".read": "auth != null", 
                                            ".write": "auth != null",  
                                          }
                                      }          
         --Go to Authentication Section in Firebase Console and ENABLE Methods as Required i.e Email/Password or Google....
         --Now We need to HIT certain END-POINTS of FIREBASE to CREATE or LOGIN the Users.
         --VISIT https://firebase.google.com/docs/reference/rest/auth
                        --API Usage:- Sign up with email/password && Sign in with email/password
         --Copy the EndPoint- https://identitytoolkit.googleapis.com/v1/accounts:signUp?key=[API_KEY]
         --Create Request and Response interfaces if Needed
         --Http Request to Firebase
         --Do Error Handling With Firebase i.e Common error Codes


### SIGN UP and SIGN IN with PASSWORD ###
                   ## CREATING A SIGN UP and SIGN IN  HTML FORM -- user.component.html -- REACTIVE FORM ##

<div class="user__auth">
        <div class="user__container" *ngIf="switchMode">
                <h2 style="color: #5D5179;">Create your Account</h2>
                <form class="user__signUpForm" 
                #userSignUp="ngForm" 
                (ngSubmit)="onUserSignUp(userSignUp)">
            
                <!-- <input 
                class="form__input"
                type="text"
                id="name" 
                placeholder="Enter Name"
                name="name"
                ngModel
                #name = "ngModel"
                required
                name
                /> -->
            
                <!-- Email Field -->
                <input 
                class="form__input" 
                placeholder="Enter your email"
                type="email"
                id="email"
                ngModel 
                name="email"
                #email = "ngModel"
                required
                email 
                />
            
                <!-- Password Field -->
                <input 
                class="form__input" 
                placeholder="Password"
                type="password"
                id="password" 
                ngModel
                name="password"
                #password = "ngModel"
                required 
                minlength="6" />
                
                <button 
                [disabled]="!userSignUp.valid"
                class="signup__button" 
                type="submit" >Sign Up</button>

                <!-- Display the wrong Credentials Error -->
                <h5 style="color: red;" *ngIf="userErrorMessage">{{ userErrorMessage }}</h5>
                        
                <!-- To Toggle Between Sign-Up and Sign-In -->
                <button 
                class="redirect__button"
                (click)="redirectToSignIn()">Already have an account? Click to Sign In</button>        
            
            </form>
        </div>

        <!-- Signin Conatiner -->
    <div class="signin__container" *ngIf="!switchMode">
        <!-- signIn Form -->
        <form 
        class="user__signinForm" 
        #userSignInForm="ngForm" 
        (ngSubmit)="onUserSignIn(userSignInForm)">
            <!-- Email Field -->
            <input 
            class="form__input" 
            placeholder="Email"
            type="email"
            id="email"
            name="email"
            ngModel 
            #email = "ngModel"
            required
            email
            />

            <!-- Password Field -->
            <input 
            class="form__input" 
            placeholder="Password"
            type="password"
            id="password"  
            name="password"
            ngModel
            #password = "ngModel"
            required 
            minlength="6" />
            
            <!-- The Link for Forgotten your password -->
            <a class="forgot__password"><strong>Forgotten your password?</strong>
                <fa-icon class="forgot__icon" [icon]="forgotPasswordIcon"></fa-icon></a>
            
                <button 
            [disabled]="!userSignInForm.valid"
            class="signin__button" 
            type="submit">Sign In</button>

            <!-- Display the wrong Credentials Error -->
            <h5 style="color: red;" *ngIf="userErrorMessage">{{ userErrorMessage }}</h5>

            <!-- To Display the Error -->
            <!-- <h2 style="color: red; font-family: Arial, Helvetica, sans-serif;">{{ sellerErrorMessage }}</h2> -->

            <p style="color: rgb(134, 30, 50);font-size: large;">
                By signing-in you agree to Madani.in Conditions of Use & Sale. Please see our Privacy Notice, our Cookies Notice and our Interest-Based Ads Notice. 
            </p>

            <!-- To Toggle Between Sign-Ip and Sign-Un -->
            <h2>Don't have an account?</h2>
            <button 
            class="redirect__button"
            (click)="redirectToSignUp()">Sign Up</button>
        </form>
    </div>
</div>


                  ## CREATING SERVICE FOR USER SIGN UP and SIGN IN -- user.service.ts FILE ##

import { HttpClient } from '@angular/common/http';
import { EventEmitter, Injectable } from '@angular/core';
import { Router } from '@angular/router';

// For Handling Firebase Errors
import { catchError } from 'rxjs/operators';
import { throwError } from 'rxjs';

// Firebase user Sign up Response Data
interface FirebaseSignUpResponseData {
  idToken: string;
  email: string;
  refreshToken: string;
  expiresIn: string;
  localId: string;
}

// Firebase user Sign in Response Data
interface FirebaseSignInResponseData {
  idToken: string;
  email: string;
  refreshToken: string;
  expiresIn: string;
  localId: string;
  registered: boolean;
}

@Injectable({
  providedIn: 'root'
})
export class UserService {

  // To show the Error Messsage for Wrong Emial,Password we need EventEmiiter
  userAuthErrorMessage = new EventEmitter<boolean>(false);

  // it is Important to Create a INSTANCE/INJECT httpClient to call the API
  constructor( private http: HttpClient,
              // Inject Router to redirect to HomePage
              private router: Router ) { }

  // This is the service for User to Sign Up --call API
  userSignUp(email: string, password: string) {
       // Send Http Request to Firebase
       // Firebase Endpoint
      //  We Attach the Response Data Interface 
      return this.http.post<FirebaseSignUpResponseData>('https://identitytoolkit.googleapis.com/v1/accounts:signUp?key=AIzaSyAepA2tOFa9U6ii_-1o4KLPq2yM1BaUY64', {
        email: email,
        password: password,
        returnSecureToken: true,
      })
      // Error Handling With Firebase With the Help of RxJS Operator i.e catchError
      .pipe(catchError( errorRes => {
        let errorMessage = 'An error occurred...';
        // User Network Errors
        if(!errorRes.error || !errorRes.error.error) {
          return throwError(errorMessage);
        }
        switch (errorRes.error.error.message) {
          case 'EMAIL_EXISTS':
            errorMessage = 'The email address is already in use by another account.'
            break;
          case 'OPERATION_NOT_ALLOWED':
            errorMessage = 'Password sign-in is disabled for this project.'
            break;
          case 'TOO_MANY_ATTEMPTS_TRY_LATER':
            errorMessage = 'We have blocked all requests from this device due to unusual activity. Try again later.'
            break;
        }
        return throwError(errorMessage);
      }))
  }

  // This service will Sign In the User
  userSignIn(email: string, password: string) {
    // console.log({user});
     // Send Http Request to Firebase
       // Firebase Endpoint
      //  We Attach the Response Data Interface 
      return this.http.post<FirebaseSignInResponseData>('https://identitytoolkit.googleapis.com/v1/accounts:signInWithPassword?key=AIzaSyAepA2tOFa9U6ii_-1o4KLPq2yM1BaUY64', {
        email: email,
        password: password,
        returnSecureToken	: true
      })
       // Error Handling With Firebase With the Help of RxJS Operator i.e catchError
       .pipe(catchError( errorRes => {
        let errorMessage = 'An error occurred...';
        // User Network Errors
        if(!errorRes.error || !errorRes.error.error) {
          return throwError(errorMessage);
        }
        switch (errorRes.error.error.message) {
          case 'EMAIL_NOT_FOUND':
            errorMessage = 'There is no user record for this email.'
            break;
          case 'INVALID_PASSWORD':
            errorMessage = 'The password is invalid.'
            break;
          case 'USER_DISABLED':
            errorMessage = 'The user account has been disabled by an administrator.'
            break;
        }
        return throwError(errorMessage);
      }))
  }

  // For Auto-Login
  userAuthReload() {
    if(localStorage.getItem('userData')) {
      // this.router.navigate(['/']);
    }
  }
}


                       ## CONNECTING THE SERVICE FILE IN User Component -- user.component.ts FILE ##

import { Component, OnInit } from '@angular/core';
import { UserService } from '../user-services/user.service';
import { HttpClient } from '@angular/common/http';
import { NgForm } from '@angular/forms';
import { Router } from '@angular/router';

@Component({
  selector: 'app-user',
  templateUrl: './user.component.html',
  styleUrls: ['./user.component.css']
})
export class UserAuthComponent implements OnInit {

  // To Display the Error Message
  userErrorMessage: string = '';

  // To switch user between SignUp and Sign In
  switchMode = true;

  // Inject userService to use the service Functionality and Methods for API
  constructor( private userService: UserService,
             private http: HttpClient, private router: Router ) {}

  ngOnInit(): void {
    this.userService.userAuthReload();
  }

  // The Function for User to Sign Up
  onUserSignUp(form: NgForm) {
    // console.log({data});
    // call the Method userSignUp from userService
    // Store the email and Password Values First
    // Check Condition for Form
    if(!form.valid) {
      return;
    }
    const email = form.value.email;
    const password = form.value.password;
    this.userService.userSignUp(email, password).subscribe( resdata => {
      console.log(resdata);
    }, errorMessage => {
      // Handling the error Coming From the UserService
      this.userErrorMessage = errorMessage;
    });
    form.reset();
  }

   // This will Redirect from SignIn to SignUp
   redirectToSignUp() {
    this.switchMode = true;
  }

  // This will Redirect from SignUp to SignIn Page
  redirectToSignIn() {
    this.switchMode = false;
  }

  //
  onUserSignIn(form: NgForm) {
    // console.log({data});
    // call the Method userSignIn from userService
    // Store the email and Password Values First
    // Check Condition for Form
    if(!form.valid) {
      return;
    }
    const email = form.value.email;
    const password = form.value.password;
    this.userService.userSignIn(email, password).subscribe( resdata => {
      console.log(resdata);
    }, errorMessage => {
      console.log(errorMessage);
      this.userErrorMessage = errorMessage;
    })
    // this.router.navigate(['/']);
  }

}

### User SIGN OUT METHOD -- Initialized in the HEADER Component Sign Out Button ### 
 // This will SignOut the User
  userSignOut() {
    localStorage.removeItem('userData');
    this.router.navigate(['/']);
    // this.switchMode = 'default';
  }
