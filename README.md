# What is Spring Data MongoDB ?
According to spring.io website:

Spring Data for MongoDB is part of the umbrella Spring Data project which aims to provide a familiar and consistent Spring-based programming model for new datastores while retaining store-specific features and capabilities.

spring.io website
Link to Source code
You can download the source code from Github: https://github.com/SaiUpadhyayula/spring-boot-mongodb-tutorial

If you are a visual learner like me, you can check out the video version of this tutorial


# Bootstrapping the Project
Now let’s go ahead and bootstrap the project we are going to use in this tutorial. We are going to build a REST API for an Expense Manager application, where you can track your expenses.

You can download the starter project with all the needed dependencies at Spring Initializr website.


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

1
2
####### Mongo Properties ###########
spring.data.mongodb.uri=mongodb://localhost:27017/expense-tracker
Approach 2:

1
2
3
4
5
####### Mongo Properties ###########
spring.data.mongodb.host=localhost
spring.data.mongodb.username=<your-username>
spring.data.mongodb.password=<your-password>
spring.data.mongodb.database=expense-tracker
Creating Domain Model
The model class we are going to create for the Expense Manager application is the Expense.java class.


package com.programming.techie.mongo.model;
 
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.index.Indexed;
import org.springframework.data.mongodb.core.mapping.Document;
import org.springframework.data.mongodb.core.mapping.Field;
 
import java.math.BigDecimal;
 
@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
@Document("expense")
public class Expense {
    @Id
    private String id;
    @Field("name")
    @Indexed(unique = true)
    private String expenseName;
    @Field("category")
    private ExpenseCategory expenseCategory;
    @Field("amount")
    private BigDecimal expenseAmount;
}
ExpenseCategory.java


package com.programming.techie.mongo.model;
 
public enum ExpenseCategory {
    ENTERTAINMENT, GROCERIES, RESTAURANT, UTILITIES, MISC
}
This is just a normal POJO class, which contains some interesting annotations, if you are not already aware of Lombok, it is an annotation library which helps us reduce the boiler plate code.

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


package com.programming.techie.mongo.repository;
 
import com.programming.techie.mongo.model.Expense;
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.data.mongodb.repository.Query;
 
import java.util.Optional;
 
public interface ExpenseRepository extends MongoRepository<Expense, String> {
    @Query("{'name': ?0}")
    Optional<Expense> findByName(String name);
}
We have to just, create an ExpenseRepository.java interface and extend it with MongoRepository interface to be able to perform all the basic Database operations.

If you have a look at the source code of MongoRepository interface, it provides all the basic methods we need to read and persist the documents to MongoDB.


public interface MongoRepository<T, ID> extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> {
    <S extends T> List<S> saveAll(Iterable<S> var1);
 
    List<T> findAll();
 
    List<T> findAll(Sort var1);
 
    <S extends T> S insert(S var1);
 
    <S extends T> List<S> insert(Iterable<S> var1);
 
    <S extends T> List<S> findAll(Example<S> var1);
 
    <S extends T> List<S> findAll(Example<S> var1, Sort var2);
}
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


package com.programming.techie.mongo.controller;
 
import com.programming.techie.mongo.service.ExpenseService;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
 
@RestController
@RequestMapping("/expense")
@RequiredArgsConstructor
public class ExpenseController {
 
    private final ExpenseService expenseService;
     
}

package com.programming.techie.mongo.service;
 
import com.programming.techie.mongo.repository.ExpenseRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
 
@Service
@RequiredArgsConstructor
public class ExpenseService {
 
    private final ExpenseRepository expenseRepository;
 
}
There is nothing interesting going on inside these classes, so let’s go ahead and add the functionality

Add Expense
Let’s add logic to add a new Expense first inside ExpenseService.java


package com.programming.techie.mongo.service;
 
import com.programming.techie.mongo.model.Expense;
import com.programming.techie.mongo.repository.ExpenseRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
 
