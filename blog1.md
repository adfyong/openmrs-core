I looked at three different open-source projects: Google's TensorFlow, iPython and OpenMRS.

While looking at the issues for the first two, I noticed that the issues contained domain knowledge
that was a bit out of my depth, so I ended up looking at OpenMRS. It seemed to have a healthy
developer community and a nice curated issues dashboard in JIRA specifically for new developers.

Artifact 1: The build process
-----------------------------

Starting from a forked clone of the master repo, I tried to follow the OpenMRS wiki page for
new installs

https://wiki.openmrs.org/display/docs/Getting+Started+as+a+Developer
https://wiki.openmrs.org/display/docs/Step+by+Step+Installation+for+Developers

I opted not to use Eclipse, but IntelliJ IDEA which I already had installed.

I had to install Apache's Maven and add it to my PATH variable for this project.

The first time I built the project, I did so within IntelliJ's included Maven build system.
However, when trying to start the OpenMRS web server, it crashed. Mysteriously, after trying
`mvn clean install` from the command line, and running the server again using `mvn jetty:run`,
it worked!

OpenMRS includes many many unit tests, many of which failed when I built the project.

OpenMRS requires the user to run initial setup from `localhost:8080/openmrs` after the server
is started. I opted to seed the database. This step failed badly with a SQL statement exception.

After some Googling, I learned that OpenMRS requires MySQL 5.6 instead of the more recent version
MySQL 5.7. After reverting my MySQL version, the seeding worked properly.

Once this was seeded, it seemed to work. However, OpenMRS 2.0 does not ship with a default UI.
So I set up the legacy UI from <https://github.com/openmrs/openmrs-module-legacyui>. This
built properly and I just copied the .omod file into the proper modules directory.


Artifact 2: Finding an issue
----------------------------

After browsing the issues on OpenMRS's JIRA, I noticed that the vast majority of the issues
dealt with modules that extended `openmrs-core`. I decided to try looking into one issue
involving the reference application moduke:

https://issues.openmrs.org/browse/RA-995

For this, I had to clone a fork of reference application and several of the associated packages.

When building this, I got a number of `javac` compile errors when building with Maven
: all with `Symbol not found`.

After digging through the Java source code, I noticed that some of the method names and variables
were recently renamed in preparation of a new version of OpenMRS Reference Application.

After fixing the issues and building successfully (ignoring a number of failed test cases that were
likely not caused by me), I created a pull request with one commit.

However, I did not go through the proper channel of making a JIRA issue, waiting for triage, and
claiming it for myself. No one has commented on the PR yet.

https://github.com/openmrs/openmrs-module-referenceapplication/pull/25
