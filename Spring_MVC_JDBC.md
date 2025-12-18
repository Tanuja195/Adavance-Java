```
Student_MVC_Project
│
├── src
│   └── main
│       ├── java
│       │   └── com
│       │       └── cdac
│       │           |
│       │           │── StudentController.java
│       │           │── StudentDAO.java
│       │           |── Student.java
│       │             
│       ├── resources
│       │   └── application.properties   (DB config)
│       │
│       └── webapp
│           └── WEB-INF
│               ├── views
│               │   ├── register.jsp
│               │   ├── editData.jsp
│               │   ├── viewStudent.jsp
│               │
│               └── web.xml   (agar XML config use ho rahi ho)
│
├── pom.xml
└── README.md

```


### Student Registration

## Student.java
```java

package com.cdac;

public class Student {
	
	private int id;
	private String name;
	private String email;
	private String course;
	
	
	public int getId() {
		return id;
	}
	public void setId(int id) {
		this.id = id;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getEmail() {
		return email;
	}
	public void setEmail(String email) {
		this.email = email;
	}
	public String getCourse() {
		return course;
	}
	public void setCourse(String course) {
		this.course = course;
	}

}


```

## StudentController.java
```java

package com.cdac;
import java.sql.SQLException;
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

import com.cdac.StudentDAO;
import com.cdac.Student;

@Controller
public class StudentController {

    @Autowired
    private StudentDAO studentDAO;

    // Register form ----------------------------
    
    @GetMapping("/register")		// get mapping me vahi lenge jo jsp ki file ka nam  hain
    public String showForm() {
        return "register";
    }


    //Registration --------------------------------------
    
    @PostMapping("/registerStudent")
    public String register(@ModelAttribute Student student) throws SQLException {
        studentDAO.registerStudent(student);
        return "redirect:/viewStudent";
    }

    // view data -------------------------------------
    
    @GetMapping("/viewStudent")
    public String viewStudents(Model model) throws SQLException 
    {
        List<Student> students = studentDAO.viewAllStudents();
        model.addAttribute("students", students);
        return "viewStudent";
    }
    

    // Delete --------------------------------------
    
    @PostMapping("/delete")
    public String deleteStudentData(@RequestParam("id") int id) throws SQLException {
    	
    	studentDAO.deleteStudentData(id);
		return "redirect:/viewStudent";
    	
    }
    
    //Edit form --------------------------------------
    
    @GetMapping("/edit")
    public String editForm(@RequestParam("id") int id, Model model) throws SQLException {
		
    	Student stud = studentDAO.getStudentById(id);
    	model.addAttribute("student", stud);
    	return "editData";
    }
    
    @PostMapping("/update")
    public String updateData(@ModelAttribute Student s) throws SQLException {
    	
    	studentDAO.updateStudentData(s);
    	return "redirect:/viewStudent";
    }
        
//    DELETE / INSERT / UPDATE ke baad hamesha redirect: use karna chahiye.
}


```

