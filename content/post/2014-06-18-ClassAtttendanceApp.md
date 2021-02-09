---
title:  "Online Class Attendance App using Flask/MongoDB"
date:   2014-06-18
description: Sometimes you just gotta do it yourself.
url: ClassAtttendanceApp/
---

![Classes HomePage](/images/classattendance/Classes.png)

# Motivation

My friends and I teach a parkour class at a local gymnastics gym, which we organize ourselves.  We needed a solid method for logging attendance.  There were many free solutions, but they all followed the attendance model similar to school classroom attendance: each class has assigned students and you mark attendance by checking them off as absent, tardy, attended, excused, etc.  This didn't fit our model because we ran our class on a come-as-you-can basis.  For lack of a better solution we used an Excel Google Doc or just left our attendance logs on paper.

Fast forward: not long ago I decided to take MongoDB's free [on-line developer's course](https://university.mongodb.com/).  Part of the class used a simple blog example using the [Bottle framework](http://bottlepy.org/docs/dev/index.html) in Python, which made me think that a Python framework / MongoDB application would suit itself extremely well to what we wanted.  So I set to working using these tools to write my own simple class attendance application in my free time.

# Overview

My final application is pieced together using the following components:

- The [Flask micro-framework](http://flask.pocoo.org/) to host the web interface, with [Jinja2](http://jinja.pocoo.org/docs/) to template web pages based on Python objects.
- Flask's [Oauth plug-in](https://pythonhosted.org/Flask-OAuth/) to enable using Google OAuth for simple user authentication.  
- [PyMongo](http://api.mongodb.org/python/current/) to link the web framework to the [MongoDB](http://www.mongodb.org/) database.
- [Pure CSS](http://purecss.io/) to simplify making the pages look 'pretty.'
- [JQuery UI](http://jqueryui.com/) simplified adding handy interface features such as a javascript calendar to select class dates as well as student name auto-completion during attendance logging.
- [OpenShift](http://www.openshift.com/) by RedHat for a free hosting solution easily managed by git and ssh.

# Schema

The MongoDB database schema uses four collections:

1. authentication: contains Google Oauth credentials as well as the secret key for the application.
2. coaches: contains all coaches as well as their e-mails to validate against Google Oauth login.
3. students: holds all details about students such as emergency contacts, e-mail, birthdays, etc.
4. classes: Contains all records of classes.  Each record contains a date and associates to the record for its indicated coach.  Within the record is a list of attendances with purchase info if applicable and an association to a student record for the attended student.

The schema for the student and classes collections are shown below.

<pre><code class="json"># Students
{
    "_id" : ObjectId("53a0ee5b798a010258167aa7"),
    "firstname" : "Eunice",
    "lastname" : "Anderson",
    "gender" : "female",
    "email" : "altheabanks@zensor.com"
    "dob" : ISODate("1997-09-25T00:00:00Z"),
    "emergencyphone" : "+1 (949) 511-3347",
    "emergencycontact" : "Althea Banks",
}
</code></pre>

<pre><code class="json"># Classes
{
    "_id" : ObjectId("53a0f734798a0110dc74abc1"),
    "date" : ISODate("2014-01-01T00:00:00Z"),
    "coach" : ObjectId("53a0ef10798a010f40e8cabc"),
    "type" : "beginner"
    "attendance" : [
        {
            "student" : ObjectId("53a0ee5b798a010258167ab6")
        },
        {
            "payment" : {
                "amount" : 15,
                "method" : "credit",
                "purchased" : "drop in"
            },
            "student" : ObjectId("53a0ee5b798a010258167ab5")
        },
        {
            "payment" : {
                "amount" : 0,
                "method" : "punched",
                "purchased" : null
            },
            "student" : ObjectId("53a0ee5b798a010258167aaf")
        },
        ...
    ]
}
</code></pre>

![Schema Diagram](/images/classattendance/Schema.png)

# Screenshots

![Class Page](/images/classattendance/Class.png)
![Students Page](/images/classattendance/Students.png)
![Edit Student Page](/images/classattendance/edit_student.png)

# Demo and Source Code

<a href="http://classdemo-evargreen.rhcloud.com/" class="btn btn-success">Live Demo</a>    <a href="https://github.com/serdmanczyk/parkour_class_attendance_app" class="btn btn-success">GitHub Repository</a>

Editing on the demo is disabled, but forms buttons will still follow through to their final destinations.  Authorization through Google is still used, no user info is maintained, just a session token.  All student and coach data is random sample data generate using [json-generator](http://www.json-generator.com/).

All source code including provisions for OpenShift is available on GitHub.  Fork it and build it into your own class app!

# Summary

This is just a quick base application I programmed for fun to get practice using new frameworks and technologies.  Future work includes adding more feedback to users when errors happend server-side.  Currently there's no real error checking and only minor form validation.  If there's an exception flask returns a feedback page, if a submit fails the new data simply fails to show up.  I would also add a page to manage coaches as well as an ability to view charts of attendance and revenue over time.  I may use a future post to demonstrate using MapReduce with MongoDB to retrieve related data from two collections in one call, similar to a SQL Join.
