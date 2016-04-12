Artifact 3 : Finding an issue : Redux
-------------------------------------

After getting nowhere trying to compile the module for the previous issue, I decided to hit the Jira tracker again looking for
something I could actually do.

Since I was having trouble compiling the Reference Application module, I decided to try tickets in other OpenMRS modules.
After playing around with the radiology module, I found I didn't understand it very well. I didn't want to spend a lot of
time learning how to use the program, especially as school was ramping up.

Around mid-March, a new curated issue appeared involving unit testing (https://issues.openmrs.org/browse/RA-980). Although the ticket
says it was created at the end of December last year, I didn't remember seeing it before. Maybe the `curated` tag was added mid-March.

I figured updating old tests was a good way to learn the code base. Having browsed through the repository before, I noticed a lot of old
code that was probably dead already. When building OpenMRS core application previously, I had noted the large number of unit test
failures.

However, the unit tests were meant for the Reference Application (the new and improved OpenMRS 2.0 or somesuch). I figured I would
find some way to work through it -- maybe even ask about it on the forums! But after closer reading, the repository wasn't
`openmrs-module-referenceapplication` but `openmrs-distro-referenceapplication`. The former is the source code for the module while
the latter seems to include a Vagrant box that runs the reference application (though I never used this).

The ticket seemed to be meant for people brand new to the OpenMRS project and even had a wiki page (https://wiki.openmrs.org/display/docs/Automated+Testing+Guidelines)
which detailed exactly what had to be done. Even so, I was wary of creating a new ticket with a subtask in case I had trouble either
getting the application to work or if I would have too much trouble sifting through the code and understanding it. I'm far from a Java
expert after all.

Luckily, building the project was rather painless though VERY lengthy. It only required `maven` which I had to install 
for the core application. The build process was very simple:
1. `git clone` the repo
2. `mvn clean install`

![A typical build](https://raw.githubusercontent.com/adfyong/openmrs-core/blog/build.png)
*I quickly learned to @Ignore all of the other tests to save a minute per compile*

Testing for the OpenMRS Reference Application is a rather humourous affair. It's not required to have your own copy as the unit tests
operate on a live development server. Using Firefox, the tests automagically type and click links in the browser and verify the output.
Because of the network requirement and the number of links being clicked, each test takes between 10 and 30 seconds to perform.
Most of the tests were set on `@Ignore` for one reason or another because the evolution of the application had left the unit tests behind.

I decided on a relatively innocuous-looking test called `EditDemographicTest` which involved editing a Patient's name,
gender, and date of birth and verifiying that the results were saved properly. After tracing its intended operation (it wasn't working
at the time), I felt confident that I would be able to handle it. I went back to Jira to claim the issue and make sure no one would
be able to steal it from me which is when the bug tracking server became unresponsive. After trying periodically for 20 to 30 minutes,
I decided to leave it for another day.

Thanks to a deluge of schoolwork I forgot about the task until after the final week. When I returned to OpenMRS's Jira, I was
pleasantly surprised to see that no one had taken the test. By this time, there were several other pull requests made
to update unit tests, so I took some time to read through their code changes and noted what the project maintainers didn't like.

For some reason, I didn't create the ticket at this time and I forgot to do so until most of the work on the ticket was done.
Nevertheless, I got around to creating an account on openmrs.org and creating the ticket, which can be found here:

https://issues.openmrs.org/browse/RA-1110

(In case you're curious, the edit on the comment is because I accidentally pasted the link to the ticket instead of the PR.)



Artifact 4 : Look, ma! A pull request!
--------------------------------------

Despite picking an "easy" test, I soon ran into a number of issues that were specific to my choice. I can be a bit stubborn
so I opted not to hop on IRC and worked out the issue myself. In the end, I learned a lot about the testing framework
and could probably update all of the rest of the tasks by myself. (Maybe if I don't have any summer projects of my own to do...)

The most time-consuming issue came about speicifically because of the issue I picked. The form used when editing a patient is
really a smaller version of the one used when registering a patient, so the previous tester used the same `Page` testing framework
for the edit page. I figured this was a reasonable thing to do to avoid code duplication.

Little did I know that the new testing framework used page URLs to figure out when a page has loaded. I didn't realize this until
I switched from Sublime Text to IntelliJ IDEA and decompiled the super classes since all of the loading logic was abstracted away.
`I guess IDEs do have their place :)` The super classes were likely pulled in during the maven install. Nevertheless, a decompiled 
Java .class file is somewhat magical since it's extremely readable. While missing comments, the method names are either integral
to the compiled bytecode itself or deduced from Classes and methods that call it. The whitespace is probably nicer than the
actual source code too.

![Decompiled byte-code](https://raw.githubusercontent.com/adfyong/openmrs-core/blog/decompile.png)
*Pretty much as good as the real thing*

When updating the test, I decided to improve the logic too. The previous logic was:

1. Pick the most recent patient
2. Change their name to John Bob Smith, gender to Male, and birth date to 2 April 1985
3. Confirm "John Bob Smith" appears in the source code of the page after submission

This led a lot of John Bob Smiths being renamed John Bob Smith so it was difficult to see whether anything worked. I decided
on the following:

1. Pick the most recent patient (I assumed one was there too...)
2. Use the `PatientGenerator` to generate a random patient
3. Fill that patient's information in the form
4. Verify that the generated information is the same as the new information after compilation

During the course of updating the test, I had to familiarize myself with a few web technologies
 - Selenium to grab DOM objects from the webpage and manipulate them
 - Xpath to parse HTML and find DOM objects that don't have IDs

Once I got everything up and running, I committed and pushed to my fix branch (`RA-1100`). It was here that I noticed
a bug with single digit days. Knowing that the project maintainers prefer a small number of commits, I fixed the issue
and used `git rebase -i` to squash them together.

Then I created the PR and linked the PR and the ticket together. It usually takes some time for someone to review the code, but a bot
has replied already notifying the appropriate people.

https://github.com/openmrs/openmrs-distro-referenceapplication/pull/111

Addendum : Anatomy of a Test
----------------------------

![Step 1](https://raw.githubusercontent.com/adfyong/openmrs-core/blog/test1.png)
*Step 1: Go to the home page (logs in via POST request so nothing is actually entered here)*

![Step 2](https://raw.githubusercontent.com/adfyong/openmrs-core/blog/test2.png)
*Step 2: Click on active visits*

![Step 3](https://raw.githubusercontent.com/adfyong/openmrs-core/blog/test2andahalf.png)
*Step 3: Click on the small Edit link at the top*

![Step 4](https://raw.githubusercontent.com/adfyong/openmrs-core/blog/test3.png)
*Step 4: Enter a name (this demo is actually randomly generated... not)*

![Step 5](https://raw.githubusercontent.com/adfyong/openmrs-core/blog/test4.png)
*Step 5: Enter a gender*

![Step 6](https://raw.githubusercontent.com/adfyong/openmrs-core/blog/test5.png)
*Step 6: Enter a birth date (this is accurate... right?)*

![Step 7](https://raw.githubusercontent.com/adfyong/openmrs-core/blog/test6.png)
*Step 7: Parse the page and verify that all the correct elements are there* 
