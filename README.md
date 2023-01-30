# What is Spring Data MongoDB ?
According to spring.io website:

Spring Data for MongoDB is part of the umbrella Spring Data project which aims to provide a familiar and consistent Spring-based programming model for new datastores while retaining store-specific features and capabilities.

![D_h4u_5U0AAv6km](https://user-images.githubusercontent.com/93249038/215417642-2162ae5c-6450-4f5f-92d5-2ff4eb8adb79.png)


# Bootstrapping the Project
Now let’s go ahead and bootstrap the project we are going to use in this tutorial. We are going to build a REST API for an Expense Manager application, where you can track your expenses.

You can download the starter project with all the needed dependencies at Spring Initializr website.

# What You Need


A favorite text editor or IDE

Java 17 or later

Gradle 7.5+ or Maven 3.5+

You can also import the code straight into your IDE:

Spring Tool Suite (STS)

IntelliJ IDEA

VSCode

# In the Spring Initializr home page, select the following options:

1)Project : Maven Project

2)Language: Java

3)Spring Boot: Version 2.4.1


# Dependencies:
1)Spring Web: To be able to build RESTful API using Spring MVC

2) Spring Data MongoDB: To interact with MongoDB from the Spring Boot Application

3) Lombok: Java Annotation Library which helps to reduce boiler plate code.

4)Testcontainers: Provides lightweight instances of the Mongo Database which we can run inside a Docker Container.


5)After providing the Project Metadata, you can download the project to your machine, by clicking on the Generate button.

# Exploring Maven Dependencies
Once you unzip and open the project in your favorite IDE, open the pom.xml file to have a look at the Maven dependencies which we are going to use in our project.