@Service
@RequiredArgsConstructor
@Transactional
public class ExpenseService {
 
    private final ExpenseRepository expenseRepository;
 
    public void addExpense(Expense expense) {
        expenseRepository.insert(expense);
    }
}
And inside the ExpenseController.java


package com.programming.techie.mongo.controller;
 
import com.programming.techie.mongo.model.Expense;
import com.programming.techie.mongo.service.ExpenseService;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
 
@RestController
@RequestMapping("/expense")
@RequiredArgsConstructor
public class ExpenseController {
 
    private final ExpenseService expenseService;
 
    @PostMapping
    public ResponseEntity addExpense(@RequestBody Expense expense) {
        expenseService.addExpense(expense);
        return ResponseEntity.status(HttpStatus.CREATED).build();
    }
 
}
To add a new entry to the database, we are using the insert() method of the ExpenseRepository interface.

Let’s start our SpringBootMongoDbTutorialApplication and make the REST API call to add an expense.


# Get All Expenses
Let’s add logic to read all expenses stored inside the database


package com.programming.techie.mongo.service;
 
import com.programming.techie.mongo.model.Expense;
import com.programming.techie.mongo.repository.ExpenseRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
 
import java.util.List;
 
@Service
@RequiredArgsConstructor
@Transactional
public class ExpenseService {
 
    private final ExpenseRepository expenseRepository;
 
    public void addExpense(Expense expense) {
        expenseRepository.insert(expense);
    }
 
    public List<Expense> getAllExpenses() {
        return expenseRepository.findAll();
    }
}

package com.programming.techie.mongo.controller;
 
import com.programming.techie.mongo.model.Expense;
import com.programming.techie.mongo.service.ExpenseService;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
 
import java.util.List;
 
@RestController
@RequestMapping("/expense")
@RequiredArgsConstructor
public class ExpenseController {
 
    private final ExpenseService expenseService;
 
    @PostMapping
    public ResponseEntity addExpense(@RequestBody Expense expense) {
        expenseService.addExpense(expense);
        return ResponseEntity.status(HttpStatus.CREATED).build();
    }
 
    @GetMapping
    public ResponseEntity<List<Expense>> getAllExpenses() {
        return ResponseEntity.ok(expenseService.getAllExpenses());
    }
}
To be able to read all the documents inside our collection, we are using the findAll() method from ExpenseRepository.

# Let’s test this logic using Postman


Update Expense
Now the logic to update an expense.


package com.programming.techie.mongo.service;
 
import com.programming.techie.mongo.model.Expense;
import com.programming.techie.mongo.repository.ExpenseRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
 
import java.util.List;
 
@Service
@RequiredArgsConstructor
@Transactional
public class ExpenseService {
 
    private final ExpenseRepository expenseRepository;
 
    public void addExpense(Expense expense) {
        expenseRepository.insert(expense);
    }
 
    public void updateExpense(Expense expense) {
        Expense savedExpense = expenseRepository.findById(expense.getId()).orElseThrow(() -> new RuntimeException(String.format("Cannot Find Expense by ID %s", expense.getId())));
        savedExpense.setExpenseName(expense.getExpenseName());
        savedExpense.setExpenseCategory(expense.getExpenseCategory());
        savedExpense.setExpenseAmount(expense.getExpenseAmount());
 
        expenseRepository.save(expense);
    }
 
    public List<Expense> getAllExpenses() {
        return expenseRepository.findAll();
    }
}

package com.programming.techie.mongo.controller;
 
import com.programming.techie.mongo.model.Expense;
import com.programming.techie.mongo.service.ExpenseService;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
 
import java.util.List;
 
@RestController
@RequestMapping("/expense")
@RequiredArgsConstructor
public class ExpenseController {
 
    private final ExpenseService expenseService;
 
    @PostMapping
    public ResponseEntity addExpense(@RequestBody Expense expense) {
        expenseService.addExpense(expense);
        return ResponseEntity.status(HttpStatus.CREATED).build();
    }
 
