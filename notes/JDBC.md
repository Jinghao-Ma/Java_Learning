## JDBC

------

[TOC]



------

### Java Database Connectivity

- Java程序访问数据库的标准接口



```flow
st=>start: Java App
op1=>operation: JDBC Interface(JDK)
op2=>operation: JDBC Driver (Database Vendor)
e=>end: Database

st->op1->op2->e
```



**JDBC Driver**

```flow
st=>start: JVM--[class, java.sql.*, mysql-xxx.jar]
cond=>operation: Network
e=>end: MySQL Server

st(right)->cond(right)->e
```



**获取数据库连接**

```java
// JDBC connect to the URL
String JDBC_URL = "jdbc:mysql://localhost:3306/test";

// database account
String JDBC_USER = "root";
String JDBC_PASSWORD = 'password';

// connect database
Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD);
// operation in database
conn.close();
```



- 各数据库厂商使用相同的接口，Java代码无需进行针对性的开发
- Java仅依赖`java.sql.*`，不依赖具体的数据库jar包
- 可随时替换底层数据库，访问数据库的代码无需改动

------



### JDBC Query 



**Query**

```java
try (Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD)) {
    try (Statement stat = conn.createStatement()) {
        try (ResultSet rs = stat.executeQuery("SQL statement")) {
            while (rs.next()) {
                long id = rs.getLong(1); // SQL 索引是从1开始
                long classid = rs.getLong(2);
                String name = rs.getString(3);
                String gender = rs.getString(4);
            }
        }
    }
}
```

**PreparedStatement**

```java
 // use ? to replace the value , and set it by setObject(order, value)
PreparedStatement pstat = conn.preparedStatement("SELECT * FROM table_name WHERE username=? AND passwd=?");
pstat.setObject(1, name);
pstat.setObject(2, password);
```

**Query with PreparedStatement**

```java
try (Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD)) {
    try (PreparedStatement pstat = conn.preparedStatement("SELECT * FROM table_name WHERE username=? AND passwd=?")) {
        pstat.setObject(1, name);
        pstat.setObject(2, password);
        try (ResultSet rs = pstat.executeQuery()) {
            while (rs.next()) {
                long id = rs.getLong(1); // SQL 索引是从1开始
                long classid = rs.getLong(2);
                String name = rs.getString(3);
                String gender = rs.getString(4);
            }
        }
    }
}
```

------



#### Demo -- SELECT

two tables here

```sql
mysql> select * from students;
+----+------+----------+--------+
| id | name | class_id | gender |
+----+------+----------+--------+
|  1 | kb   |        1 | M      |
|  2 | cp   |        1 | M      |
|  3 | ca   |        1 | F      |
|  4 | dw   |        2 | M      |
|  5 | sn   |        2 | F      |
|  6 | kl   |        2 | M      |
|  7 | jg   |        3 | M      |
|  8 | sc   |        3 | F      |
|  9 | zm   |        3 | M      |
| 10 | lj   |        4 | M      |
+----+------+----------+--------+

mysql> select * from classes;
+----+------+
| id | name |
+----+------+
|  1 | Frog |
|  2 | Bird |
|  3 | Cat  |
|  4 | Dog  |
+----+------+

```

operation by code

Students class

```java
package org.jdbc.test;

import java.math.BigInteger;

public class Students {
    private long id;
    private long class_id;
    private String name;
    private String gender;

    public Students(long id, long class_id, String name, String gender) {
        this.id = id;
        this.class_id = class_id;
        this.name = name;
        this.gender = gender;
    }

    public long getId() {
        return id;
    }

    public void setId(long id) {
        this.id = id;
    }

    public long getClass_id() {
        return class_id;
    }

    public void setClass_id(long class_id) {
        this.class_id = class_id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getGender() {
        return gender;
    }

    public void setGender(String gender) {
        this.gender = gender;
    }

    @Override
    public String toString() {
        return "id= " + this.id +
                ", class id= " + this.class_id +
                ", name= " + this.name +
                ", gender= " + this.gender;
    }
}
```

Main operation

