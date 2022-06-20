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


BUILD up projects: PREPARE steps
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


1.login function implementation:
 -1.code frontend code in login jsp file
 -2.config a welcome page in the web.xml

3.business layer
public interface UserService {
    //user login
    public User login(String userCode, String password);


}
4.business implement layer
-----------------------------------------------------------
public class UserServiceImpl implements UserService{
    //visit dao layer, import dao layer
    private UserDao userDao;
    public UserServiceImpl(){
        userDao = new UserDaoImpl();
    }
    public User login(String userCode, String password) {
        Connection connection = null;
        User user = null;
        try {
            connection = BaseDao.getConnection();
            //use buainess to use relevant database operate
            user = userDao.getLoginUser(connection, userCode);
        } catch (SQLException e) {
            e.printStackTrace();
        }finally {
            BaseDao.closeResource(connection, null, null);
        }
        return user;
    }
//    @Test
//    public void test(){
//        UserServiceImpl userService = new UserServiceImpl();
//        User admin = userService.login("admin", "hsp");
//        System.out.println(admin.getUserPassword());
//    }
}
------------------------------------------------------------

7. code servlet:
----------------------------------------------
package com.qilun.servlet.user;

import com.qilun.pojo.User;
import com.qilun.service.user.UserServiceImpl;
import com.qilun.util.Constants;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class LoginServlet extends HttpServlet {

    //servlet: controller layer, transfer business layer code
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("login servlet start");
        //get login and password
        String userCode = req.getParameter("userCode");
        String userPassword = req.getParameter("userPassword");

        //compare password with db, transfer business
        UserServiceImpl userService = new UserServiceImpl();
        User user = userService.login(userCode, userPassword); //get the people log in
        if (user != null){//people exist, login
            //put user info in the session
            req.getSession().setAttribute(Constants.USER_SESSION, user);
            //get in to the inner home page
            resp.sendRedirect("jsp/frame.jsp");
        }else {//cannot login, go back to login page
            req.setAttribute("error", "username or password error");
            req.getRequestDispatcher("login.jsp").forward(req, resp);
        }

    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}
---------------------------------------

8. regist servlet



Login function
logout:
remove session
----------------------------------
public class LogOutServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //remove user session
        req.getSession().removeAttribute(Constants.USER_SESSION);
        //go back to login page
        resp.sendRedirect(req.getContextPath() + "/login.jsp");//return login page, use getContextpath + login.jsp to get correct path
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}

----------------------------------
filter 
----------------------------------
package com.qilun.filter;

import com.qilun.pojo.User;
import com.qilun.util.Constants;

import javax.servlet.*;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class SysFilter implements Filter {
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws IOException, ServletException {
        //transfer session, get session
        HttpServletRequest request = (HttpServletRequest) req;
        HttpServletResponse response = (HttpServletResponse) resp;

        //filter, user session to get users
        User user = (User) request.getSession().getAttribute(Constants.USER_SESSION);

        if (user == null){//user session moved
            response.sendRedirect("/smbms/error.jsp");

        }else {
            chain.doFilter(req, resp);
        }


    }

    public void destroy() {

    }
}
----------------------------------
then register in the webxml

















































