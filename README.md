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

https://stackblitz.com/edit/reactive-angular-with-rxjs-1-imperative-style-declaration