    @PutMapping
    public ResponseEntity updateExpense(@RequestBody Expense expense) {
        expenseService.updateExpense(expense);
        return ResponseEntity.status(HttpStatus.OK).build();
    }
 
    @GetMapping
    public ResponseEntity<List<Expense>> getAllExpenses() {
        return ResponseEntity.ok(expenseService.getAllExpenses());
    }
 
}
To update an existing document, we can use the save() method from the MongoRepository interface.

I am going to copy the response from the Get All Expenses call and change the price to 30.


Let’s verify whether the expense is updated or not by calling the Get All Expense API again.


Get Expense By Name
Now let’s implement the logic to get an expense by providing the name.


package com.programming.techie.mongo.service;
 
import com.programming.techie.mongo.model.Expense;
import com.programming.techie.mongo.repository.ExpenseRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
 
import java.util.List;
 
@Service
@RequiredArgsConstructor
@Transactional
public class ExpenseService {
 
    private final ExpenseRepository expenseRepository;
 
    public void addExpense(Expense expense) {
        expenseRepository.insert(expense);
    }
 
    public void updateExpense(Expense expense) {
        Expense savedExpense = expenseRepository.findById(expense.getId()).orElseThrow(() -> new RuntimeException(String.format("Cannot Find Expense by ID %s", expense.getId())));
        savedExpense.setExpenseName(expense.getExpenseName());
        savedExpense.setExpenseCategory(expense.getExpenseCategory());
        savedExpense.setExpenseAmount(expense.getExpenseAmount());
 
        expenseRepository.save(expense);
    }
 
    public Expense getExpense(String name) {
        return expenseRepository.findByName(name)
                .orElseThrow(() -> new RuntimeException(String.format("Cannot Find Expense by Name - %s", name)));
    }
 
    public List<Expense> getAllExpenses() {
        return expenseRepository.findAll();
    }
}

package com.programming.techie.mongo.controller;
 
import com.programming.techie.mongo.model.Expense;
import com.programming.techie.mongo.service.ExpenseService;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
 
import java.util.List;
 
@RestController
@RequestMapping("/expense")
@RequiredArgsConstructor
public class ExpenseController {
 
    private final ExpenseService expenseService;
 
    @PostMapping
    public ResponseEntity addExpense(@RequestBody Expense expense) {
        expenseService.addExpense(expense);
        return ResponseEntity.status(HttpStatus.CREATED).build();
    }
 
    @PutMapping
    public ResponseEntity updateExpense(@RequestBody Expense expense) {
        expenseService.updateExpense(expense);
        return ResponseEntity.status(HttpStatus.OK).build();
    }
 
    @GetMapping
    public ResponseEntity<List<Expense>> getAllExpenses() {
        return ResponseEntity.ok(expenseService.getAllExpenses());
    }
 
    @GetMapping("/{name}")
    public ResponseEntity getExpenseByName(@PathVariable String name) {
        return ResponseEntity.ok(expenseService.getExpense(name));
    }
}
We are going to use the Custom Query defined in the section (Creating Custom Queries using @Query) to find an expense by it’s name, as the name field is indexed and unique, we can be sure that there won’t be any duplicate documents with same name.


# Delete Expense
Now let’s go ahead and implement the last API in the Expense Manager application, to delete an expense.


package com.programming.techie.mongo.service;
 
import com.programming.techie.mongo.model.Expense;
import com.programming.techie.mongo.repository.ExpenseRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
 
import java.util.List;
 
@Service
@RequiredArgsConstructor
@Transactional
public class ExpenseService {
 
    private final ExpenseRepository expenseRepository;
 
    public void addExpense(Expense expense) {
        expenseRepository.insert(expense);
    }
 
