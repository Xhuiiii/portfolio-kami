---
layout: post
title: BlocTime - Track time
---

In the home controller, I first created: hi
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

{% highlight javascript %}
this.startResetTimer = function(){
    if(self.buttonName === "Start"){
        $interval.cancel(timer);
        timer = $interval(countdown, 1000);
    }else{
        setTimer();
    }
};
{% endhighlight %}

The function first checks that the timer hasn't started via the buttonName (this will be set to "reset" if started), and if it's true, it will stop the current timer, and set the timer to call the countdown function every second (or 1000 milliseconds). If the timer has already begun, and this function is called, the function should set the timer via `setTimer()`.

Lastly, I went back to the view on added an ng-click directive to the button. `ng-click="home.startResetTimer()"`.

And voila! A working timer. 