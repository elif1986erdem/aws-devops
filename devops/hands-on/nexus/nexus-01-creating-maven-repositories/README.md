# Hands-on 01: Create Nexus Proxy and Hosted Repositories for a Maven Project

Purpose of this hands-on training is to learn how to create and use Nexus repositories to store a Maven project.

## Learning Outcomes

At the end of this hands-on training, students will be able to;

- Create hosted and proxy repostiories.

- Create a group repository consisting of hosted and proxy repositories.

- Consolidate all the components of a project into a repository group.

## Outline

- Part 1 - Start Nexus Repository and Create Credentials

- Part 2 - Create and Build a Maven Project (POM)

- Part 3 - Create Maven Proxy Repository

- Part 4 - Create New Components from Maven Central to the Proxy

- Part 5 - Configure and Build Components in Maven Hosted Repository

- Part 6 - Create a Maven Group Repository Consisting of Previously Created Hosted and Proxy Repositories

- Part 7 - Check the Components in the Group Repository by Accesing the UI

## Part 1 : Start Nexus Repository and Create Credentials
Updates: 
```
sudo yum update -y
```

Install Java: 
```
sudo yum install java-1.8.0-openjdk
```
Download and install Maven

```
sudo wget https://repos.fedorapeople.org/repos/dchen/apache-maven/epel-apache-maven.repo -O /etc/yum.repos.d/epel-apache-maven.repo
```
```
sudo sed -i s/\$releasever/6/g /etc/yum.repos.d/epel-apache-maven.repo
```
```
sudo yum install -y apache-maven
```


Create a directory for Nexus and cd inside it:
```
sudo mkdir /app && cd /app
```


Download and install Nexus.

```
sudo wget -O nexus.tar.gz https://download.sonatype.com/nexus/3/latest-unix.tar.gz
```

Remember to create your installation directory first.  

When you are ready to extract the repository manager, run:  

```
sudo tar xvzf latest-unix.tar.gz 
```

Rename  nexus-3* directory for convinience:
```
sudo mv nexus-3* nexus
```

As a good security practice, nexus should not be run from root user, instead it should be run from a newly created user. So we create the user:

```
sudo adduser nexus
```

Give the ownership of the directories related to Nexus to the newly created user:


```
sudo chown -R nexus:nexus /app/nexus
sudo chown -R nexus:nexus /app/sonatype-work
```
Tell nexus that user "nexus" is going to be running the service:

```
sudo vi  /app/nexus/bin/nexus.rc
```
```
run_as_user="nexus"
```

Run Nexus as Systemctl Service: 

First, we have to add Nexus service to systemctl. Create a new file named nexus.service in the following directory:

```
sudo vi /etc/systemd/system/nexus.service
```

Add this content to the directory:
```
[Unit]
Description=nexus service
After=network.target

[Service]
Type=forking
LimitNOFILE=65536
User=nexus
Group=nexus
ExecStart=/app/nexus/bin/nexus start
ExecStop=/app/nexus/bin/nexus stop
User=nexus
Restart=on-abort

[Install]
WantedBy=multi-user.target
```

Finally, enable nexus and start the service: 
```
sudo chkconfig nexus on
sudo systemctl start nexus
```


Open your browser to load the repository manager: http://<AWS public dns>:8081

In your terminal locate the data directory (../sonatype-work/nexus3/).

Retrieve the temporary password from admin.password, e.g. cat admin.password.

Paste the string from admin.password into the Sign in dialog.

Click the Sign in button to start the Setup wizard. Click Next through the steps to update your password.

Leave the Enable Anonymous Access box unchecked.

Click Finish to complete the wizard. 

## Part 2 : Create and Build a Maven Project (POM)

Use your terminal to create a sample POM.

Create a project folder, e.g. 

```
mkdir nexus-hands-on
```

Change your directory:

```
 cd nexus-hands-on
```

Create a pom.xml in the directory: 

```
touch pom.xml
```

Open the POM in vim or vi or any other text editor in your terminal and add the following snippet. Be aware that the quotation marks in xmlns tag might not apear if you directly copy and paste the pom.xml content. Do not forget to fix that:

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>nexus-proxy</artifactId>
  <version>1.0-SNAPSHOT</version>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.10</version>
    </dependency>
  </dependencies>
</project>
```
Save the file

## Part 3 : Create Maven Proxy Repository

Open Nexus Repository Manager UI

Create a repo called maven-proxy-hands-on.

Choose recipe: maven2 (proxy) 

Name: maven-proxy-hands-on

Remote storage URL:  https://repo1.maven.org/maven2 

Click Create repository to complete the form.

Open your settings.xml (create one in the root directory if you don't have one) and add the repo URL configured in steps 1 and 2, so that it points to maven-proxy-lab.

Save your changes in the settings.xml file.

If your settings.xml is read only, set it to write mod with: 
```
sudo chmod 755 settings.xml
```
or open it with sudo.

Your settings.xml file should look like this (Don't forget to change the admin and password): 

```
<settings>
  <mirrors>
    <mirror>
      <!--This sends everything else to /public -->
      <id>nexus</id>
      <mirrorOf>*</mirrorOf>
      <url>http://<AWS public DNS:8081/repository/maven-proxy-hands-on/</url>
    </mirror>
  </mirrors>
  <profiles>
    <profile>
      <id>nexus</id>
      <!--Enable snapshots for the built in central repo to direct -->
      <!--all requests to nexus via the mirror -->
      <repositories>
        <repository>
          <id>central</id>
          <url>http://central</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </repository>
      </repositories>
     <pluginRepositories>
        <pluginRepository>
          <id>central</id>
          <url>http://central</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </pluginRepository>
      </pluginRepositories>
    </profile>
  </profiles>