    public void updateExpense(Expense expense) {
        Expense savedExpense = expenseRepository.findById(expense.getId()).orElseThrow(() -> new RuntimeException(String.format("Cannot Find Expense by ID %s", expense.getId())));
        savedExpense.setExpenseName(expense.getExpenseName());
        savedExpense.setExpenseCategory(expense.getExpenseCategory());
        savedExpense.setExpenseAmount(expense.getExpenseAmount());
 
        expenseRepository.save(expense);
    }
 
    public Expense getExpense(String name) {
        return expenseRepository.findByName(name)
                .orElseThrow(() -> new RuntimeException(String.format("Cannot Find Expense by Name - %s", name)));
    }
 
    public List<Expense> getAllExpenses() {
        return expenseRepository.findAll();
    }
 
    public void deleteExpense(String id) {
        expenseRepository.deleteById(id);
    }
}

package com.programming.techie.mongo.controller;
 
import com.programming.techie.mongo.model.Expense;
import com.programming.techie.mongo.service.ExpenseService;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
 
import java.util.List;
 
@RestController
@RequestMapping("/expense")
@RequiredArgsConstructor
public class ExpenseController {
 
    private final ExpenseService expenseService;
 
    @PostMapping
    public ResponseEntity addExpense(@RequestBody Expense expense) {
        expenseService.addExpense(expense);
        return ResponseEntity.status(HttpStatus.CREATED).build();
    }
 
    @PutMapping
    public ResponseEntity updateExpense(@RequestBody Expense expense) {
        expenseService.updateExpense(expense);
        return ResponseEntity.status(HttpStatus.OK).build();
    }
 
    @GetMapping
    public ResponseEntity<List<Expense>> getAllExpenses() {
        return ResponseEntity.ok(expenseService.getAllExpenses());
    }
 
    @GetMapping("/{name}")
    public ResponseEntity getExpenseByName(@PathVariable String name) {
        return ResponseEntity.ok(expenseService.getExpense(name));
    }
 
    @DeleteMapping("/{id}")
    public ResponseEntity deleteExpense(@PathVariable String id) {
        expenseService.deleteExpense(id);
        return ResponseEntity.noContent().build();
    }
 
}
We can delete a document from the collection using the delete() method from the MongoRepository interface.

Let’s try to delete the Expense we created before



After deleting the expense, you can see that the Get All Expenses API call returned an empty array, which means the expense was deleted successfully.

Performing Migrations using Mongock
Till now, we discussed how to perform basic CRUD operations using Spring Data MongoDB, now let’s go ahead and see how to apply Migrations to our MongoDB collections.

Database Migrations helps us to easily maintain, track and automate the changes to the database schema, if you are coming from Relational Database world, you may be familiar with the tools Liquibase or Flyway.

MongoDB also has a tool called as Mongock, which is responsible to perform database migrations.

To get started, we have to install Mongock in our project, for that we have to add the below dependencies, to our pom.xml file.

Please make sure to check the maven repository of Mongock, to make sure that you have the latest of version of these libraries.

mongock-bom


<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.github.cloudyrock.mongock</groupId>
            <artifactId>mongock-bom</artifactId>
            <version>4.1.19</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
mongock-spring-v5


<dependency>
    <groupId>com.github.cloudyrock.mongock</groupId>
    <artifactId>mongock-spring-v5</artifactId>
</dependency>
mongodb-springdata-v3-driver

dependency>
    <groupId>com.github.cloudyrock.mongock</groupId>
    <artifactId>mongodb-springdata-v3-driver</artifactId>
</dependency>
If you are facing problems download Mongock, have a look at the QuickStart section of the documentation.

There are 2 steps we have to follow, to enable Mongock in our application.

Preview in new tab

Add the @EnableMongock annotation to our main class (SpringBootMongoDbTutorialApplication.java)
Add the path to the database migration changelogs to the application.properties file.

We can use the mongock.change-logs-scan-package to point to the package where the Java files responsible to scan the change logs are present.
ChangeLogs and ChangeSets
Mongock uses ChangeLogs which are the Java Classes responsible for Migrations and ChangeSets, which are methods which are responsible to apply the migrations to the database schema.

