---
layout: post
title: BlocTime
thumbnail-path: "img/blocflix.png"
short-description: BlocTime is a time management system based on the Pomodoro technique using AngularJS and Firebase.

---

{:.center}
![]({{ site.baseurl }}/img/blocflix.png)

## Introduction

BlocTime. One of the projects I worked on during my Bloc frontend web developer apprenticeship, uses Firebase and AngularJS to provide a Pomodoro technique inspired time management application. This program aims to increase work productivity and quality. The user can add/remove tasks, and set a 25 minute work timer, followed with a 5 minute break timer. Every 4 work sessions, the 5 minute break will then change to a long 30 minute break.

## Problem

The following are the user stories I was given:
1. Start and reset a 25 minute work session.
1. Start and reset a 5 minute break after each completed work session.
1. Start and reset a longer, 30 minute break after every 4 completed work sessions.
- Displaying the live timer during sessions.
- Creating "beep" sound after every completed session.
- The ability to record completed tasks, and link the to a firebase database.
- Displaying the tasks in reverse chronological order. 

## Solution

Following the given wireframes, I first created a button and displayed a time using {{ }} markup, linking it to the time variable in my home controller. I put most of the logic for this project inside the home controller.

### Track time 

In the home controller, I first created: 
Three constants:
  * WORK_TIME
  * BREAK_TIME
  * LONG_BREAK_TIME
(They're pretty self explanatory)

Two private variables:
  * timer
  * completedSessions (initialized to = 0)

Three public variables:
  * this.onBreak (initialized as false)
  * this.time (initialized as WORK_TIME)
  * this.buttonName (initialized as "start")

These variables gave my timer its frame. Now, time for some logic...

I was going to need to set the timer for the start of the work session, break session, and long break session. I've already created the constants, so it only made sense to create a function to help me handle this. The `setTimer()` function handles this logic. If this function is called, it first stops the current timer (if it's running and I forgot to stop it already). It then determines the timer's duration based on the current buttonName (i.e. "start"), `this.onBreak`, and `completedSessions`. The logic is fairly straightforward, setting `this.time` to equal one of the three constant times based on the information. 

A `countdown()` function was then created to...well..countdown the timer! I had a bit of trouble getting this function to work, and realised that the scope of using `this` was the cause. So, I then created a private variable `self = this`, which allowed for inner functions to gain access to `this`. The countdown function decremented the time variable, and set the buttonName to equal "Reset". Once the timer had run out (i.e. `self.time <= 0`), I check to see if it wasn't a break session to know whether or not to increment `completedSessions`, swap the break statuses (`onBreak = !onBreak), and reset the timer by calling (setTimer).

I "injected" Angular's `$interval` service into my home controller and created the public function `startResetTimer()`. The function starts the countdown timer and handles resetting the timer if needed.

```javascript
this.startResetTimer = function(){
    if(self.buttonName === "Start"){
        $interval.cancel(timer);
        timer = $interval(countdown, 1000);
    }else{
        setTimer();
    }
};
```

The function first checks that the timer hasn't started via the buttonName (this will be set to "reset" if started), and if it's true, it will stop the current timer, and set the timer to call the countdown function every second (or 1000 milliseconds). If the timer has already begun, and this function is called, the function should set the timer via `setTimer()`.

Lastly, I went back to the view on added an ng-click directive to the button. `ng-click="home.startResetTimer()"`.

And voila! A working timer. 

### Filtering time

The timer worked perfectly, and would be displayed to the view. However, it was in seconds...so 5 minutes would be 300, 299, 298... you get the point. I don't really like having my time shown to me in seconds.. imagine trying to use seconds to relay how much time left you have...so, I "borrowed" a piece of code provided by Bloc, the timecode.js filter which pretty much filters time. It takes in a time in seconds, and outputs the time in mm:ss format. The longest time I need for this timer is 30 minutes, so I don't need an hour filter for now. Then, applying the filter to the view: `{{ home.time | timecode }}`, I now have time dispalyed correctly. 

### Play a ding

This was fairly easy, using the Buzz Javascript library, I created a new buzz object in the home controller, and played the sound every time the countdown hit 0. 

### Tasks & Syncing with firebase

This was the first time I've even heard of firebase, so it is still quite foreign to me. I instantiated the firebase service in a Tasks.js service, and abstracted it into a factory which handles all of the data management. I created a `tasks` variable which stored an array of tasks from Firebase queried using $firebaseArray. `var tasks = $firebaseArray(firebaseRef);`

After establishing the relationship with firebase, I created three functions: add, delete, and all. All functions are pretty self expanatory. I then linked up these functions to the view: 
..* add via ng-submit on the add task form

In the home controller, I created a public addTask function which took the input from the view's input box (the task name, and added them to firebase. I also created a public `taskList` variable which allowed the tasks to be shown in the view via ng-repeat. 

### Present tasks in reverse chronological order

Now, this part took a bit of researching. When creating the add function in the tasks service, I created a date along with the task, setting it to `Firebase.ServerValue.TIMESTAMP`. Then, when calling ng-repeat on the home controller's taskList in the view, I filtered it using `home.taskList | orderBy: '-date'`. The "-" before date gave it the "reverse" it needed. 

## Results

I used grunt to run the timer in a browser (chrome). I tested every function by refreshing the page, and checking the functionality/ to see if anything broke. I obviously didn't wait 30 minutes every work session, and set the timer's to shorter spans. (I think I forgot to change them back...)

## Conclusion

This was the first Bloc project I completed without guidance. A lot of the other projects provided chunks of code. However, this one was more of an experimental project. In the end, I managed to complete the user stories provided, along with an added remove function. 

