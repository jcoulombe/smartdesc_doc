@page page_install Installing the Software



There is currently no packaged version that can be distributed as a regular software. You will have to rely on Eclipse for building the source, and then use a workaround for using it interactively in a terminal window. Here is the current (tedious) procedure:

First, make sure everything listed in the [what you need](page_get_started.html) section is properly installed.

If you have cloned the source repository in a local directory and need to import the project into Eclipse, use the _File_ drop down menu, select _Import…_, then choose _Maven / Existing Maven Projects_ and browse to the root of the repository, where the `pom.xml` file can be found. If this is your first time using Maven, it might take a while before everything is set up and ready to go. But do not worry, subsequent updates will be much quicker.

Things might be properly set up already, or you also might see a large number of errors. In the latter case, you are probably seeing errors related to packages/classes named `main.java.ca.smartdesc`…. If it is so, go through the following steps:

1.	right-click on the `src` folder;
2.	in the drop-down menu, select _build path -> remove from build path_;
3.	right-click on the `src/main/java` folder;
4.	in the drop-down menu, select _build path -> use as source folder_;
5.	repeat the last steps with the folder `src/test/java`  (if it exists).

You can now create a link that will allow you to run the software interactively from outside the development environment. 

Open and run the class defined in `ca.smartdesc.App.java`. You should see a message on the console complaining that you didn't provide the required arguments, as well as some info about the proper usage. If things are not working properly, check your project properties, and make sure you are using a version of Java that is at least 1.7.

Next, open the debug perspective: select _Window_, _Open Perspective_, then _Debug_. You should see a tab labeled _Debug_ containing a message starting with `<terminated>App...`. Make sure it is expanded, right click on the line containing `<terminated, exit value: 0>` and select `Properties`. In the new open window, select **everything** found in the `Command Line` area, and copy it to the clipboard.

Using a text editor, create a new file containing:

	#!/bin/bash
	PasteTheContentCopiedInThePreviousStepHere $*

Save this file in a directory that is on your `$PATH`, with a name that will be the one of the command used for invoking the program. For example, you can save it as `~/bin/smartdesc`, so that you will be able to call the program simply as `smartdesc`.

Make this new file executable by going into the same directory, then typing (for the example above)

	chmod 755 smartdesc

If all goes well, things should be working now. Open a new terminal window, then enter the command defined by the name you gave to your script file. You should see the same message as the one printed earlier in the Eclipse environment. If the system complains that the program doesn't exist, enter `$PATH` and carefully verify if the directory containing your script is in the colon separated list. If not, add it (probably in `.bashrc`, `bash_profile` or something similar). You should also check what Java version you are running with `java -version`. 


[Back to Index](index.html) / [Basic Usage Tutorial](page_user_tutorial.html)