To be able to run migrations,

We have to annotate the class responsible for migration with @ChangeLog
And annotate the method responsible to apply migrations with @Changeset annotation.
At the time of startup, Mongock scans the package provided for the mongock.change-logs-scan-package for the ChangeLog and Changesets, and will start executing it.

Let’s create the java class responsible for the migration – DatabaseChangeLog.java


package com.programming.techie.mongo.config;
 
import com.github.cloudyrock.mongock.ChangeLog;
import com.github.cloudyrock.mongock.ChangeSet;
import com.programming.techie.mongo.model.Expense;
import com.programming.techie.mongo.model.ExpenseCategory;
import com.programming.techie.mongo.repository.ExpenseRepository;
 
import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.List;
 
import static com.programming.techie.mongo.model.ExpenseCategory.*;
 
@ChangeLog
public class DatabaseChangeLog {
 
    @ChangeSet(order = "001", id = "seedDatabase", author = "Sai")
    public void seedDatabase(ExpenseRepository expenseRepository) {
        List<Expense> expenseList = new ArrayList<>();
        expenseList.add(createNewExpense("Movie Tickets", ENTERTAINMENT, BigDecimal.valueOf(40)));
        expenseList.add(createNewExpense("Dinner", RESTAURANT, BigDecimal.valueOf(60)));
        expenseList.add(createNewExpense("Netflix", ENTERTAINMENT, BigDecimal.valueOf(10)));
        expenseList.add(createNewExpense("Gym", MISC, BigDecimal.valueOf(20)));
        expenseList.add(createNewExpense("Internet", UTILITIES, BigDecimal.valueOf(30)));
 
        expenseRepository.insert(expenseList);
    }
 
    private Expense createNewExpense(String expenseName, ExpenseCategory expenseCategory, BigDecimal amount) {
        Expense expense = new Expense();
        expense.setExpenseName(expenseName);
        expense.setExpenseAmount(amount);
        expense.setExpenseCategory(expenseCategory);
        return expense;
    }
}
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

{ 
    "_id" : ObjectId("5ff6303090b13917dfb170b7"), 
    "name" : "Movie Tickets", 
    "category" : "ENTERTAINMENT", 
    "amount" : "40", 
    "_class" : "com.programming.techie.mongo.model.Expense"
}
{ 
    "_id" : ObjectId("5ff6303090b13917dfb170b8"), 
    "name" : "Dinner", 
    "category" : "RESTAURANT", 
    "amount" : "60", 
    "_class" : "com.programming.techie.mongo.model.Expense"
}
{ 
    "_id" : ObjectId("5ff6303090b13917dfb170b9"), 
    "name" : "Netflix", 
    "category" : "ENTERTAINMENT", 
    "amount" : "10", 
    "_class" : "com.programming.techie.mongo.model.Expense"
}
{ 
    "_id" : ObjectId("5ff6303090b13917dfb170ba"), 
    "name" : "Gym", 
    "category" : "MISC", 
    "amount" : "20", 
    "_class" : "com.programming.techie.mongo.model.Expense"
}
{ 
    "_id" : ObjectId("5ff6303090b13917dfb170bb"), 
    "name" : "Internet", 
    "category" : "UTILITIES", 
    "amount" : "30", 
    "_class" : "com.programming.techie.mongo.model.Expense"
}
Mongock also keep track of the ChangeSets, by storing them inside the mongockChangeLog collection, once a changset is applied, it marks it as EXECUTED to prevent re-running this migration once again.

use expense-tracker;
db.getCollection("mongockChangeLog").find({});
{ 
    "_id" : ObjectId("5ff6303090b13917dfb170bc"), 
    "executionId" : "2021-01-06T22:48:32.481939600-722", 
    "changeId" : "seedDatabase", 
    "author" : "Sai", 
    "timestamp" : ISODate("2021-01-06T21:48:32.520+0000"), 
    "state" : "EXECUTED", 
    "changeLogClass" : "com.programming.techie.mongo.config.DatabaseChangeLog", 
    "changeSetMethod" : "seedDatabase", 
    "executionMillis" : NumberLong(36), 
    "_class" : "io.changock.driver.api.entry.ChangeEntry"
}
Testing MongoDB using TestContainers
TestContainers is a library which provides light weight throw away instances of commonly used Databases and other libraries which can run inside a Docker Container.

