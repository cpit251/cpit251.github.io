---
title: "Working With Frameworks in Java: The Spring Framework"
date: 2023-02-07T23:30:52+03:00
lastmod: 2023-02-07T23:30:52+03:00
weight: 8
draft: false
description: This is an introduction to the Spring framework and writing REST APIs using Spring Boot.
---

![Spring framework logo](/images/notes/spring-framework/spring-logo.svg)

The [Spring framework](https://spring.io/projects/spring-framework) is an open-source application framework for the Java programming language. It provides a set of classes and libraries to build flexible and scalable enterprise grade web applications.

[Spring boot](https://spring.io/projects/spring-boot) is a module or an extension built on top of the Spring framework to speed up the development process of building Spring applications.

For a complete description of Spring boot and how its relate to the Spring framework, refer to the links above or to [this article "What is Java Spring Boot?" by Microsoft](https://azure.microsoft.com/en-us/resources/cloud-computing-dictionary/what-is-java-spring-boot/).



## Building a REST API for a ToDo app

We're going to build a simple ToDo list API using Spring boot, which is an extension of the Spring framework. The API will have three endpoints:

- HTTP GET method that returns a todo item by its Id.
  ```
  GET /todos/{id}
  ```
- HTTP POST method that creates a new todo item.
  ```
  POST /todos/
  ```
- HTTP DELETE method that deletes a todo item by its Id.
  ```
  DELETE /todos/{id}
  ```
  
  
### Step 1: Generate the starter project using Spring Initializr


1. Go to [https://start.spring.io](https://start.spring.io). This service allows you to add all the dependencies you need for an application to start up your Spring Boot project.

2. Choose **Maven** and **Java** and complete the project metadata.

3. Click **Add Dependencies** and select `Spring Web`.

4. Click Generate to download the resulting ZIP file, which is an archive of the Spring web application.

![Spring initializer](/images/notes/spring-framework/spring-initializer.png)


### Step 2: Import the project into your IDE and write the code
- Unzip the downloaded project and open it up in Intellij IDEA.
- Open the `pom.xml` file. If you see an error message that says the `spring-boot-maven-plugin` can not be found, then add the version to this plugin as highlighted below:

{{< highlight xml "linenos=table,hl_lines=4, linenostart=1" >}}
<plugin>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-maven-plugin</artifactId>
	<version>${project.parent.version}</version>
</plugin>
{{< / highlight >}}

- Open the main class under `src/main/java/<your-project-id>` and add the following to the main method:

{{< highlight java "linenos=table,hl_lines=4, linenostart=1" >}}
@SpringBootApplication
public class TodoApiApplication {
	public static void main(String[] args) {
		SpringApplication.run(TodoApiApplication.class, args);
	}
}
{{< / highlight >}}

- Create the Controller class `TodoController.java` next to the main class (under `src/main/java/<your-project-id>`)
- Add the API end points to the controller as shown below.
  - Note that this class will make use of features such as streams and collection, which are available in Java 8 and above, to find, add, and delete a ToDo item.

{{< highlight java "linenos=table, linenostart=1" >}}

import org.springframework.web.bind.annotation.*;
import java.util.ArrayList;
import java.util.List;


@RestController
@RequestMapping("todos")
@CrossOrigin(origins ="*", allowedHeaders = "*")
public class TodoController {

    List<ToDo> todoList = new ArrayList<ToDo>();

    // HTTP GET method that returns a todo item by its Id.
    @GetMapping(value="/{id}", produces = "application/json")
    public @ResponseBody ToDo getToDo(@PathVariable String id) {
        System.out.println("Finding a todo item with id " + id);
        return findToDoById(id);
    }

    // HTTP POST method that creates a new todo item.
    @PostMapping(value="/", consumes = "application/json", produces = "application/json")
    public @ResponseBody boolean createToDo(@RequestBody ToDo todo) {
        System.out.println("Adding a todo item  " + todo);
        return addToDoItem(todo);
    }

    // HTTP DELETE method that deletes a todo item by its Id.
    @DeleteMapping(value="/{id}", produces = "application/json")
    public @ResponseBody boolean deleteToDo(@PathVariable String id) {
        System.out.println("Deleting a todo item with id " + id);
        return deleteToDoById(id);
    }

    private ToDo findToDoById(String id) {
        return todoList.stream().filter(todo -> id.equals(todo.getId())).findFirst().orElse(null);
    }

    private boolean addToDoItem(ToDo todo) {
        return todoList.add(todo);
    }

    private boolean deleteToDoById(String id) {
        return todoList.removeIf(todoItem -> todoItem.getId().equals(id));
    }
}


{{< / highlight >}}

- Create a class for the ToDo item named `ToDo.java`

{{< highlight java >}}

import java.util.UUID;

public class ToDo {
    private String name;
    private String id;
    private boolean isComplete;

    public ToDo(){
        this.isComplete = false;
        this.id = UUID.randomUUID().toString();
    }

    public ToDo(String name) {
        this.name = name;
        this.isComplete = false;
        this.id = UUID.randomUUID().toString();
    }

    public String getId() {
        return this.id;
    }

    public String getName() {
        return this.name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public boolean isComplete() {
        return this.isComplete;
    }

    public void setComplete(boolean isComplete) {
        this.isComplete = isComplete;
    }

    public String toString() {
        return "ToDo: id: " + this.id + "\nName: " + this.name + "\nDone: " + (this.isComplete ? "Yes" : "No");
    }
}


{{< / highlight >}}


### Running the application

You can run the application from either the IDE or the Terminal.

#### Running the app from the Intellij IDEA IDE
  - Go to the maven window
  - Expand the project lifecycle and select package.
  - Click on Run
  - ![Maven package](/images/notes/spring-framework/maven-package.png)
  - Run the jar file from the _Terminal_ window in Intellij using `java -jar target/<your-project-id-version.jar>`

#### Running the app from the Terminal

  - On the Windows environment, open your Terminal application, `cd` to the project directory and run:
  ```shell
  mvnw.cmd spring-boot:run
  ```

  - On macOS or Linux environment, open your Terminal application, `cd` to the project directory and run:
  ```shell
  ./mvnw spring-boot:run
  ```



### Step 4: Using the API

You may use the command line tool `curl` to send requests to the API or use a desktop HTTP client app like Postman.

#### Using the command line tool curl:

  - Add a todo list item
    ```shell
    curl -X POST "http://localhost:8080/todos/" -H 'Content-Type: application/json' -d '{"name":"Buy Milk"}'
    ```
    ```
    true
    ```
  - Find a todo list Item
    ```shell
    curl http://localhost:8080/todos/5a3c41e0-0fcb-439e-91f1-3c9b0862b016
    ```
    ```
    {"name":"Buy Milk","id":"5a3c41e0-0fcb-439e-91f1-3c9b0862b016","complete":false}
    ```
  - Delete a todo item
    ```shell
    curl -X DELETE http://localhost:8080/todos/5a3c41e0-0fcb-439e-91f1-3c9b0862b016
    ```
    ```
    true
    ```

#### Using **Postman**

  - Add a todo list item
    - ![Postman HTTP POST request](/images/notes/spring-framework/postman-http-post.png)
    - ![Spring boot API HTTP POST response](/images/notes/spring-framework/http-post-output.png)

  - Find a todo list Item
    - ![Postman HTTP GET request](/images/notes/spring-framework/postman-http-get.png)
    - ![Spring boot API HTTP GET response](/images/notes/spring-framework/http-get-output.png)

  - Delete a todo item
    - ![Postman HTTP DELETE request](/images/notes/spring-framework/postman-http-delete.png)
    - ![Spring boot API HTTP DELETE response](/images/notes/spring-framework/http-delete-output.png)

