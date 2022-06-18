# SMBMS-project
Functions: 
-1.login/logout
-2.user management
  --CRUD
-3.order management
  --CRUD
-4.provider management
  --CRUD

BACK database

step1: CREATE DATABASE smbms
step2: create tables


BUILD up projects:
  -1.build up maven frame
  -2.config tomcat
  -3.import needed jar package (servlet, jsp, mysql, jstl, standard)
  -4.create project package structure
  pojo: classes
  dao: operate db
  service: business
  servlet:
  filter: 
  util:tool class
 
  -5.code the class
   ORM mapping: class and tables mapping
  -6. code public basic classes
    -1.carete database config file db.properties in the resources file
    ----------------------------------------------
    driver=com.mysql.jdbc.Driver
    url=jdbc:mysql://localhost:3306?useUnicode=true&characterEncoding=utf-8
    username=root
    password=hsp
    -----------------------------------------
    -2.code database public class
------------------------------------------------------------------------
package com.qilun.dao;


import java.io.IOException;
import java.io.InputStream;
import java.sql.*;
import java.util.Properties;

//public class for operating database
public class BaseDao {
    //start these parameters

    private static String driver;
    private static String url;
    private static String username;
    private static String password;

    //static code, when the class is loading, then start initial
    static {
        //new a property, hold the value
        Properties properties = new Properties();
        //use  classloader to get resources
        InputStream is = BaseDao.class.getClassLoader().getResourceAsStream("db.properties");
        try {
            properties.load(is);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }

        driver = properties.getProperty("driver");
        url = properties.getProperty("url");
        username = properties.getProperty("username");
        password = properties.getProperty("password");
    }

    //get access to the db
    public static Connection getConnection() {
        Connection connection = null;
        try {
            Class.forName(driver);
            connection = DriverManager.getConnection(url, username, password);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
        return connection;
    }


    //code search public method
    public static ResultSet execute(Connection connection, String sql, Object[] params, ResultSet resultSet, PreparedStatement preparedStatement) throws SQLException {
        preparedStatement = connection.prepareStatement(sql);
        for (int i = 0; i < params.length; i++) {
            //setObject start from 1 while array start from 0.
            preparedStatement.setObject(i + 1, params[i]);
        }
        resultSet = preparedStatement.executeQuery();
        return resultSet;
    }

    //code add public method

    //code delete public method

    //code update public method
    public static int execute(Connection connection, String sql, Object[] params, PreparedStatement preparedStatement) throws SQLException {
        //prepareStatement sql direct execute later
        preparedStatement = connection.prepareStatement(sql);
        for (int i = 0; i < params.length; i++) {
            //setObject start from 1 while array start from 0.
            preparedStatement.setObject(i + 1, params[i]);
        }
        int updateRows = preparedStatement.executeUpdate();
        return updateRows;
    }

    //release resources
    public static boolean closeResource(Connection connection, PreparedStatement preparedStatement, ResultSet resultSet) {
        boolean flag = true;
        if (resultSet != null) {
            try {
                resultSet.close();
                //garbage recycle
                resultSet = null;
            } catch (SQLException e) {
               e.printStackTrace();
               flag = false;
            }
        }

        if (preparedStatement != null) {
            try {
                preparedStatement.close();
                //garbage recycle
                preparedStatement = null;
            } catch (SQLException e) {
                e.printStackTrace();
                flag = false;
            }
        }

        if (connection != null) {
            try {
                connection.close();
                //garbage recycle
                connection = null;
            } catch (SQLException e) {
                e.printStackTrace();
                flag = false;
            }
        }

        return flag;

    }


}

----------------------------------------------------------------------------
    -3.set character filter in Filter and register filter in the web.xml
    -4.inport static resources.
  