Let’s go ahead and create the test inside the SpringBootMongodbTutorialApplicationTests.java class


package com.programming.techie.mongo;
 
import com.programming.techie.mongo.model.Expense;
import com.programming.techie.mongo.model.ExpenseCategory;
import com.programming.techie.mongo.repository.ExpenseRepository;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration;
import org.springframework.boot.test.autoconfigure.data.mongo.DataMongoTest;
import org.springframework.test.context.DynamicPropertyRegistry;
import org.springframework.test.context.DynamicPropertySource;
import org.testcontainers.containers.MongoDBContainer;
 
import java.math.BigDecimal;
import java.util.List;
 
import static org.junit.jupiter.api.Assertions.assertEquals;
 
@DataMongoTest(excludeAutoConfiguration = EmbeddedMongoAutoConfiguration.class)
class SpringBootMongodbTutorialApplicationTests {
 
    static MongoDBContainer mongoDBContainer = new MongoDBContainer("mongo:4.4.2");
 
    {
        mongoDBContainer.start();
    }
 
    @DynamicPropertySource
    static void setProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.data.mongodb.uri", mongoDBContainer::getReplicaSetUrl);
    }
 
    @Autowired
    private ExpenseRepository expenseRepository;
 
    @Test
    @DisplayName("Test Whether Expenses are pre-populated")
    void shouldReturnPrePopulatedExpenses() {
        List<Expense> expenses = expenseRepository.findAll();
        assertEquals(5, expenses.size());
    }
 
    @Test
    @DisplayName("Should Find Expense By Name")
    void shouldFindExpenseByName() {
        Expense expectedExpense = new Expense();
        expectedExpense.setExpenseName("Evening Drinks");
        expectedExpense.setExpenseCategory(ExpenseCategory.MISC);
        expectedExpense.setExpenseAmount(BigDecimal.TEN);
        expenseRepository.save(expectedExpense);
 
        Expense actualExpense = expenseRepository.findByName("Evening Drinks").orElseThrow();
        assertEquals(expectedExpense.getExpenseName(), actualExpense.getExpenseName());
        assertEquals(expectedExpense.getExpenseCategory(), actualExpense.getExpenseCategory());
        assertEquals(expectedExpense.getExpenseAmount(), actualExpense.getExpenseAmount());
    }
}
To be able to Test the Repository in isolation, we are going to use @DataMongoTest which will provide access to the Repositories and MongoAutoConfiguration classes.
As we are using TestContainers, we don’t need to use the in-memory database which comes out of the box, so we can exclude that by adding the property excludeAutoConfiguration=EmbeddedMongoAutoConfiguration.class to the @DataMongoTest annotation.
We are starting the MongoDB Container by initializing the MongoDBContainer class and passing the docker image of the MongoDB, we want to use for the test.
We are passing the spring.data.mongo.uri property to the string using the @DynamicPropertySource annotation, which will inject the properties into the Spring Test Context.
Coming to the tests, we Autowired the ExpenseRepository interface to our tests, and in the first test, we are making sure that the database is pre-populated with our testdata.
In the second test, we are making sure that we are able to find an expense by it’s name.
NOTE: Make sure to use the same MongoDB configuration inside the @DynamicProperySource as you did inisde the application.properties file, or you may face issues when running the tests.

Ex: Don’t provide the spring.data.mongo.uri property inside Tests and spring.data.mongo.host, spring.data.mongo.port and spring.data.mongo.database inside application.properties file.

You can observe that using TestContainers, we can also test whether the Mongock migrations are also running successfully or not.