## StudentDAO.java
``` java

package com.cdac;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;

import javax.sql.DataSource;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;
import org.springframework.transaction.annotation.Transactional;

import com.cdac.Student;

@Repository
@Transactional
public class StudentDAO {

	@Autowired
    private DataSource dataSource;
	
	// Register student 
	public int registerStudent(Student s) throws SQLException {
		
		int row=0;
		
		Connection con = dataSource.getConnection();
	
		String sql = "Insert Into student(name, email, course) Values (?,?,?)";
		
		PreparedStatement ps = con.prepareStatement(sql);		//imp
		ps.setString(1, s.getName());
		ps.setString(2, s.getEmail());
		ps.setString(3, s.getCourse());
		
		row = ps.executeUpdate();  // ye execute hoga to affected row 1 agyegi
		
		if(ps != null)	
			ps.close();
		
		if(con != null)
			con.close();
		
		return row;
		
		
	}

	public List<Student> viewAllStudents() throws SQLException {
		
		ArrayList<Student> list = new ArrayList<>();
		
		
			Connection con = dataSource.getConnection();
			
			String sql = "Select * From student";
			PreparedStatement ps = con.prepareStatement(sql);
			
		 	ResultSet rs = ps.executeQuery();
			
			while(rs.next()) {
				
				Student s = new Student();
				s.setId(rs.getInt("id"));
				s.setName(rs.getString("name"));
				s.setEmail(rs.getString("email"));
				s.setCourse(rs.getString("course"));
				
				list.add(s);
			}
			
		
			if(rs != null)
				rs.close();
			if(ps != null)
				ps.close();
			if(con !=null)
				con.close();
				
			return list;
	}
	
	public void deleteStudentData(int id) throws SQLException {
		
		Connection con = dataSource.getConnection();
		
		String q = "Delete From student Where id=?";
		PreparedStatement ps = con.prepareStatement(q);
		
		ps.setInt(1, id);
		ps.executeUpdate();			
		
	}


	public Student getStudentById(int id) throws SQLException {
		
		Connection con = dataSource.getConnection();
		
		String q = "Select * from student Where id=?";
		PreparedStatement ps = con.prepareStatement(q);
		
		ps.setInt(1, id);
		
		ResultSet rs = ps.executeQuery();
		
		Student s = new Student();
		
		while(rs.next()) {
			
			s.setId(rs.getInt("id"));
			s.setName(rs.getString("name"));
			s.setEmail(rs.getString("email"));
			s.setCourse(rs.getString("course"));
			
		}
		
		return s;
		
	}

	public void updateStudentData(Student s) throws SQLException {
		
		Connection con = dataSource.getConnection();
		
		String q ="Update student set name=?, email=?, course=? Where id=?";
		
		PreparedStatement ps = con.prepareStatement(q);
		
		ps.setString(1, s.getName());
		ps.setString(2, s.getEmail());
		ps.setString(3, s.getCourse());
		ps.setInt(4, s.getId());
		
		ps.executeUpdate();
	}

}


```

## Register.jsp
``` jsp

<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8" isELIgnored="false"%>

<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Student Registration</title>

<style>
body{
    margin: 0;
    padding: 0;
    font-family: Arial, sans-serif;
    background: #121212;
    color: #f1f1f1;
}

/* Container */
.container{
    width: 400px;
    margin: 80px auto;
    padding: 25px 30px;
    background: #1e1e1e;
    border-radius: 10px;
    box-shadow: 0 0 15px rgba(0,0,0,0.6);
}

/* Heading */
h2{
    text-align: center;
    margin-bottom: 25px;
    color: #4CAF50;
}

/* Labels */
label{
    display: block;
    margin-bottom: 6px;
    font-size: 14px;
}

/* Inputs */
input[type="text"],
input[type="email"]{
    width: 100%;
    padding: 10px;
    margin-bottom: 15px;
    border-radius: 6px;
    border: 1px solid #444;
    background: #2a2a2a;
    color: #fff;
    font-size: 14px;
}

input:focus{
    outline: none;
    border-color: #4CAF50;
}

/* Register button */
.btn-register{
    width: 100%;
    padding: 10px;
    background: #4CAF50;
    border: none;
    border-radius: 6px;
    color: white;
    font-size: 15px;
    cursor: pointer;
}

.btn-register:hover{
    background: #43a047;
}

/* View button */
.view-btn{
    width: 100%;
    margin-top: 12px;
    padding: 10px;
    background: #2196F3;
    border: none;
    border-radius: 6px;
    color: white;
    font-size: 15px;
    cursor: pointer;
}

.view-btn:hover{
    background: #1e88e5;
}

a{
    text-decoration: none;
}
</style>

</head>
<body>

<div class="container">

    <h2>Student Registration</h2>

    <form action="registerStudent" method="post">

        <label>Name</label>
        <input type="text" name="name" required>

        <label>Email</label>
        <input type="email" name="email" required>

        <label>Course</label>
        <input type="text" name="course" required>

        <input type="submit" value="Register" class="btn-register">
    </form>

    <a href="viewStudent">
        <button class="view-btn">View All Students</button>
    </a>

</div>

</body>
</html>

```

