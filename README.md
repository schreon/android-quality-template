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

Android Studio ships with its own copy of the Android SDK. Make sure the latest SDK is installed.

Make sure the ANDROID_HOME environment variable is set correctly. Open your .bashrc and add the following line to the end of the file:

	export ANDROID_HOME=/path/to/android-studio/sdk/

Then reload your .bashrc:

	source ~/.bashrc

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

Continuous Integration server
=============================
(TODO: describe in more detail ...)

Gain access to a server or virtual machine. Login using ssh and install [Jenkins](http://jenkins-ci.org/) and [SonarQube](http://www.sonarqube.org/).

Install mysql:

	sudo apt-get install mysql-server
	sudo apt-get install mysql-client

Connect to your mysql database using the mysql-client and paste past the following SQL, but change YOUR_SONAR_USER and YOUR_SONAR_PASSWORD first!

	CREATE DATABASE sonar CHARACTER SET utf8 COLLATE utf8generalci; 
	CREATE USER 'sonar' IDENTIFIED BY 'sonar';
	GRANT ALL ON sonar.* TO 'sonar'@'%' IDENTIFIED BY 'YOUR_SONAR_USER';
	GRANT ALL ON sonar.* TO 'sonar'@'localhost' IDENTIFIED BY 'YOUR_SONAR_PASSWORD';
	FLUSH PRIVILEGES;

Go to the Jenkins dashboard, create a new build configuration for your project and paste the following lines into the textbox where you can add buidl steps. You need to adjust the URLs and the password to your setup!

	export SONAR_URL=http://your.virtual.machine:9000/
	export SONAR_JDBC_URL=jdbc:mysql://localhost/sonar
	export SONAR_JDBC_DRIVER=com.mysql.jdbc.Driver
	export SONAR_JDBC_USER=YOUR_SONAR_USER
	export SONAR_JDBC_PASSWORD=YOUR_SONAR_PASSWORD
	./gradlew clean test sonarRunner --stacktrace

If you want jenkins to automatically build your project as soon as the source code in the github repository changes, you need to enable a webhook.

Go to your [github account settings](https://github.com/settings/applications) and create a new application token (call it "Jenkins build token" or something like that). Copy the token to your clipboard (it will look like 61ab7037c3635ae2f16cca835a798e78)

Go to the jenkins dashboard and the build configuration, where you just added the build steps. Enable the trigger which allows you to remotely start a jenkins build via http. Paste the github application token there. 
Back to github, go to the project settings ("Settings" on the right panel). Click "Service Hooks" -> "WebHook URLs". Enter the following line, but adjust it to the URL to your server, the port jenkins is listening to, the name of your build configartion and your github application token.

	http://your.virtual.machine:8080/job/your-build-configuration/build?token=61ab7037c3635ae2f16cca835a798e78

Update the settings and click "Test hook". Jenkins should now be triggered. After the tests are finished, you can view the results in your SonarQube dashboard (e.g. http://your.virtual.machine:9000/).

If not, make sure the server can be reached from the internet!!! In many cases, only Port 80 is available to the public and all other ports are blocked by a firewall. In this case you must map two different routes to jenkins and sonar respectively. This can easily be done using NGINX:

ssh to your server. Then install nginx

	sudo apt-get install nginx

Open the nginx.conf:
	
	sudo nano /etc/nginx/nginx.conf

In the http section, comment everthing out using the # character.

Then add the following section within http:

    server {
      listen 80;
      location /jenkins/ {
        proxy_pass http://localhost:8080/jenkins/;
      }
      location /sonar/ {
        proxy_pass http://localhost:9000/sonar/;
      }
    }

This assumes your are running jenkins on port 8080 and sonar on port 9000. Save the file by hitting CTRL-O and close it hitting CTRL-X.

Now restart nginx:

	sudo service nginx restart

You must edit all previous steps where you typed the URL to sonar (your.virtual.machine:9000) or jenkins (your.virtual.machine:8080) and change them to your.virtual.machine/sonar and your.virtual.machine/jenkins.

Restart sonar:

	sudo service sonar restart

Restart jenkins:

	sudo service jenkins restart


You should now be able to watch your code quality metrics on http://your.virtual.machine/sonar using your browser.