```java
package org.jdbc.test;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;

public class JdbcAction {
	/**
	 * Complete database account info with URL, USERNAME and PASSWORD
	 */
    static final String JDBC_URL="jdbc:mysql://localhost:3306/test?useSSL=false&characterEncoding=utf8";
//    static final String JDBC_URL="jdbc:mysql://localhost:3306/test";
    static final String JDBC_USER="root";
    static final String JDBC_PASSWD = "password";

    /**
     * getAllStudents -- get all students info from database
     * @ connection -- connect to database
     * @ statement -- SQL statement to database
     * @ set -- result from database
     * @ list -- transfer result to Students Object and store in it
     * 
     */
    static List<Students> getAllStudents() throws SQLException {
        try (Connection connection = getConnection()) {
            try (PreparedStatement statement = connection.prepareStatement("SELECT * FROM students;")) {
                try (ResultSet set = statement.executeQuery()) {
                    List<Students> list = new ArrayList<>();
                    while (set.next()) {
                        long id = set.getLong(1);
                        long class_id = set.getLong(3);
                        String name = set.getString(2);
                        String gender = set.getString(4);
                        Students student = new Students(id, class_id, name, gender);
                        list.add(student);
                    }
                    return list;
                }
            }
        }
    }

    /**
     * getStudentsOfClass -- get students info from from same classId
     * @ connection -- connect to database
     * @ statement -- SQL statement to database
     * @ set -- result from database
     * @ list -- transfer result to Students Object and store in it
     * 
     */
    static List<Students> getStudentsOfClass(long classId) throws SQLException {
        try (Connection connection = getConnection()) {
            try (PreparedStatement statement = connection.prepareStatement("SELECT * FROM students WHERE class_id=?")) {
                statement.setObject(1, classId);
                try (ResultSet set = statement.executeQuery()) {
                    List<Students> list = new ArrayList<>();
                    while (set.next()) {
                        long id = set.getLong(1);
                        long class_id = set.getLong(3);
                        String name = set.getString(2);
                        String gender = set.getString(4);
                        Students student = new Students(id, class_id, name, gender);
                        list.add(student);
                    }
                    return list;
                }
            }
        }
    }

    /**
     * getConnection -- connect to database by with URL, USERNAME and PASSWORD
     */
    static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWD);
    }

    /**
     * getNumOfStudents -- get students count by same classId
     * @ connection -- connect to database
     * @ statement -- SQL statement to database
     * @ set -- result from database
     * @ num -- return the number of students by classId query
     * 
     */    
    static long getNumOfStudents(long classId) throws SQLException{
        try (Connection connection = getConnection()) {
            try (PreparedStatement statement = connection.prepareStatement("SELECT COUNT(*) num FROM students WHERE class_id=?")) {
                statement.setObject(1, classId);
                try (ResultSet set = statement.executeQuery()) {
                    while (set.next()) {
                        long num = set.getLong("num");
                        return num;
                    }
                    throw new RuntimeException("Empty result set");
                }
            }
        }
    }

    /**
     * getAStudentById -- get a specific student info by student id from database
     * @ connection -- connect to database
     * @ statement -- SQL statement to database
     * @ set -- result from database
     * @ student -- return the student of stu_id
     * 
     */        
    static Students getAStudentById(long stu_id) throws SQLException{
        try (Connection connection = getConnection()) {
            try (PreparedStatement statement = connection.prepareStatement("SELECT * FROM students WHERE id=?")) {
                statement.setObject(1, stu_id);
                try (ResultSet set = statement.executeQuery()) {
                    Students student = null;
                    while (set.next()) {
                        long id = set.getLong(1);
                        long class_id = set.getLong(3);
                        String name = set.getString(2);
                        String gender = set.getString(4);
                        student = new Students(id, class_id, name, gender);
                    }
                    return student;
                }
            }
        }
    }

    public static void main(String[] args) throws SQLException {

        List<Students> students = getAllStudents();
        for (Students s : students) {
            System.out.println(s);
        }

        System.out.println("---------------------------");
        for (int i = 4; i >= 1 ; i--) {
            List<Students> list = getStudentsOfClass(i);
            for (Students s : list) {
                System.out.println(s);
            }
        }

        System.out.println("----------------------------");
        for (int i = 1; i <= 4; i++) {
            System.out.println("Class " + i + " has " + getNumOfStudents(i) + " persons.");
        }

        System.out.println("--------- get some one -----------");
        Students teststu = getAStudentById(1);
        System.out.println(teststu);
    }
}
```

------



### JDBC  UPDATE