## viewStudent.java
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8" isELIgnored="false"%>

<%@ taglib uri="jakarta.tags.core" prefix="c" %>

<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Student Data</title>

<style>
body{
    margin: 0;
    padding: 0;
    font-family: Arial, sans-serif;
    background: #121212;
    color: #f1f1f1;
}

/* Page heading */
h2{
    text-align: center;
    margin-top: 40px;
    color: #4CAF50;
}

/* Table styling */
table{
    width: 80%;
    margin: 30px auto;
    border-collapse: collapse;
    background: #1e1e1e;
    box-shadow: 0 0 12px rgba(0,0,0,0.6);
}

th, td{
    padding: 12px;
    text-align: center;
    border-bottom: 1px solid #333;
}

th{
    background: #2a2a2a;
    color: #4CAF50;
    font-size: 14px;
}

tr:hover{
    background: #2f2f2f;
}

/* Buttons */
.btn-edit{
    padding: 6px 12px;
    background: #4CAF50;
    border: none;
    border-radius: 5px;
    color: white;
    cursor: pointer;
    font-size: 13px;
}

.btn-edit:hover{
    background: #43a047;
}

.btn-delete{
    padding: 6px 12px;
    background: #e53935;
    border: none;
    border-radius: 5px;
    color: white;
    cursor: pointer;
    font-size: 13px;
}

.btn-delete:hover{
    background: #d32f2f;
}

/* Add new button */
.add-btn{
    display: block;
    width: 200px;
    margin: 20px auto;
    padding: 10px;
    background: #2196F3;
    border: none;
    border-radius: 6px;
    color: white;
    font-size: 15px;
    cursor: pointer;
}

.add-btn:hover{
    background: #1e88e5;
}

a{
    text-decoration: none;
}
</style>

</head>
<body>

<h2>Registered Students</h2>

<table>
    <tr>
        <th>ID</th>
        <th>NAME</th>
        <th>EMAIL</th>
        <th>COURSE</th>
        <th>ACTION</th>
    </tr>

    <c:forEach var="s" items="${students}">
        <tr>
            <td>${s.id}</td>
            <td>${s.name}</td>
            <td>${s.email}</td>
            <td>${s.course}</td>
            <td>
                <form action="edit" method="get" style="display:inline;">
                    <input type="hidden" name="id" value="${s.id}">
                    <button type="submit" class="btn-edit">Edit</button>
                </form>

                <form action="delete" method="post" style="display:inline;">
                    <input type="hidden" name="id" value="${s.id}">
                    <button type="submit" class="btn-delete">Delete</button>
                </form>
            </td>
        </tr>
    </c:forEach>
</table>

<a href="register">
    <button class="add-btn">Add New Student</button>
</a>

</body>
</html>


```

## editData.jsp

```jsp

<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8" isELIgnored="false"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Edit Form</title>

<style>
body{
    margin: 0;
    padding: 0;
    font-family: Arial, sans-serif;
    background: #121212;
    color: #f1f1f1;
}

/* Container */
.container{
    width: 400px;
    margin: 80px auto;
    padding: 25px 30px;
    background: #1e1e1e;
    border-radius: 10px;
    box-shadow: 0 0 15px rgba(0,0,0,0.6);
}

/* Heading */
h1{
    text-align: center;
    margin-bottom: 25px;
    color: #4CAF50;
}

/* Labels */
label{
    display: block;
    margin-bottom: 6px;
    font-size: 14px;
}

