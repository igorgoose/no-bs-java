#JDBC(запросы в базу из кода)

## Обзор JDBC
JDBC - Java Database Connectivity. Представляет собой API для работы с базой данных. 

Основные классы, которые используются в при работе с JDBC:
1. `DriverManager` - для регистрации драйвера для конкретной базы данных и получения подключений
2. `Driver` - класс, необходимый для работы с конкретной базой данных. `DriverManager` выдает подключения к бд,
   вызывая метод `connect()` у драйвера. У каждой бд есть своя библиотека с драйверами, 
   которую нужно подключать отдельно. Библиотека с драйвером `postgresql`(maven):
   
   ```
    <dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.2.23</version>
    </dependency>
   ```

3. `Connection` - класс, представляющий подключение к бд. Через него мы получаем объекты выражений(statement)
4. `Statement` - класс, в который мы передаем строку запроса для исполнения. После передачи в него запроса, 
   вызываем методы `executeQuery`(для получения данных)/`executeUpdate`(для обновления данных).
    
5. `ResultSet` - объект, который мы получаем в результате executeQuery. Можно представлять в виде сета строк.
 Итерируясь по нему, можем получать данные.

## Основной поток работы с JDBC

1. **Настройка подключения**
    1. Подключаем библиотеку с драйвером
    2. (Сначала попробовать без этого шага) Необходимо зарегистрировать драйвер бд в DriverManager. Это делается
    1 раз при запуске приложения.
    ```java
    DriverManager.registerDriver(new Driver());
    ```
    По идее, этот код не нужен, так как в коде Driver имеется следующее:
    ```java
    static {
        try {
            DriverManager.registerDriver(new Driver());
        } catch (SQLException var1) {
            throw new RuntimeException("Can't register driver!");
        }
    }
    ```
    Это значит, что драйвер регистрируется сам, когда класс будет загружен класслоадером.
    
    3. Конфигурация подключения к бд. Для подключения к нашей бд нам нужно задать несколько параметров, а именно
    куда подключаться(url), а также имя пользователя бд и его пароль. Хорошая практика — это выносить эти параметры
    в отдельный файл конфигурации .properties в директории /src/main/resources (хорошее название db.properties).
    Содержание файла может выглядеть так:
       
   ```
    url=jdbc:postgresql://localhost:5432/my_db
    user=myUser
    password=myPassword
    characterEncoding=UTF-8
   ```
   
    Используя такой url мы подключимся к базе postgresql с названием `my_db`, которая работает на сервере с адресом
    `localhost`(наш компуктер) на порту `5432`(порт по умолчанию для postgresql). `user` и `password` по умолчанию для 
    postgresql - postgres.
   
    Теперь нам нужно получить эти данные в коде. Делается 1 раз при запуске приложения. Это можно сделать так:
    
    ```java
    private static final String DB_PROPERTIES = "db.properties";
    private static final String DB_PROPERTY_URL_KEY = "url";
    private final Properties dbProperties = new Properties();
    private String dbURL;
    
    ...
    ...
   
    private void initializeProperties() throws ConnectionPoolException {
        // получаем InputStream нашего файла конфигурации
        InputStream propertiesInputStream = getClass().getClassLoader().getResourceAsStream(DB_PROPERTIES);
        if (propertiesInputStream != null) {
            try {
                // загружаем наши проперти
                dbProperties.load(propertiesInputStream);
                // url сохраним отдельно, т.к. его нам нужно будет использовать при подключении
                dbURL = dbProperties.getProperty(DB_PROPERTY_URL_KEY);
            } catch (IOException e) {
                LOGGER.error("Failed to load database properties");
                throw new ConnectionPoolException(e);
            }
        } else {
            throw new ConnectionPoolException("Failed to read database properties file");
        }
    }
    ```
    
    С конфигурацией всё. Теперь можно писать запросы.
   

