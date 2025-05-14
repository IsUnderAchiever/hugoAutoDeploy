---
title: Jsp
description: Jsp
date: 2023-03-24
slug: Jsp
image: 202412211853801.png
categories:
    - Java
---

## Servlet
> ### 项目创建

![项目创建](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303222125366.png)

![配置1](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303222127960.png)

> 删掉pom.xml

![image-20230322213236109](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303222132185.png)

![image-20230322213258427](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303222132497.png)

![image-20230322213314271](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303222133344.png)

![image-20230322213335864](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303222133936.png)

![image-20230322213347376](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303222133445.png)

> 点击运行即可

![image-20230322213425998](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303222134032.png)

![image-20230322213433714](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303222134753.png)

> 因为没有页面

![image-20230322213617014](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303222136177.png)

![image-20230322213631081](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303222136111.png)

> 在WEB-INF下新建lib包，在lib下存放jar包

![image-20230322213928118](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303222139162.png)

> 此时依赖还没完全导入，还需如下操作

![image-20230322214007002](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303222140082.png)

![image-20230322214026141](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303222140175.png)

> 选择lib，此时就算导入成功了

![image-20230322214102932](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303222141981.png)

> 网上copy一个jdbc的工具包
```java
package com.example.util;
import org.apache.commons.beanutils.BeanUtils;
import java.lang.reflect.Field;
import java.sql.*;
import java.util.ArrayList;
import java.util.List;
public class JDBCUtil {
    //1.加载驱动
    static {
        try {
            Class.forName("com.mysql.jdbc.Driver");
        } catch (ClassNotFoundException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
    //2.获取连接
    public static Connection getConnection() {
        Connection conn = null;
        try {
            conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/xiaoy", "root", "mysql");
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return conn;
    }
    //3.关闭连接
    public static void close(Connection conn, Statement st, ResultSet rs) {
        //关闭连接
        if (conn != null) {
            try {
                conn.close();
            } catch (SQLException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
        //关闭statement
        if (st != null) {
            try {
                st.close();
            } catch (SQLException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
        //关闭结果集
        if (rs != null) {
            try {
                rs.close();
            } catch (SQLException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
    }
    //-------------------------------封装sql操作------------------------------
    //查询返回List集合
    public static <T> List<T> getList(Class<T> cls, String sql, Object... obj) {
        Connection conn = null;
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
            //1.获取连接
            conn = getConnection();
            //2.获取预处理对象
            ps = conn.prepareStatement(sql);
            //循环参数，如果没有就不走这里
            for (int i = 1; i <= obj.length; i++) {
                //注意：数组下标从0开始，预处理参数设置从1开始
                ps.setObject(i, obj[i - 1]);
            }
            //3.执行SQL语句
            System.out.println(sql);
            rs = ps.executeQuery();
            //4.遍历结果集
            //遍历之前准备：因为封装不知道未来会查询多少列，所以我们需要指定有多少列
            //获取ResultSet对象的列编号、类型和属性
            ResultSetMetaData date = rs.getMetaData();
            //获取列数
            int column = date.getColumnCount();
            //获取本类所有的属性
            Field[] fields = cls.getDeclaredFields();
            //创建一个list集合对象来存储查询数据
            List<T> list = new ArrayList<T>();
            //开始遍历结果集
            while (rs.next()) {
                //创建类类型实例
                T t = cls.newInstance();
                for (int i = 1; i <= column; i++) {
                    //每一列的值
                    Object value = rs.getObject(i);
                    /**
                     *String columnName = date.getColumnName(i);//获取每一列名称
                     * 关于获取每一列名称，如果列取了别名的话，则不能用上面的方法取列的名称
                     * 用下面的方法
                     */
                    String columnName = date.getColumnLabel(i);//获取每一列名称（别名）
                    //遍历所有属性对象
                    for (Field field : fields) {
                        //获取属性名
                        String name = field.getName();
                        field.setAccessible(true);//打破封装，忽略对封装修饰符的检测
						/*if (name.equals(columnName)) {
                            //获取列类型名称
							String string = date.getColumnTypeName(i);
							//如果列类型是Date类型，转换成字符串表现形式
							SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
							String d = sdf.format(value);
							//赋值：将数据库中查询的字段赋值给对应名称的属性
							field.set(t, d);
						}else{
							field.set(t, value);
						}*/
                        if (name.equals(columnName)) {
                            BeanUtils.copyProperty(t, name, value);
                            break;//增加效率，避免不必要的循环
                        }
                    }
                }
                list.add(t);
            }
            return list;
            //5.关闭连接
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JDBCUtil.close(conn, ps, rs);
        }
        return null;
    }
    /**
     * 增加、删除、修改
     *
     * @param sql sql语句
     * @param obj 参数
     */
    public static boolean getDML(String sql, Object... obj) {
        Connection conn = null;
        PreparedStatement ps = null;
        try {
            conn = getConnection();
            ps = conn.prepareStatement(sql);
            for (int i = 1; i <= obj.length; i++) {
                ps.setObject(i, obj[i - 1]);
            }
            System.out.println(sql);
            int update = ps.executeUpdate();
            if (update > 0) {
                return true;
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            close(conn, ps, null);
        }
        return false;
    }
    //查询返回单个对象
    public static <T> T getOneObject(Class<T> cls, String sql, Object... obj) {
        Connection conn = null;
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
            //1.获取连接
            conn = getConnection();
            //2.获取预处理对象
            ps = conn.prepareStatement(sql);
            //循环参数，如果没有就不走这里
            for (int i = 1; i <= obj.length; i++) {
                //注意：数组下标从0开始，预处理参数设置从1开始
                ps.setObject(i, obj[i - 1]);
            }
            //3.执行SQL语句
            System.out.println(sql);
            rs = ps.executeQuery();
            //4.遍历结果集
            //遍历之前准备：因为封装不知道未来会查询多少列，所以我们需要指定有多少列
            //获取ResultSet对象的列编号、类型和属性
            ResultSetMetaData date = rs.getMetaData();
            //获取列数
            int column = date.getColumnCount();
            //获取本类所有的属性
            Field[] fields = cls.getDeclaredFields();
            //开始遍历结果集
            if (rs.next()) {
                //创建类类型实例
                T t = cls.newInstance();
                for (int i = 1; i <= column; i++) {
                    //每一列的值
                    Object value = rs.getObject(i);
                    //获取每一列名称
                    String columnName = date.getColumnName(i);
                    //遍历所有属性对象
                    for (Field field : fields) {
                        //获取属性名
                        String name = field.getName();
                        //打破封装，忽略对封装修饰符的检测
                        field.setAccessible(true);
                        if (name.equals(columnName)) {
                            BeanUtils.copyProperty(t, name, value);
                        }
                    }
                }
                return t;
            }
            //5.关闭连接
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JDBCUtil.close(conn, ps, rs);
        }
        return null;
    }
    //查询总记录数
    public static Integer getCount(String sql, Object... obj) {
        Connection conn = null;
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
            //1.获取连接
            conn = getConnection();
            //2.获取预处理对象
            ps = conn.prepareStatement(sql);
            //循环参数，如果没有就不走这里
            for (int i = 1; i <= obj.length; i++) {
                //注意：数组下标从0开始，预处理参数设置从1开始
                ps.setObject(i, obj[i - 1]);
            }
            //3.执行SQL语句
            System.out.println(sql);
            rs = ps.executeQuery();
            //开始遍历结果集
            if (rs.next()) {
                return rs.getInt(1);
            }
            //5.关闭连接
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JDBCUtil.close(conn, ps, rs);
        }
        return null;
    }
}
```
> BeanUtils需要导包，`commons-beanutils-1.9.4.jar`
```xml
<dependency>
    <groupId>commons-beanutils</groupId>
    <artifactId>commons-beanutils</artifactId>
    <version>1.9.4</version>
</dependency>
```
> 如果需要连接oracle，则jdbcUtil需要改改配置
>
> 导入jquery，使用ajax，[下载网址](https://cdn.staticfile.org/jquery/1.12.4/jquery.min.js)

![image-20230322215453085](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303222154368.png)

![image-20230322215530812](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303222155920.png)

> 配置webservlet
>
> 我这里@WebServlet注解爆红，导入`javax.servlet-api-3.1.0.jar`包后解决
> 看到有博客说`servlet.jar`也可以，不过我就导入了`javax.servlet-api-3.1.0.jar`就已解决

![image-20230322220056475](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303222200659.png)

![image-20230322220646463](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303222206576.png)

![image-20230322220429739](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303222204889.png)

> 注释掉SubHandle内的`super.doPost(req, resp);`即可，`记得重启服务`

![image-20230322220511121](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303222205247.png)

> ## 前端之前一直接受不到数据
>
> 既不执行success也不执行error，是因为后端的返回格式不是json

![image-20230323074135515](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303230741695.png)