```java
// update for table
try (Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD)) {
    try (PreparedStatement statement = conn.preparedStatement("SQL-UPDATE statement -- UPDATE students SET name=? WHERE id=?")) {
        statement.setObject(1, "name_value");
        statement.setObject(2, "id_value");
        int n = statement.executeUpdate();
    } 
}

// insert into table
try (Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD)) {
    try (PreparedStatement statement = conn.preparedStatement("SQL-INSERT statement -- INSERT INTO students (ITEMS..) VALUES (?,?,?,?)")) {
        statement.setObject(1, id_value);
        statement.setObject(2, name_value);
        statement.setObject(3, class_id_value);
        statement.setObject(4, gender_value);
        int n = statement.executeUpdate();
    } 
}

// insert into table and get primary key
try (Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD)) {
    try (PreparedStatement statement = conn.preparedStatement("SQL-INSERT statement -- INSERT INTO students (ITEMS..) VALUES (?,?,?)", Statement.RETURN_GENERATED_KEYS)) {
        statement.setObject(1, name_value);
        statement.setObject(2, class_id_value);
        statement.setObject(3, gender_value);
        int n = statement.executeUpdate();
        try (ResuletSet set = statement.getGeneratedKeys()) {
            if (set.next()) {
                long id = set.getLong(1); // get primary key
            }
        }
    } 
}

// delete
try (Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD)) {
    try (PreparedStatement statement = conn.preparedStatement("SQL-DELETE statement -- DELETE FROM students WHERE id=?")) {
        statement.setObject(1, id_value);
        int n = statement.executeUpdate();
    }
}
```



#### Demo -- Update

Main operation

```java
package org.jdbc.test;


import com.sun.xml.internal.ws.addressing.WsaTubeHelper;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;

public class JdbcUpdate {

    static final String JDBC_URL="jdbc:mysql://localhost:3306/test?useSSL=false&characterEncoding=utf8";
    static final String JDBC_USER="root";
    static final String JDBC_PASSWD = "password";

    static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWD);
    }
	
    /**
     * update -- Update data in table by update statement
     * PreparedStatement.executeUpdate() -- run the SQL statement
     * @ n -- retuen an int value
     * 
     * it can add more detail for optimize the method in business 
     */
    static void update() throws SQLException{
        try (Connection connection = getConnection()) {
            try (PreparedStatement statement = connection.prepareStatement("UPDATE students SET id=? WHERE name=?")) {
                statement.setObject(1, 24);
                statement.setObject(2, "kb");
                int n = statement.executeUpdate();
                System.out.println("update: " + n);
            }
        }
    }

    /**
     * insertInto -- Insert a new Students Object data in table by insert statement
     * PreparedStatement.executeUpdate() -- run the SQL statement
     * @ n -- retuen an int value
     * 
     * it can add more detail for optimize the method in business 
     */
    static void insertInto(Students s) throws SQLException{
        try (Connection connection = getConnection()) {
            try (PreparedStatement statement = connection.prepareStatement("INSERT INTO students VALUES (?, ?, ?, ?)")) {
                statement.setObject(1, s.getId());
                statement.setObject(2, s.getName());
                statement.setObject(3, s.getClass_id());
                statement.setObject(4, s.getGender());
                int n = statement.executeUpdate();
                System.out.println("INSERT :" + n);
            }
        }
    }

    /**
     * insertIntoWithAutoGenerateId -- Insert a new Students Object data without id in table by insert statement 
     * Statement.RETURN_GENERATED_KEYS -- generate a primary key to it
     * PreparedStatement.getGeneratedKeys() -- get the primary key id from the talbe
     * PreparedStatement.executeUpdate() -- run the SQL statement
     * @ n -- retuen an int value
     * 
     * it can add more detail for optimize the method
     */
    static void insertIntoWithAutoGenerateId(Students s) throws SQLException{
        try (Connection connection = getConnection()) {
            try (PreparedStatement statement = connection.prepareStatement("INSERT INTO students (name, class_id, gender) VALUES (?, ?, ?)", Statement.RETURN_GENERATED_KEYS)) {
                statement.setObject(1, s.getName());
                statement.setObject(2, s.getClass_id());
                statement.setObject(3, s.getGender());
                int n = statement.executeUpdate();
                System.out.println("INSERT: " + n);
                try (ResultSet set = statement.getGeneratedKeys()) {
                    if (set.next()) {
                        long id = set.getLong(1);
                        s.setId(id);
                    }
                }
            }
        }
    }

    /**
     * delete -- delete a data in table by delete statement
     * PreparedStatement.executeUpdate() -- run the SQL statement
     * @ n -- retuen an int value
     * 
     * it can add more detail for optimize the method in business 
     */
    static void delete(long stu_id) throws SQLException{
        try (Connection connection = getConnection()){
            try (PreparedStatement statement = connection.prepareStatement("DELETE FROM students WHERE id=?")) {
                statement.setObject(1, stu_id);
                int n  = statement.executeUpdate();
                System.out.println("DELETE: " + n);
            }
        }
    }

    static List<Students> getAllStudents() throws SQLException{
        try (Connection connection = getConnection()) {
            try (PreparedStatement statement = connection.prepareStatement("SELECT * FROM students")) {
                try (ResultSet resultSet = statement.executeQuery()) {
                    List<Students> studentsList = new ArrayList<>();
                    while (resultSet.next()) {
                        long id = resultSet.getLong(1);
                        String name = resultSet.getString(2);
                        long class_id = resultSet.getLong(3);
                        String gender = resultSet.getString(4);
                        Students student = new Students(id, class_id, name, gender);
                        studentsList.add(student);
                    }
                    return studentsList;
                }
            }
        }
    }


    public static void main(String[] args) throws SQLException {

//        Students s1 = new Students(99, 4, "killua", "F");
//        insertInto(s1);
//        Students s2 = new Students(67, 3, "Emile", "F");
//        insertIntoWithAutoGenerateId(s2);
//        System.out.println(s2);

//        delete(101);

        List<Students> students = getAllStudents();
        for (Students s : students) {
            System.out.println(s);
        }
    }
}
```

