# Angular reactive course (RXjs)


## Imperative Style Declaration
What is Imperative Style Declaration ?

> “Imperative programming is like how you do something, and declarative programming is more like what you do.”


```html
<div class="courses-panel">

    <h3>All Courses</h3>

    <mat-tab-group>

        <mat-tab label="Beginners">

          <mat-card *ngFor="let course of beginnerCourses" class="course-card mat-elevation-z10">

            <mat-card-header>

              <mat-card-title>{{course.description}}</mat-card-title>

            </mat-card-header>

            <img mat-card-image [src]="course.iconUrl">

            <mat-card-content>
              <p>{{course.longDescription}}</p>
            </mat-card-content>

            <mat-card-actions class="course-actions">

              <button mat-button class="mat-raised-button mat-primary" [routerLink]="['/courses', course.id]">
                VIEW COURSE
              </button>

              <button mat-button class="mat-raised-button mat-accent"
                      (click)="editCourse(course)">
                EDIT
              </button>

            </mat-card-actions>

          </mat-card>

        </mat-tab>

        <mat-tab label="Advanced">

          <mat-card *ngFor="let course of advancedCourses" class="course-card mat-elevation-z10">

            <mat-card-header>

              <mat-card-title>{{course.description}}</mat-card-title>

            </mat-card-header>

            <img mat-card-image [src]="course.iconUrl">

            <mat-card-content>
              <p>{{course.longDescription}}</p>
            </mat-card-content>

            <mat-card-actions class="course-actions">

              <button mat-button class="mat-raised-button mat-primary" [routerLink]="['/courses', course.id]">
                VIEW COURSE
              </button>

              <button mat-button class="mat-raised-button mat-accent"
                      (click)="editCourse(course)">
                EDIT
              </button>

            </mat-card-actions>

          </mat-card>

        </mat-tab>

    </mat-tab-group>



</div>
```

```typescript
import {Component, OnInit} from '@angular/core';
import {Course, sortCoursesBySeqNo} from '../model/course';
import {interval, noop, Observable, of, throwError, timer} from 'rxjs';
import {catchError, delay, delayWhen, filter, finalize, map, retryWhen, shareReplay, tap} from 'rxjs/operators';
import {HttpClient} from '@angular/common/http';
import {MatDialog, MatDialogConfig} from '@angular/material/dialog';
import {CourseDialogComponent} from '../course-dialog/course-dialog.component';


@Component({
  selector: 'home',
  templateUrl: './home.component.html',
  styleUrls: ['./home.component.css']
})
export class HomeComponent implements OnInit {

  beginnerCourses: Course[];

  advancedCourses: Course[];


  constructor(private http: HttpClient, private dialog: MatDialog) {

  }

  ngOnInit() {

    this.http.get('/api/courses')
      .subscribe(
        res => {

          const courses: Course[] = res["payload"].sort(sortCoursesBySeqNo);

          this.beginnerCourses = courses.filter(course => course.category == "BEGINNER");

          this.advancedCourses = courses.filter(course => course.category == "ADVANCED");

        });

  }

  editCourse(course: Course) {

    const dialogConfig = new MatDialogConfig();

    dialogConfig.disableClose = true;
    dialogConfig.autoFocus = true;
    dialogConfig.width = "400px";

    dialogConfig.data = course;

    const dialogRef = this.dialog.open(CourseDialogComponent, dialogConfig);

  }
}
```

Demo
> https://stackblitz.com/edit/reactive-angular-with-rxjs-1-imperative-style-declaration

## Understanding potential problems of a progaram written in Imperative Style

> http://callbackhell.com/

This type of callback hell is something that we want to avoid by using Obervable.

The potentaial problem with this component is there is too much login in this component. It means, the component knows alot about where the data comes from the component know that .the data comes from backend. The component knows how to do HTTP call and how to process the data that it receives. 

> Impeative data contais more duplicate data to avoid this duplication we use Observables.
 
 ```typescript
 
  beginnerCourses: Course[];

  advancedCourses: Course[];
 ```

A bove code type called "**Immutable**" type declaration. Keeping data in these local mutable state variables is potentially problematic if we make a change to  these data here locally in the component. There is no way for the rest of the application to know that this data was modified the data.

 Modifying the directly at the level of a component by **mutating** a member variable might introduce certain dependencies with other components of the applicataion making our program harder. 

## Stateless Observable-based service

**Statelesss Observable Service**, which is the first design pattern we're going to cover.

The service that we are about to write is going to be a stateless observable service  

