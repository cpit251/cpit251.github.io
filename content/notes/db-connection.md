---
title: "Working with Databases in Java"
date: 2023-01-23T21:31:51+03:00
weight: 4
description: This lecture note discusses how to connect your Java application to a database and be able to perform CRUD operations (create, read, update, and delete).
draft: false
---

In this lecture note, you will learn how to connect your Java application into a database. While we will use postgres database as an example, the same principles apply to any major relational database. We will create a simple todo list application with tasks that can be marked as done.

> This lecture notes was implemented on PostgreSQL 15.1 and Java 18.0. Other versions may still work given the fact that the used interfaces/APIs have been very stable.

## 1. Start the database server and create the database
The first step is to download the database server for postgres.  Installing postgres itself is enough to work with the database server as it comes with a CLI client tool called `psql`. However, it's often more beginner friendly to interact with the database using a GUI tool. pgAdmin is a web-based GUI tool used to interact with Postgres databases. In short, we will install the following:
1. Postgres database server
2. pgAdmin web interface
3. pgJDBC driver

They should be all installed using the Postgres installer, so you do not need to install them individually. Here's how:

1. **Install postgres**: Download and install postgres from [postgresql.org](https://www.postgresql.org/). Follow the on screen instructions and check all options including pgAdmin4 and stack builder.
    ![Postgres pgAdmin installation](/images/notes/db/postgres-installation-1.png)
  
  - Enter the root password. You will need to remember this password

    ![Postgres pgAdmin installation](/images/notes/db/postgres-installation-2.png)
  
  - Select all default options to complete the installation

2. **Install PostgreSQL JDBC Driver**: Next, the installer will launch stack builder, which should ask you to select additional tools and drivers to install with your postgres installation. We will need to install the PostgreSQL JDBC Driver, which allows Java programs to connect to a PostgreSQL database.
  ![Postgres pgAdmin installation](/images/notes/db/postgres-installation-3.png)

3. **Start pgAdmin**: Enter the root password you set during installation. Then, select the database server from the left pane and enter the same root password.
  ![pgAdmin start server](/images/notes/db/postgres-pgadmin-1.png)
4. **Create a database**: Expand the database server (Postgres 15) and right click to create a database.
  ![pgAdmin create a database](/images/notes/db/postgres-pgadmin-2.png)
  - Enter the database name (e.g., I used `tododb` for this example).
5. **Create a table:** On pgAdmin, expand the database `tododb` -> Schemas -> public -> Right click on Tables and select create Table. Create a table with the following columns:

| Name  | Data type            | Length | Not NULL | Primary key | Default
| ----- | -------------------- | ------ | -------  | -----------  | ----------- |
| id    |   serial             |        | Yes      |  Yes         |             |
| task  |    character varying | 256    | Yes      |              |             |
| done  |    boolean           |        | Yes      |              | false       |

![pgAdmin create a database](/images/notes/db/postgres-pgadmin-3.png)


## 2. Crate a Maven project and add `postgresql` dependency
1. **Create a maven project:** We will create a maven project in the IDE using the quickstart archtype, `maven-archetype-quickstart`, which generates sample Maven project. For example in Intellij IDEA, this looks like:

![pgAdmin create a database](/images/notes/db/intellij-idea-step-1.png)

2. Add the `postgresql` dependency into your `pom.xl` file.
  - Open the `pom.xml` file and add the Postgres dependency:
    
    ```xml
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <version>42.5.1</version>
    </dependency>
    ```
    - Right click on the `pom.xml` file and select **Maven** -> **Reload project**


## 3. Implement the Java database connection and clients
- Create the database connection class next to the main class with the name **DBConnection** and replace your username and password values with the one you chose in the first step.

{{< highlight java "linenos=table,hl_lines=19-20, linenostart=1" >}}
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.util.Properties;

public class DBConnection{
  private final String url;
  private final int port;
  private final String dbName;
  private Connection connection;

  private static DBConnection instance;

  private DBConnection() throws SQLException {
    this.dbName = "tododb";
    this.port = 5432;
    this.url = "jdbc:postgresql://localhost:" + Integer.toString(this.port) + "/" + this.dbName;
    Properties props = new Properties();
    props.setProperty("user","postgres");
    props.setProperty("password","secret");
    props.setProperty("ssl","false");
    this.connection = DriverManager.getConnection(url, props);
  }

  public Connection getConnection() {
    return this.connection;
  }

  public static DBConnection getInstance() throws SQLException {
    if(instance == null){
      instance = new DBConnection();
    }
    else if (instance.getConnection().isClosed()) {
      instance = new DBConnection();
    }
    return instance;
  }
}

{{< / highlight >}}

- Create a client class named **Task** with the following implementation:

{{< highlight java "linenos=table,hl_lines=62-64, linenostart=1" >}}
import java.sql.*;

public class Task {
    private String name;
    private boolean isComplete;

    public Task(String name, boolean isComplete){
        this.name = name;
        this.isComplete = isComplete;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public boolean isComplete() {
        return isComplete;
    }

    public void setComplete(boolean complete) {
        isComplete = complete;
    }

    public void insertTask(){
        try {
            Connection dbConnection = DBConnection.getInstance().getConnection();
            Statement stmt = dbConnection.createStatement();
            PreparedStatement insertStmt =
                    dbConnection.prepareStatement("INSERT INTO todo (task, done) VALUES (?, ?);");
            insertStmt.setString(1, this.name);
            insertStmt.setBoolean(2, (this.isComplete));
            int rows = insertStmt.executeUpdate();
            System.out.println("Rows affected: " + rows);
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    public void retrieveTasks(){
        try {
            Connection dbConnection = DBConnection.getInstance().getConnection();
            Statement stmt = dbConnection.createStatement();
            String query = "SELECT id, task, done FROM todo";
            ResultSet rs = stmt.executeQuery(query);
            while(rs.next()){
                //Display values
                String row = "ID: " + rs.getInt("id") +
                        " Task: " + rs.getString("task") +
                        " Done: " + rs.getBoolean("done") + "\n";
                System.out.print(row);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }

    }

    public void updateTask(){

    }

    public String toString(){
        return "Task: " + this.name + "\nDone: " + (this.isComplete ? "1": "0");
    }
}
{{< / highlight >}}

- Add the following code to the main class that drives the program:

{{< highlight java "linenos=table, linenostart=1" >}}
import java.sql.SQLException;
public class App 
{
    public static void main(String[] args) {
        try {
            DBConnection db = DBConnection.getInstance();
            // Insert
            Task t = new Task("Do the laundry.", false);
            t.insertTask();
            // Retrieve all tasks
            t.retrieveTasks();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}

{{< / highlight >}}


- Run the program:
You should see the following output:

```
Rows affected: 1
ID: 1 Task: Do the laundry. Done: false
```

- Open pgAdmin to see the values in the database:
- Right clickon the `todo` table and select **View/Edit Data** -> **All Rows**
  - ![pgAdmin view table](/images/notes/db/postgres-pgadmin-4.png)
- You should see the values in the database table as listed below:
  - ![pgAdmin view table](/images/notes/db/postgres-pgadmin-5.png)


That's how you connect to a postgres database in Java!


### Extra credit
- Remove the hardcoded database credentials in line # and use environment variables (the highlighted code in the _DBConnection_ class).
- Implement the update method (the highlighted code in the _Task_ class)