/* Inputs */
input[type="text"],
input[type="email"]{
    width: 100%;
    padding: 10px;
    margin-bottom: 15px;
    border-radius: 6px;
    border: 1px solid #444;
    background: #2a2a2a;
    color: #fff;
    font-size: 14px;
}

input:focus{
    outline: none;
    border-color: #4CAF50;
}

/* Update button */
.btn-update{
    width: 100%;
    padding: 10px;
    background: #4CAF50;
    border: none;
    border-radius: 6px;
    color: white;
    font-size: 15px;
    cursor: pointer;
    margin-bottom: 10px;
}

.btn-update:hover{
    background: #43a047;
}

/* Delete button */
.delete-btn{
    width: 100%;
    padding: 10px;
    background: #e53935;
    border: none;
    border-radius: 6px;
    color: white;
    font-size: 15px;
    cursor: pointer;
}

.delete-btn:hover{
    background: #d32f2f;
}
</style>
</head>
<body>

<div class="container">

<h1>Edit Data</h1>

<form action="update" method="post">
    <input type="hidden" name="id" value="${student.id}">

    <label>Name</label>
    <input type="text" name="name" value="${student.name}" required>

    <label>Email</label>
    <input type="email" name="email" value="${student.email}" required>

    <label>Course</label>
    <input type="text" name="course" value="${student.course}" required>

    <input type="submit" value="Update" class="btn-update">
</form>

<form action="delete" method="post">
    <input type="hidden" name="id" value="${student.id}">
    <button type="submit" class="delete-btn">Delete</button>
</form>

</div>

</body>
</html>


```

## applicationContext.xml
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>cdac</groupId>
  <artifactId>Student_MVC_JDBC</artifactId>
  <packaging>war</packaging>
  <version>0.0.1-SNAPSHOT</version>
  <name>Student_MVC_JDBC Maven Webapp</name>
  <url>http://maven.apache.org</url>
  
  <dependencies>
   
	    <dependency>
	      <groupId>junit</groupId>
	      <artifactId>junit</artifactId>
	      <version>3.8.1</version>
	      <scope>test</scope>
	    </dependency>
	    
	   	<dependency>
	        <groupId>org.springframework</groupId>
	        <artifactId>spring-webmvc</artifactId>
	        <version>6.1.6</version>
	    </dependency>

		<!-- https://mvnrepository.com/artifact/javax.servlet/javax.servlet-api -->
		<!-- https://mvnrepository.com/artifact/jakarta.servlet/jakarta.servlet-api -->
		<dependency>
		    <groupId>jakarta.servlet</groupId>
		    <artifactId>jakarta.servlet-api</artifactId>
		    <version>6.0.0</version>
		    <scope>provided</scope>
		</dependency>
		
		 <!-- https://mvnrepository.com/artifact/jakarta.servlet.jsp.jstl/jakarta.servlet.jsp.jstl-api -->
		   <!-- JSTL -->
		       <!-- https://mvnrepository.com/artifact/jakarta.servlet.jsp.jstl/jakarta.servlet.jsp.jstl-api -->
		<dependency>
		    <groupId>jakarta.servlet.jsp.jstl</groupId>
		    <artifactId>jakarta.servlet.jsp.jstl-api</artifactId>
		    <version>3.0.0</version>
		</dependency>
		
		<dependency>
		    <groupId>org.glassfish.web</groupId>
		    <artifactId>jakarta.servlet.jsp.jstl</artifactId>
		    <version>3.0.1</version>
		</dependency>
        
        <dependency>
	        <groupId>org.springframework</groupId>
	        <artifactId>spring-context</artifactId>
	        <version>6.1.8</version>
    	</dependency>
    
	    <dependency>
	        <groupId>org.springframework</groupId>
	        <artifactId>spring-jdbc</artifactId>
	        <version>6.1.6</version>
	    </dependency>

	    <dependency>
	        <groupId>org.springframework</groupId>
	        <artifactId>spring-tx</artifactId>
	        <version>6.1.6</version>
	    </dependency>
        
       	<dependency>
	        <groupId>org.springframework</groupId>
	        <artifactId>spring-webmvc</artifactId>
	        <version>6.1.8</version>
      	</dependency> 
        
        <dependency>
	        <groupId>com.mysql</groupId>
	        <artifactId>mysql-connector-j</artifactId>
	        <version>8.4.0</version>
    	</dependency>
        <!-- https://mvnrepository.com/artifact/org.springframework/spring-jdbc -->

  </dependencies>
  
  <build>
    <finalName>Student_MVC_JDBC</finalName>
  </build>
  
  
</project>




```