2. **Запрос в базу**
    1. Подключаемся
    ```java
    Connection connection = DriverManager.getConnection(dbURL, dbProperties);
    ```
   2. Создаем запрос.
      
    Рассмотрим следующий пример. Пусть у нас есть таблица `people` с колонками `id` - bigint/bigserial, 
    `name` - varchar, `age` - int. Мы хотим получить все записи и поместить их в класс Person:
   
    ```java
    public class Person {
        private long id;
        private String name;
        private int age;
        
        // геттеры, сеттеры, equals, hashcode
    }
    ```
   
    Наш запрос выглядит так:
    
    ```postgresql
    select * from people
    ```
    
    Выполняем его:
    
    ```java
    String queryString = 'select * from people';
    Statement statement = connection.createStatement();
    ResultSet resultSet = statement.executeQuery(queryString);
    ```
   
    Теперь самая неприятная часть: извлекаем данные из `resultSet` и записываем их в `Person`.
   
    ```java
    List<Person> people = new ArrayList<>();
    while (resultSet.next()) {
       Person person = new Person();
       // также можно получить значение по номеру колонки в результате запроса
       // по-моему номера колонок начинаются с 1
       person.setId(resultSet.getLong("id"));
       person.setName(resultSet.getString("name"));
       person.setAge(resultSet.getLong("age"));
       people.add(person);
    }
    ```
   
    Всё! У нас есть лист с людьми из базы. 
   
    Рассмотрим следующую задачу: нужно выбрать людей с определенным именем. Наш запрос: 

    ```postgresql
    select * from people where name = '{name_parameter}'
    ```
   
    Итак, действуем по аналогии. Но так как параметр в запросе мы не знаем, нам нужно вставлять его вручную.
   ```java
    List<Person> getPeopleByName(String name){
        try (Connection connection = DriverManager.getConnection(dbURL, dbProperties);
                Statement statement = connection.createStatement()) {
            String queryString = "select * from people where name = '" + name + "'";
            ResultSet resultSet = statement.executeQuery(queryString);
            // заполняем пользователей по аналогии
            return people;
        } // catch ...
   }
    ```
   
    Казалось бы, все работает хорошо. Или нет? Что если я передам строку `"' or ' ' = ' "`? 
   А что если `"'; delete from people where ' ' = ' "`? Ответ: в первом случае мы выберем всех пользователей, 
   во втором очистим таблицу. Как быть? Использовать `PreparedStatement`. Если вкратце, `PreparedStatement` 
   используется для запросов с параметрами, сам расставляет кавычки и в целом заботится о предотвращении 
   SQL-инъекций, а также запросы, выполненные через `PreparedStatement` кэшируются в базе. Статейки на эту тему на 5 минут 
   [здесь](https://www.baeldung.com/java-statement-preparedstatement) и 
   [здесь](https://stackoverflow.com/questions/3271249/difference-between-statement-and-preparedstatement).
   Так будет выглядеть наш метод при использовании `PreparedStatement`:
   
    ```java
    List<Person> getPeopleByName(String name){
        String queryString = "select * from people where name = ?"; // ? там где нужно вставить параметр
        try (Connection connection = DriverManager.getConnection(dbURL, dbProperties);
                PreparedStatement preparedStatement = connection.prepareStatement(queryString)) {
            preparedStatement.setString(1, name); // номер вопросика начиная с 1
            ResultSet resultSet = preparedStatement.executeQuery();
            // заполняем пользователей по аналогии
            return people;
        } // catch ...
   }
    ```
   
    Кратко рассмотрим выполнение запросов обновления (`INSERT`, `UPDATE`, `DELETE`). Пусть нам надо создать 
   нового человека с заданными именем и возрастом и вернуть его. Обрати внимание, что его id мы не знаем,
   оно сгенерится в базе. Код:
   
    ```java
    Person createPerson(String name, int age){
        String queryString = "insert into people(name, age) values (?, ?)";
        try (Connection connection = DriverManager.getConnection(dbURL, dbProperties);
                PreparedStatement preparedStatement = 
                    connection.prepareStatement(queryString, Statement.RETURN_GENERATED_KEYS)) {
            preparedStatement.setString(1, name); 
            preparedStatement.setInt(2, age); 
            int affectedRows = preparedStatement.executeUpdate();
            
            if (affectedRows == 0) {
                throw new SQLException("Creating user failed, no rows affected.");
            }
            Person person = new Person();
            person.setName(name);
            person.setAge(age);
            try (ResultSet generatedKeys = preparedStatement.getGeneratedKeys()) {
                if (generatedKeys.next()) {
                    person.setId(generatedKeys.getLong(1));
                }
                else {
                    throw new SQLException("Creating person failed, no ID obtained.");
                }
            }
            return person;
        } // catch ...
   }
    ```
   
    Я взял этот пример, потому что здесь нам нужно получить сгенеренный `id` новой записи. Если оно не нужно, то мы 
    просто выполняем `executeUpdate` и не заморачиваемся. Итак, разберемся, что происходит. При получении `PreparedStatement`
   нам нужно указать параметр `Statement.RETURN_GENERATED_KEYS`, чтобы мы могли получить наш `id`. Далее заполняем запрос,
   получаем количество измененных записей, проверяем, что все ок(ни разу так не делал, если не ок,
   то по идее выскочит SQLException). Далее вызываем метод `preparedStatement.getGeneratedKeys()`, получаем `id` и 
   записываем его в `person`. На этом все.


## Задание
1. Создать классы `Student`, `Payment` для данных из соответствующих таблиц. В `Student` должна быть коллекция `Payment`
   (т.к. связь один ко многим).
   
2. Настроить подключение к базе, создать класс DAO для `Student` (он должен быть singleton).

3. Создать функционал для получения студента с его платежами(можно выводить `Student` в консоль через toString()).
   `Student` должен выводиться со списком его платежей.

4. Создать функционал для создания студента.

5. Создать функционал для удаления студента по `id`.

P.S. Схема обработки запроса:
1. `Контроллер` получает запрос
2. `Контроллер` передает данные из запроса в `сервис`
3. `Сервис` проводит валидацию данных из запроса(для пункта 3 можно что-то типа проверки что `id` неотрицательный)
4. `Сервис` обращается к `DAO` за данными(список студентов), либо просит обновить данные.
5. `DAO` идет в базу, заполняет объекты и возвращает сервису, либо обновляет данные.
