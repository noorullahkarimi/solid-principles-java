<div dir="rtl">

# اصول SOLID در یک نگاه

راهنمایی برای یادگیری هر چه سریع تر اصول SOLID همراه با مثال.

## جدول محتوا


  - [اصول SOLID در یک نگاه](#اصول-SOLID-در-یک-نگاه)
  - [جدول محتوا](#جدول-محتوا)
  - [مقدمه](#مقدمه)
  - [اصل اول Single Responsibility Principle (SRP)](#اصل-اول-Single-Responsibility-Principle)
  - [اصل دوم Open-Close Principle (OCP)](#اصل-دوم-Open-Close-Principle)
  - [اصل سوم Liskov Substitution Principle (LSP)](#اصل-سوم-Liskov-Substitution-Principle)
  - [اصل چهارم Interface Segregation Principle (ISP)](#اصل-چهارم-Interface-Segregation-Principle)
  - [اصل پنجم Dependency Inversion Principle (DIP)](#اصل-پنجم-Dependency-Inversion-Principle)

## مقدمه

در اکثر منابع یادگیری اصول SOLID همواره مباحث بصورت خیلی پیچیده نوشته شده است.
در این داکیومنت تلاش ما بر این بوده است که با ارائه مثال های ساده فهم این اصول را برای تمامی فارسی زبانان مهیا کنیم.
پ.ن. اگر نظر یا پیشنهادی و یا مثال های بهتری برای بهبود داکیومنت دارید خوشحال میشم در قسمت issue ها مطرح کنید.

## اصل اول Single Responsibility Principle

کلاس ها و متد ها باید یک مسئولیت را دنبال کنند.
در کد زیر شما یک مثال ساده از ذخیره ی کاربر در دیتابیس را مشاهده می کنید که اصل اول را نقض میکند.اما چرا؟

بد:

```java
public class RegistrationService {
    public void registerUser(String name, String email) {
        if (validateName(name) && validateEmail(email)) {
            saveToDatabase(name, email);
            System.out.println("User registered successfully.");
        } else {
            System.out.println("Invalid user data. Registration failed.");
        }
    }

    private boolean validateName(String name) {
        return name != null && !name.isEmpty();
    }

    private boolean validateEmail(String email) {
        return email != null && email.contains("@");
    }

    private void saveToDatabase(String name, String email) {
        // کد برای ذخیره در دیتابیس
    }
}
```

در کد بالا سه وظیفه در یک کلاس قرار داده شده است.

- ذخیره در دیتابیس
- اعتبارسنجی داده های ورودی(Validation)
- اتصال به دیتابیس

ما این وظایف را از هم جدا و در کلاس های جداگانه قرار می دهیم.

خوب:

```java
public class RegistrationService {

// وظیفه اول : ذخیره در دیتابیس

    private UserRepository userRepository;

    public RegistrationService() {
        userRepository = new UserRepository();
    }

    public void registerUser(String name, String email) {
        User user = new User(name, email);
        if (user.isValid()) {
            userRepository.save(user);
            System.out.println("User registered successfully.");
        } else {
            System.out.println("Invalid user data. Registration failed.");
        }
    }
}

// وظیفه ی دوم : اعتبار سنجی
public class User {
    private String name;
    private String email;

    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }

    public boolean isValid() {
        return name != null && !name.isEmpty() && email != null && email.contains("@");
    }
}

// وظیفه ی سوم : اتصال به دیتابیس
public class UserRepository {
    public void save(User user) {
        // کد برای ذخیره در دیتابیس
        System.out.println("User saved to database.");
    }
}

```

## اصل دوم Open-Close Principle

کامپوننت ها (کلاس، ماژول ها،متدها ) باید برای توسعه باز و برای تغییر بسته باشند.(اضافه کردن امکانات بدون تغییر در هسته ی اصلی کد)

در کد زیر اگر بخواهیم یک قسمت برای شکل شش ضلعی اضافه کنیم
باید در کلاس Shape تغییراتی ایجاد کنیم.(نقض اصل دوم سالید)


بد:
```java
public class Shape {
    private String type;

    public Shape(String type) {
        this.type = type;
    }

    public void draw() {
        if (type.equals("square")) {
            System.out.println("Drawing a square.");
        } else if (type.equals("circle")) {
            System.out.println("Drawing a circle.");
        } else if (type.equals("triangle")) {
            System.out.println("Drawing a triangle.");
        }
    }
}
```
در کد زیر اگر بخواهیم که شش ضلعی اضافه کنیم، کافیست یک کلاس برای آن بسازیم.

خوب:
```java
public interface Drawable {
    void draw();
}

public class Square implements Drawable {
    public void draw() {
        System.out.println("Drawing a square.");
    }
}

public class Circle implements Drawable {
    public void draw() {
        System.out.println("Drawing a circle.");
    }
}

public class Triangle implements Drawable {
    public void draw() {
        System.out.println("Drawing a triangle.");
    }
}


public class DrawingApp {
    public Shape(String type) {
       //ساخت شئ از String type
       //صدا زدن متد draw
       //نکته : همه ی کلاس های shape باید متد draw داشته باشند، به همین خاطر باید از قرارداد(interface) Drawable، implement کرده باشند.
    }
}
```

## اصل سوم Liskov Substitution Principle

تمامی کلاس هایی که از یک interface، implements می شوند، باید حتما تمامی متد های داخل آن را کامل پیاده سازی کنند.
(حتی آرگومان های پاس داده شده و حتی نوع داده برگشت داده شده باید همانند باشند)

در مثال آورده شده کلاس Birds به دلیل آنکه موتور ندارد نمیتوان آنرا پیاده سازی کرد.

بد:
```java
public interface Details {
    void motor(int power);
    void fly();
}
public class Airplan implements Details {
    public void motor(int power) {
        System.out.println(power + " hp");
    }

    public void fly() {
        System.out.println("yes");
    }
}

public class Birds implements Details {
    public void motor(int power) {
        //این متد پیاده سازی ندارد زیرا پرنده موتور ندارد
    }

    public void fly() {
        System.out.println("yes");
    }
}
```


خوب:
```java
public interface Details {
    void motor(int power);
    void fly();
}
// نوشتن interface (قرارداد ) دیگر
public interface Details2 {
    void fly();
}

public class Airplan implements Details {
    public void motor(int power) {
        System.out.println(power + "hp");
    }

    public void fly() {
        System.out.println("yes");
    }
}
// استفاده از interface Details2
public class Birds implements Details2 {
    public void fly() {
        System.out.println("yes");
    }
}
```

## اصل چهارم Interface Segregation Principle

این اصل شباهت بسیار زیاد به اصل سوم دارد. با این تفاوت که اصل چهارم تمرکز روی طراحی interface ها دارد،
اما اصل سوم روی کامل پیاده سازی شدن interface ها.

مثال قبل را با طراحی interface جدید بررسی می کنیم.

بد:
```java
public interface Details {
    void motor(int power);
    void fly();
}

public interface Details2 {
    void fly();
}

public class Airplan implements Details {
    public void motor(int power) {
        System.out.println(power + "hp");
    }

    public void fly() {
        System.out.println("yes");
    }
}

public class Birds implements Details2 {
    public void fly() {
        System.out.println("yes");
    }
}
```
خوب:
```java
public interface Details {
    void motor(int power);
}

public interface Details2 {
    void fly();
}

public class Airplan implements Details, Details2{
    public void motor(int power) {
        System.out.println(power + "hp");
    }

    public void fly() {
        System.out.println("yes");
    }
}

public class Birds implements Details2 {
    public void fly() {
        System.out.println("yes");
    }
}
```

## اصل پنجم Dependency Inversion Principle


کلاس های ما نباید مستقیما به زیر کلاس ها وابستگی داشته باشند، به جای آن باید به abstractions(interface) ها وابستگی داشته باشند.

اگر بخواهیم به جای Mysql از MongoDB استفاده کنیم، کلاس User تغییر خواهد کرد زیرا مستقیما وابستگی ایجاد شده است.
پس ما بجای وابستگی مستقیم از Interface استفاده می کنیم.

بد:
```java

class MysqlCRUD {
    public void save(String name){
        // کدهای ذخیره
    }

    public void edit(){
        //کد های تغییر اطلاعات 
    }

    public void delete(){
        //کد های حذف 
    }
    
}

class User {
    public void addUser(){
        MysqlCRUD crud = new MysqlCRUD();
        crud.save("john");
    }
}

public class Run {
    public static void main(String[] args) {
        User user = new User();
        user.addUser();
    }
}
```

خوب:
```java
public interface DbAction {
    void save(String name)
    void edit();
    void delete();
}

//اضافه شدن دیتابیس مونگو
class MongoCRUD implements DbAction{
    public void save(String name){
        // کدهای ذخیره
    }

    public void edit(){
        //کد های تغییر اطلاعات 
    }

    public void delete(){
        //کد های حذف 
    }
    
}

class MysqlCRUD implements DbAction{
    public void save(String name){
        // کدهای ذخیره
    }

    public void edit(){
        //کد های تغییر اطلاعات 
    }

    public void delete(){
        //کد های حذف 
    }
    
}

class User {

    private DbAction dbAction;

    User (DbAction dbAction){
        this.dbAction = dbAction;
    }

    public void addUser(){
        dbAction.save("john");
    }
}

public class Run {
    public static void main(String[] args) {
        DbAction dbAction = new MongoCRUD();
        User user = new User(dbAction);
        user.addUser();
    }
}

```