```typescript
import { Injectable } from "@angular/core";
import { HttpClient } from "@angular/common/http";
import { Course } from "../model/course";
import { Observable } from "rxjs";
import { map } from "rxjs/operators";

@Injectable()
export class CourseService {
  constructor(private http: HttpClient) {}

  loadAllCourses(): Observable<Course[]> {
    return this.http
      .get<Course[]>("/assets/courses.json")
      .pipe(map(res => res["payload"]));
  }
}
```
 
> Observable<Course[]>, we are going back to return to the course array of observable.

>get<Course[]>("/assets/courses.json"), get expect course[] type of data from the get call

>map operator : RXjs operator are chainable functions that allow us to quickly combine different observable is in obtain to obtain different type of results.

example: map(x => 10 * x), 
let x is 1 return 10, 2 return 20, 3 return 30 and so on.


## Consuming Observable-based services using Angular async Pipe

```html
<div class="courses-panel">
  <h3>All Courses</h3>

  <mat-tab-group>
    <mat-tab label="Beginners">
      <mat-card *ngFor="let course of (beginnerCourses$ | async)" class="course-card mat-elevation-z10">
        <mat-card-header>
          <mat-card-title>{{course.description}}</mat-card-title>
        </mat-card-header>

        <img mat-card-image [src]="course.iconUrl" />

        <mat-card-content>
          <p>{{course.longDescription}}</p>
        </mat-card-content>

        <mat-card-actions class="course-actions">
          <button
            mat-button
            class="mat-raised-button mat-primary"
            [routerLink]="['/courses', course.id]"
          >
            VIEW COURSE
          </button>

          <button
            mat-button
            class="mat-raised-button mat-accent"
            (click)="editCourse(course)"
          >
            EDIT
          </button>
        </mat-card-actions>
      </mat-card>
    </mat-tab>

    <mat-tab label="Advanced">
      <mat-card *ngFor="let course of (advancedCourses$ | async)" class="course-card mat-elevation-z10">
        <mat-card-header>
          <mat-card-title>{{course.description}}</mat-card-title>
        </mat-card-header>

        <img mat-card-image [src]="course.iconUrl" />

        <mat-card-content>
          <p>{{course.longDescription}}</p>
        </mat-card-content>

        <mat-card-actions class="course-actions">
          <button
            mat-button
            class="mat-raised-button mat-primary"
            [routerLink]="['/courses', course.id]"
          >
            VIEW COURSE
          </button>

          <button
            mat-button
            class="mat-raised-button mat-accent"
            (click)="editCourse(course)"
          >
            EDIT
          </button>
        </mat-card-actions>
      </mat-card>
    </mat-tab>
  </mat-tab-group>
</div>
```

> *ngFor="let course of (beginnerCourses$ | async)"

Without async pipe we were not able to do complete reactive programming in angular. The Async pipe subscribe to the beginner courses observable and this could make the values emitted by the observable available here to our templates. The Async pipe will **not only subscribe to the observable and get its value available to the template** but also **when the component gets destroyed these async pipe**, will also **unsubscribe from the observable if it is not yet completed** or effort out. This way we will avoid any potential memory leaks in our component. The also means that in our home compoent we will never have to take care of and subscribing from these observable manually.


```typescript
  beginnerCourses$: Observable<Course[]>;

  advancedCourses$: Observable<Course[]>;

 const courses$ = this.courseService
      .loadAllCourses()
      .pipe(map(courses => courses.sort(sortCoursesBySeqNo)));

    this.beginnerCourses$ = courses$.pipe(
      map(courses => courses.filter(course => (course.category = "BEGINNER")))
    );
    this.advancedCourses$ = courses$.pipe(
      map(courses => courses.filter(course => (course.category = "ADVANCED")))
    );
```

## Avoid Angular duplicate HTTP requests with the RXjs shareReplay operator

[ Duplicate Requests]: (https://github.com/padamaranagen/reactive-angular-course/blob/main/Angular%20duplicate%20requeset.PNG)

We have two HTTP requests here, why this is happening and how to avooid it.
 If you see above home component code, We have created courses observable. This is built uinsg the loadAllCourses from the service (course.service.ts), which in turn uses the angular HTTP client in order to produce an angular HTTP observable that send back to the view. These types of observables are HTTP observables have a very specific behavior. 

 In our template we have two Async pipe which we are using beginner and advance, because of this pipe it generate two HTTP call.

 We can use resolve the issue by using a method in RXjs called 
 ```typescript 
  shareReplay() 
 ```  
 at service level.

[No Duplicate]:
(https://github.com/padamaranagen/reactive-angular-course/blob/main/After-apply-sharereplay.PNG)