------



### DataSource



**JDBC连接池接口**

- javax.sql.DataSource



**HikariCP**

```xml
<dependency>
	<groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <version>2.7.1</version>
</dependency>
```

```java
HikariConfig config = new HikariConfig();
config.setJdbcUrl("url to database");
config.setUsername("username");
config.setPassword("password");
config.addDataSourceProperty("idleTimeout", "seconds_value"); 
config.addDataSourceProperty("connectionTimeout", "seconds_value");
config.addDataSourceProperty("maximumPoolSize", "size_value"); // 最大连接数

DataSource dataSource = new HikariDataSource(config);
```



#### Demo -- Pool

```java
package org.jdbc.test;

import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;

public class JdbcDataSource {
	/**
	 * database account;
	 */
    static final String JDBC_URL="jdbc:mysql://localhost:3306/test?useSSL=false&characterEncoding=utf8";
    static final String JDBC_USER="root";
    static final String JDBC_PASSWD = "password";

    /**
     * CREATE a pool by Hikari with database account info and some properties
     * RETURN HikariDataSource object
     */
    static DataSource createDataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl(JDBC_URL);
        config.setUsername(JDBC_USER);
        config.setPassword(JDBC_PASSWD);
        config.addDataSourceProperty("idleTimeout", "6000");
        config.addDataSourceProperty("connectionTimeout", "1000");
        config.addDataSourceProperty("maximumPoolSize", "10");
        return new HikariDataSource(config);
    }

    /**
     * getStudentsOfClass -- get students in same class
     * 
     * @ students -- a list of students in same class
     * Connect to a dataSource(pool)
     * RETURN students list
     */
    static List<Students> getStudentsOfClass(DataSource source, long classId) {
        try (Connection connection = source.getConnection()) {
            System.err.println("Running : " + connection);
            try (PreparedStatement statement = connection.prepareStatement("SELECT * FROM students WHERE class_id=?")){
                statement.setObject(1, classId);
                try (ResultSet resultSet = statement.executeQuery()) {
                    List<Students> students = new ArrayList<>();
                    while (resultSet.next()) {
                        long id = resultSet.getLong(1);
                        String name = resultSet.getString(2);
                        long class_id = resultSet.getLong(3);
                        String gender = resultSet.getString(4);
                        Students s = new Students(id, class_id, name, gender);
                        students.add(s);
                    }
                    return students;
                }
            }
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }


    /**
     * @ source -- create a DataSource(Pool)
     * @ threads -- create a arraylist of thread
     * @ Thread.run() -- sleep 1s;
     * 
     * every thread gets info of students of one single class 
     * after first for(), threads got 4 thread objects, that mean there are 4 classes in sample
     * 
     * deeply explain : 4 threads ran in source 4 times by one connection until source close,
     * which means many tasks can run in one connection of pool, drop off how many tasks and how many connection  
     * 
     * run threads, and PRINT OUT info of students of same class separately
     */
    public static void main(String[] args) throws InterruptedException {
        DataSource source = createDataSource();
        List<Thread> threads = new ArrayList<>();

        for (int i = 1; i <= 4; i++) {
            final  int  classId = i;
            Thread thread = new Thread() {
                @Override
                public void run() {
                    try {
                        Thread.sleep((long) (Math.random() * 1000));
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    List<Students> students = getStudentsOfClass(source, classId);
                    System.out.println("Class " + classId + " has ");
                    for (Students s : students) {
                        System.out.println(s);
                    }
                }
            };
            threads.add(thread);
        }
        for (Thread t : threads) {
            t.start();
        }
        for (Thread t : threads) {
            t.join();
        }
        ((HikariDataSource) source).close();
    }
}
```