<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.4.1</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.programming.techie</groupId>
    <artifactId>spring-boot-mongodb-tutorial</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>spring-boot-mongodb-tutorial</name>
    <description>Demo project for Spring Boot</description>
 
    <properties>
        <java.version>15</java.version>
        <testcontainers.version>1.15.1</testcontainers.version>
    </properties>
 
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-mongodb</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
 
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>junit-jupiter</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>mongodb</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
 
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.testcontainers</groupId>
                <artifactId>testcontainers-bom</artifactId>
                <version>${testcontainers.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
 
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
 
</project>
If you want to learn more about Maven, have a look at the comprehensive Blog Post I wrote Maven Complete Tutorial

# Define MongoDB properties
Now it’s time to configure the MongoDB properties inside the application.properties file

You can define the MongoDB properties by either using the MongoURI or by defining the host, username, password and database details.

Approach 1:


####### Mongo Properties ###########
spring.data.mongodb.uri=mongodb://localhost:27017/expense-tracker
Approach 2:

####### Mongo Properties ###########
spring.data.mongodb.host=localhost
spring.data.mongodb.username=<your-username>
spring.data.mongodb.password=<your-password>
spring.data.mongodb.database=expense-tracker
Creating Domain Model
The model class we are going to create for the Expense Manager application is the Expense.java class.



In the above class,

You can see that by adding the @Getter, @Setter, @AllArgsConstructor and @NoArgsConstructor we can generate the required Getter, Setter Methods and the Constructors at compile time.
To define a Model Class as a MongoDB Document, we are going to use the @Document(“expense”) where expense is the name of the Document.
@Id represents a unique identifier for our Document.
We can represent different fields inside the Document using the @Field annotation.
By Default, Spring Data creates the field inside the document using the fieldName of the model class (Eg: expenseName), but we can override this by providing the required value to the annotation eg: @Field(“name”)
To be able to easily retrieve the documents, we can also create an index using the @Indexed annotaion.
We can also specify the unique=true property to make sure that this field is unique.
Creating Repository
Now it’s time to creating the Repository interface, which is responsible to interact with MongoDB. Spring Data MongoDB provides an interface called MongoRepository which provides an API to perform read and write operations to MongoDB.



Creating Custom Queries using @Query
We can also perform custom queries using the @Query annotation and by passing in the required query we need to run to this annotation.

In the below example:

@Query("{'name': ?0}")
Optional<Expense> findByName(String name);
Spring Data will inject the value of the name field into the query, in the place of the ?0 placeholder.

# Creating REST API
We are going to create a REST API, which exposes the following functionality:

Add Expense
Update Expense
Delete Expense
Get All Expenses
Get Expense by Name
First I am going to create two classes ExpenseController.java which acts as the RestController to accept the incoming request and ExpenseService.java which contains the business logic of the ExpenseManager application.


p
To be able to read all the documents inside our collection, we are using the findAll() method from ExpenseRepository.

# Let’s test this logic using Postman


Update Expense
Now the logic to update an expense.



We are going to use the Custom Query defined in the section (Creating Custom Queries using @Query) to find an expense by it’s name, as the name field is indexed and unique, we can be sure that there won’t be any duplicate documents with same name.


# Delete Expense
Now let’s go ahead and implement the last API in the Expense Manager application, to delete an expense.



If you are facing problems download Mongock, have a look at the QuickStart section of the documentation.

There are 2 steps we have to follow, to enable Mongock in our application.

Preview in new tab

Add the @EnableMongock annotation to our main class (SpringBootMongoDbTutorialApplication.java)
Add the path to the database migration changelogs to the application.properties file.

We can use the mongock.change-logs-scan-package to point to the package where the Java files responsible to scan the change logs are present.
ChangeLogs and ChangeSets
Mongock uses ChangeLogs which are the Java Classes responsible for Migrations and ChangeSets, which are methods which are responsible to apply the migrations to the database schema.

# To be able to run migrations,

We have to annotate the class responsible for migration with @ChangeLog
And annotate the method responsible to apply migrations with @Changeset annotation.
At the time of startup, Mongock scans the package provided for the mongock.change-logs-scan-package for the ChangeLog and Changesets, and will start executing it.

Let’s create the java class responsible for the migration – DatabaseChangeLog.java


package com.programming.techie.mongo.config;
 

Here you can see that the class is annotated with @ChangeLog annotation, which indicates that it is a class Mongock should run for applying migrations.
We created a method called seedDatabase() and annotated it with @ChangeSet annotation.
The @ChangeSet takes in some properties called order which decides the order in which the migration should be applied to the database schema.
id to be able to uniquely identify a ChangeSet
author to be able to set the name of the author who created this Changeset.
Inside the ChangeSet I am just seeding the MongoDB with some testdata, and you can see that I am using the ExpenseRepository interface to save the data, which is injected into the method by Spring.

In our Example we are using the Spring Data Repository to save the data to database but Mongock provides different ways of applying the ChangeSets, you can refer the documentation for more details.

Now let’s start the application and see if Mongock is able to seed the MongoDB with our test data or not.


By having a look at the logs, we can see that Mongock applied the ChangeSet successfully to MongoDB. Let’s verify by querying the database.

use expense-tracker;
db.getCollection("expense").find({});
This should return the following output:


To be able to Test the Repository in isolation, we are going to use @DataMongoTest which will provide access to the Repositories and MongoAutoConfiguration classes.
As we are using TestContainers, we don’t need to use the in-memory database which comes out of the box, so we can exclude that by adding the property excludeAutoConfiguration=EmbeddedMongoAutoConfiguration.class to the @DataMongoTest annotation.
We are starting the MongoDB Container by initializing the MongoDBContainer class and passing the docker image of the MongoDB, we want to use for the test.
We are passing the spring.data.mongo.uri property to the string using the @DynamicPropertySource annotation, which will inject the properties into the Spring Test Context.
Coming to the tests, we Autowired the ExpenseRepository interface to our tests, and in the first test, we are making sure that the database is pre-populated with our testdata.
In the second test, we are making sure that we are able to find an expense by it’s name.
NOTE: Make sure to use the same MongoDB configuration inside the @DynamicProperySource as you did inisde the application.properties file, or you may face issues when running the tests.

Ex: Don’t provide the spring.data.mongo.uri property inside Tests and spring.data.mongo.host, spring.data.mongo.port and spring.data.mongo.database inside application.properties file.

You can observe that using TestContainers, we can also test whether the Mongock migrations are also running successfully or not.
 
 ![Screenshot (100)](https://user-images.githubusercontent.com/93249038/215419579-959f704c-55d1-4444-94c1-caae52b07767.png)


