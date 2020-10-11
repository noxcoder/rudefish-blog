---
title: Hacker-Camp
date: 2020-10-11T00:35:20.578Z
image: ""
tags:
  - DamCTF
  - Web
  - HackerCamp
draft: false
---
**Challenge**

![](/images/hacker-camp-damctf.png "Challenge description")

**Description**

By following the provided link `https://hacker-camp.chals.damctf.xyz/`we are presented with a login page

![](/images/home-page.png "OSUSEC Login Page")

So we have a PHP application requesting for a username and password. We did some directory brute forcing with gobuster but didn't get much result, so we proceeded to bypass the login page using SQLi. Providing `' or 1=1#` as the username and any password, we are logged in as the user `rhonda.daniels`

![](/images/sqli.png "Login page bypass")

![](/images/rhodes.png "Logged in as rhodes.daniels")

Then we have a list of students, looks like `rhonda.daniels` is a staff

![](/images/students.png "List of students")

Going back to the challenge description, we notice that we cannot find our target student Natasha Drew on the list of students and also we do not have a way of updating grades, so we keep digging. Looking through the source page we discover the following:

![](/images/table.png "Student name base64 encoded")

there is a base64 encoding on each student record which translates to the the format `lastname_firstname` and also a script that shows the admin status of the user, turns out our user is not an admin after all.

![](/images/admin_script.png "Check admin")

The next line of action would be to make our user an admin or find a way to perform admin operations. Looking further at the source, we found a JS file at `/assets/js/app.min.js` with the following code:

```javascript
(function(s, objectName) {
    setupLinks = function() {
        if (s.admin) {
            var sl = document.getElementsByClassName("student-link");
            for (i = 0; i < sl.length; i++) {
                let name = sl[i].innerHTML;
                sl[i].style.cursor = 'pointer';
                sl[i].addEventListener("click", function() {
                    window.location = '/update-' + objectName + '/' + this.dataset.id;
                });
            }
        }
    }
    ;
    updateForm = function() {
        var submitButton = document.getElementsByClassName("update-record");
        if (submitButton.length === 1) {
            submitButton[0].addEventListener("click", function() {
                var english = document.getElementById("english");
                english = english.options[english.selectedIndex].value;
                var science = document.getElementById("science");
                science = science.options[science.selectedIndex].value;
                var maths = document.getElementById("maths");
                maths = maths.options[maths.selectedIndex].value;
                var grades = new Set(["A", "B", "C", "D", "E", "F"]);
                if (grades.has(english) && grades.has(science) && grades.has(maths)) {
                    document.getElementById('student-form').submit();
                } else {
                    alert('Grades should only be between A - F');
                }
            });
        }
    }
    ;
    setupLinks();
    updateForm();
}
)(staff, 'student');
```

Following the code, after it checks if the user is an admin it obtains the student record with the "student-link" class and then extract the id which is the base64 encoding of the format lastname_firstname we saw earlier and the location is at '`/update-' + objectname + '/' + this.dataset.id`. We can obtain the id for each student, then the objectname is 'student' by looking that the app.min.js, it's now time to see if we can update a student's record. By picking the first student on the table `Brett, Nancie` with id `TmFuY2llX0JyZXR0` with when decoded translates to `Nancie_Brett`, we can construct our url to be `https://hacker-camp.chals.damctf.xyz/update-student/TmFuY2llX0JyZXR0`

![](/images/brett.png "Brett, Nancie")

Nice. So we know to update we just need the id of the target student. Our student is Natasha Drew so let's make a base64 encoding using the format lastname_firstname, that is Drew_Natasha and try to update her record, `https://hacker-camp.chals.damctf.xyz/update-student/RHJld19OYXRhc2hh`

![](/images/invalid.png "Student record invalid")

But then we get a student record invalid. Okay let's change the format and use the base64 encoding of Natasha_Drew, `https://hacker-camp.chals.damctf.xyz/update-student/TmF0YXNoYV9EcmV3`

![](/images/drew.png "Natasha Drew records")

Now we have her records (poor grades, explains why she couldn't attend Hacker Camp xD). Let's upgrade her records to all A's.

![](/images/finale.png "Update records")

 And we have the flag

FLAG: **dam{n0w_w3_c4n_h4ck_th3_pl4n3t}**