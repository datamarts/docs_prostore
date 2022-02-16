---
layout: default
title: Программное подключение
nav_order: 2
parent: Подключение
grand_parent: Работа с системой
has_children: false
has_toc: false
---

# Программное подключение {#connection_via_code}

JDBC-драйвер системы позволяет подключаться программно (без использования SQL-клиента). Вы можете 
реализовать свое приложение, работающее с системой через JDBC-подключение.

Чтобы подключиться к системе с помощью программного подключения:
1.  Загрузите скомпилированный файл драйвера с именем `dtm-jdbc-<version>.jar`.
2.  Определите путь к jar-файлу драйвера любым из способов:
    1.  задайте путь с помощью переменной окружения `CLASSPATH`;
    2.  задайте путь в командной строке при запуске своего приложения (формат зависит от операционной 
        системы):
        ```java
        java -classpath /<path-to-driver>/dtm-jdbc-<version>.jar myapplication.class
        ```
        
3.  В реализации класса вашего приложения, который отвечает за подключение к системе (см. пример [ниже](#ex_connection_class)):
    1.  импортируйте пакеты Java SQL:
        ```java
        import java.sql.*;
        ```

    2.  если используется Java версии менее 1.6, загрузите драйвер в память:
        ```java        
        Class.forName("ru.datamart.prostore.jdbc.Driver");
        ```

    3.  установите соединение с системой с помощью метода `DriverManager.getConnection()` в следующем 
        формате:
        ```java  
        String url = "jdbc:prostore://DtmHost:portNumber/logicalDatabaseName";
        Connection conn = DriverManager.getConnection(url, null, null);
        ```
    
Строка `url` содержит параметры:
*   `DtmHost` — IP-адрес или имя хоста, на котором установлена система;
*   `portNumber` — номер порта для подключения;
*   (опционально) `logicalDatabaseName` — имя логической базы данных, используемой по умолчанию.

Пример `url`:
```java  
String url = "jdbc:prostore://10.92.3.3:9092/demo";
Connection conn = DriverManager.getConnection(url, null, null);
```

После установки соединения можно выполнять [запросы SQL+](../../../reference/sql_plus_requests/sql_plus_requests.md). 
По окончании работы с системой нужно закрыть подключение.

<a id="ex_connection_class"></a>
В примере ниже показана базовая реализация класса `SimpleDtmJDBCExample`, который устанавливает соединение 
с системой по заданному адресу и затем закрывает соединение.
```java 
import java.sql.*;
public class SimpleDtmJDBCExample {
    public static void main(String[] args) {
        Connection conn;
        String url = "jdbc:prostore://10.92.3.3:9092/demo";
        try {
            conn = DriverManager.getConnection(url);
            System.out.println("Connected");
        } catch (SQLException e) {
            // Catch all for the SQL exceptions
            e.printStackTrace();
        } finally {
            conn.close();
        }
}
```
