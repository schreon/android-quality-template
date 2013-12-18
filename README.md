This tutorial will guide you through all steps which are necessary to setup a development environment on ubuntu featuring several software quality tools and a continuous integration server.

Developer Machine
=================
	sudo apt-get install lib32stdc++6 lib32z1
	sudo apt-get install openjdk-6-jdk

Open your .bashrc and add the following line to the end of the file:

	export JAVA_HOME=/usr/lib/jvm/java-6-openjdk-amd64
	export PATH=$PATH:$JAVA_HOME/jre

To change the java version which is used:

	sudo update-alternatives --config java
	sudo update-alternatives --config javac

Download and install [Android Studio](http://developer.android.com/sdk/installing/studio.html).

Download [Gradle version 1.8](http://www.gradle.org/downloads). Extract it to some folder, e.g. ~/Dev.

Android Studio ships with its own copy of the Android SDK. Start the SDK manager and install Android SDK Buid-tools 18.1.1. Also install everything from Android 4.3 (API 18) except for the ARM EABI v7a System Image. 

Setup Gradle
------------
Open your .bashrc and add the following line to the end of the file:

	export PATH=$PATH:/home/<username>/Dev/gradle-1.8/bin

Then reload your .bashrc:

	source ~/.bashrc

Make Android Studio use your local gradle distribution:

	Android Studio -> File -> Settings -> Gradle -> Use local gradle distribution = ~/Dev/gradle1.8/

Create the project
------------------
Fork this repository and clone it, or simply clone this repository directly to try it out:

	git clone https://github.com/schreon/android-quality-template.git

Go to the project folder and run `gradle clean assemble`

Then synchronize Android Studio with gradle by hitting the small icon with the gradle symbol and a small blue arrow. 

Write Tests
-----------
Write tests using robolectric, have a look at the example in the `src/test/java/` folder and [consider the docs](http://robolectric.org/).

Run Tests
---------
Go to the project folder and run `gradle clean test`

Setup a remote Continuous Integration server
============================================

Gain access to a ubuntu server, for example a virutal machine.