<activeProfiles>
    <!--make the profile active all the time -->
    <activeProfile>nexus</activeProfile>
  </activeProfiles>
  <servers>
    <server>
      <id>nexus</id>
      <username>admin</username>
      <password>your-password</password> 
    </server>
  </servers>
</settings>
```

## Part 4 : Create New Components from Maven Central to the Proxy

Locate the POM file in the directory you created (nexus-hands-on).

Run the build with the command mvn package. Your build is ready when you see a BUILD SUCCESS message.

Click Browse in the main toolbar to see the list of repositories.

Click Browse from the left-side menu.

Click maven-proxy-hands-on. You’ll see the components you built in the previous exercise.

Click on the component name to review its details.

## Part 5 : Configure and Build Components in Maven Hosted Repository 

Next up, configure Maven release and snapshot repositories (which come configured with the nexus repository manager) for deployment. This means updating pom.xml and settings.xml.

Your mirror url now should point to http://<AWS public DNS>:8081/repository/maven-public 

So your settings.xml should look like this:

```
<settings>
  <mirrors>
    <mirror>
      <!--This sends everything else to /public -->
      <id>nexus</id>
      <mirrorOf>*</mirrorOf>
      <url>http://<AWS public DNS>:8081/repository/maven-public/</url>
    </mirror>
  </mirrors>
  <profiles>
    <profile>
      <id>nexus</id>
      <!--Enable snapshots for the built in central repo to direct -->
      <!--all requests to nexus via the mirror -->
      <repositories>
        <repository>
          <id>central</id>
          <url>http://central</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </repository>
      </repositories>
     <pluginRepositories>
        <pluginRepository>
          <id>central</id>
          <url>http://central</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </pluginRepository>
      </pluginRepositories>
    </profile>
  </profiles>
<activeProfiles>
    <!--make the profile active all the time -->
    <activeProfile>nexus</activeProfile>
  </activeProfiles>
  <servers>
    <server>
      <id>nexus</id>
      <username>admin</username>
      <password>your-password</password> 
    </server>
  </servers>
</settings>
```


Update your POM with a distributionManagement element. Include the endpoints to your maven-releases and maven-snapshots repos:

```
 <project>
…
<distributionManagement>
<repository>
<id>nexus</id>
<name>maven-releases</name>
<url>http://localhost:8081/repository/maven-releases/</url>
</repository>
<snapshotRepository>
<id>nexus</id>
<name>maven-snapshots</name>
<url>http://localhost:8081/repository/maven-snapshots/</url>
</snapshotRepository>
</distributionManagement>
```

Save the file.

 Run the snapshots build:

``` 
mvn clean deploy.
```

Once you see BUILD SUCCESS, you can see the components in the repository from the UI. Check the components inside the snapshot and public repository. 

Open the POM and remove the -SNAPSHOT tag from the version element.

Run the releases build: 

```
mvn clean deploy.
```

Once you see BUILD SUCCESS, you can see the components in the repository from the UI. Go and check the components in the release and public repository.

## Part 6 : Create a Maven Group Repository Consisting of Previously Created Hosted and Proxy Repositories

Create the repository group – the centralized endpoint to access hosted and proxied components. Remember to order the repositories as hosted, then proxy. Do the following to create the repository:

Select Repositories, then Create repository.

Choose the maven2 (group) recipe.

Name: maven-all.

Drag and drop the Available repositories you created in the earlier exercises to the Members field.

Customize your settings for group access.

Copy the URL from the Group Repository you just created. (In this exercise, the repository URL is http://<AWS public DNS>:8081/repository/maven-all).

Modify the mirror configuration in your settings.xml to point to your group. So the "<url>" tag should have http://<AWS public DNS>:8081/repository/maven-all inside it.

So your settings.xml should look like this: 

```
<settings>
  <mirrors>
    <mirror>
      <!--This sends everything else to /public -->
      <id>nexus</id>
      <mirrorOf>*</mirrorOf>
      <url>http://<AWS public DNS>:8081/repository/maven-all/</url>
    </mirror>
  </mirrors>
  <profiles>
    <profile>
      <id>nexus</id>
      <!--Enable snapshots for the built in central repo to direct -->
      <!--all requests to nexus via the mirror -->
      <repositories>
        <repository>
          <id>central</id>
          <url>http://central</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </repository>
      </repositories>
     <pluginRepositories>
        <pluginRepository>
          <id>central</id>
          <url>http://central</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </pluginRepository>
      </pluginRepositories>
    </profile>
  </profiles>
<activeProfiles>
    <!--make the profile active all the time -->
    <activeProfile>nexus</activeProfile>
  </activeProfiles>
  <servers>
    <server>
      <id>nexus</id>
      <username>admin</username>
      <password>your-password</password> 
    </server>
  </servers>
</settings>
```

Download components directly to your repository group: mvn install.

## Part 7 : Check the Components in the local repository that was installed from the Group Repository that was configured
Go inside the .m2/repository/ 

You will see that you have everything installed that is on your group repository. 