## dispatcher-servlet.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/contexta
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">

<mvc:annotation-driven />
<context:component-scan base-package="com.cdac"/>

    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

</beans>

```

## web.xml

```xml

<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>

    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>
            org.springframework.web.servlet.DispatcherServlet
        </servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:applicationContext.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

</web-app>

```

## pom.xml
```xml

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>cdac</groupId>
  <artifactId>Student_MVC_JDBC</artifactId>
  <packaging>war</packaging>
  <version>0.0.1-SNAPSHOT</version>
  <name>Student_MVC_JDBC Maven Webapp</name>
  <url>http://maven.apache.org</url>
  
  <dependencies>
   
	    <dependency>
	      <groupId>junit</groupId>
	      <artifactId>junit</artifactId>
	      <version>3.8.1</version>
	      <scope>test</scope>
	    </dependency>
	    
	   	<dependency>
	        <groupId>org.springframework</groupId>
	        <artifactId>spring-webmvc</artifactId>
	        <version>6.1.6</version>
	    </dependency>

		<!-- https://mvnrepository.com/artifact/javax.servlet/javax.servlet-api -->
		<!-- https://mvnrepository.com/artifact/jakarta.servlet/jakarta.servlet-api -->
		<dependency>
		    <groupId>jakarta.servlet</groupId>
		    <artifactId>jakarta.servlet-api</artifactId>
		    <version>6.0.0</version>
		    <scope>provided</scope>
		</dependency>
		
		 <!-- https://mvnrepository.com/artifact/jakarta.servlet.jsp.jstl/jakarta.servlet.jsp.jstl-api -->
		   <!-- JSTL -->
		       <!-- https://mvnrepository.com/artifact/jakarta.servlet.jsp.jstl/jakarta.servlet.jsp.jstl-api -->
		<dependency>
		    <groupId>jakarta.servlet.jsp.jstl</groupId>
		    <artifactId>jakarta.servlet.jsp.jstl-api</artifactId>
		    <version>3.0.0</version>
		</dependency>
		
		<dependency>
		    <groupId>org.glassfish.web</groupId>
		    <artifactId>jakarta.servlet.jsp.jstl</artifactId>
		    <version>3.0.1</version>
		</dependency>
        
        <dependency>
	        <groupId>org.springframework</groupId>
	        <artifactId>spring-context</artifactId>
	        <version>6.1.8</version>
    	</dependency>
    
	    <dependency>
	        <groupId>org.springframework</groupId>
	        <artifactId>spring-jdbc</artifactId>
	        <version>6.1.6</version>
	    </dependency>

	    <dependency>
	        <groupId>org.springframework</groupId>
	        <artifactId>spring-tx</artifactId>
	        <version>6.1.6</version>
	    </dependency>
        
       	<dependency>
	        <groupId>org.springframework</groupId>
	        <artifactId>spring-webmvc</artifactId>
	        <version>6.1.8</version>
      	</dependency> 
        
        <dependency>
	        <groupId>com.mysql</groupId>
	        <artifactId>mysql-connector-j</artifactId>
	        <version>8.4.0</version>
    	</dependency>
        <!-- https://mvnrepository.com/artifact/org.springframework/spring-jdbc -->

  </dependencies>
  
  <build>
    <finalName>Student_MVC_JDBC</finalName>
  </build>
   
</project>

```



