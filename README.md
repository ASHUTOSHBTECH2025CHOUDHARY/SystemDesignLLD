"# SystemDesignLLD" 
# Creational Design Patterns — Complete LLD Guide

Creational patterns ka kaam hai **object creation ko flexible aur decoupled banana**. Matlab, jab bhi `new` keyword use karke object banane ka logic complex ho jaaye, ya future me implementation change hone ka chance ho — tab creational pattern kaam aata hai.

**5 Creational Patterns:**
1. Singleton
2. Factory Method
3. Abstract Factory
4. Builder
5. Prototype

Interview me sabse zyada pucha jaata hai: **Singleton, Factory, Builder**. Abstract Factory aur Prototype thoda kam frequent hai but concept clear hona chahiye.

---

## 1. Singleton Pattern

### Kya hai?
Ek class ka **sirf ek hi instance** poori application me exist kare, aur usko globally access karne ka ek point ho.

### Real-world analogy
Government ka Prime Minister — ek time par ek hi hota hai, aur sab log usi ek office se contact karte hain. Nayi PM office nahi bana sakte.

### Kab use karo?
- Database connection pool
- Logger class
- Configuration manager
- Cache manager

### Code — Thread-Safe Singleton (Double-Checked Locking)

```java
public class DatabaseConnectionManager {

    // volatile ensures visibility across threads
    private static volatile DatabaseConnectionManager instance;

    private String connectionUrl;

    // private constructor -> koi bahar se 'new' nahi kar sakta
    private DatabaseConnectionManager() {
        this.connectionUrl = "jdbc:mysql://localhost:3306/oms_db";
        System.out.println("Connection Manager Initialized");
    }

    public static DatabaseConnectionManager getInstance() {
        if (instance == null) {                       // 1st check - no locking overhead
            synchronized (DatabaseConnectionManager.class) {
                if (instance == null) {                // 2nd check - thread safety
                    instance = new DatabaseConnectionManager();
                }
            }
        }
        return instance;
    }

    public String getConnectionUrl() {
        return connectionUrl;
    }
}
```

### Better Approach — Enum Singleton (Effective Java recommends this)

```java
public enum ConfigManager {
    INSTANCE;

    private final String appName = "OMS-Stellantis";

    public String getAppName() {
        return appName;
    }
}

// Usage: ConfigManager.INSTANCE.getAppName();
```

Enum approach automatically thread-safe hai aur serialization/reflection attacks se bhi safe hai — interview me ye point bolna impress karta hai.

### Pitfalls (Interviewer ye zaroor poochega)
- Reflection se bhi private constructor break kiya ja sakta hai (Enum isse immune hai).
- Serialization ke baad naya object ban sakta hai agar `readResolve()` implement nahi kiya.
- Multithreaded environment me bina `synchronized`/`volatile` ke race condition aa sakti hai — do threads ek saath `null` dekh kar 2 instance bana sakte hain.

---

## 2. Factory Method Pattern

### Kya hai?
Object creation ka logic ek **subclass/factory method** ke andar daal do, taaki calling code ko exact class ka naam pata na ho. Parent class ek interface define karta hai, child classes decide karti hain **konsa object banana hai**.

### Real-world analogy
Pizza store — tum "Margherita" order karte ho, tumhe pata nahi kaunsa chef kaunsa dough use karega. Factory (store) decide karta hai object (pizza) kaise banega.

### Kab use karo?
- Jab object creation logic complex ho aur future me naye types add hone wale ho
- Jab client ko concrete class se decouple rakhna ho

### Code

```java
// Step 1: Product interface
interface Notification {
    void notifyUser();
}

// Step 2: Concrete Products
class EmailNotification implements Notification {
    public void notifyUser() {
        System.out.println("Sending Email Notification");
    }
}

class SMSNotification implements Notification {
    public void notifyUser() {
        System.out.println("Sending SMS Notification");
    }
}

class PushNotification implements Notification {
    public void notifyUser() {
        System.out.println("Sending Push Notification");
    }
}

// Step 3: Creator (Factory) - abstract method defines "what to create"
abstract class NotificationFactory {

    // Factory Method
    protected abstract Notification createNotification();

    // Common logic that uses the factory method (template-ish behaviour)
    public void send() {
        Notification notification = createNotification();
        notification.notifyUser();
    }
}

// Step 4: Concrete Creators
class EmailNotificationFactory extends NotificationFactory {
    protected Notification createNotification() {
        return new EmailNotification();
    }
}

class SMSNotificationFactory extends NotificationFactory {
    protected Notification createNotification() {
        return new SMSNotification();
    }
}

// Step 5: Client code
public class FactoryMethodDemo {
    public static void main(String[] args) {
        NotificationFactory factory = new EmailNotificationFactory();
        factory.send();   // Output: Sending Email Notification

        NotificationFactory smsFactory = new SMSNotificationFactory();
        smsFactory.send(); // Output: Sending SMS Notification
    }
}
```

### Simple Factory (Non-GoF but interview me casual pucha jaata hai)

```java
class SimpleNotificationFactory {
    public static Notification getNotification(String type) {
        switch (type.toUpperCase()) {
            case "EMAIL": return new EmailNotification();
            case "SMS":   return new SMSNotification();
            case "PUSH":  return new PushNotification();
            default: throw new IllegalArgumentException("Unknown type: " + type);
        }
    }
}
```

**Note:** Ye "Simple Factory" technically GoF pattern nahi hai, ye ek *idiom* hai. Real Factory Method me inheritance + polymorphism involve hota hai (jaisa upar wale example me hai).

---

## 3. Abstract Factory Pattern

### Kya hai?
Ye Factory Method ka "factory of factories" version hai. Ye **related objects ka poora family** create karta hai without specifying unki concrete classes.

### Real-world analogy
Furniture manufacturing company — agar tum "Modern" theme choose karte ho, to tumhe Modern-Chair, Modern-Sofa, Modern-Table milega ek saath, matching set me. "Victorian" theme choose karoge to Victorian family milega. Factory ensure karti hai ki mismatch na ho (Modern Chair + Victorian Sofa nahi milega).

### Kab use karo?
- Jab multiple related products ka ek consistent "family" banana ho
- Cross-platform UI (Windows vs Mac buttons/checkboxes) jaisa use case

### Code

```java
// Product Family 1: Button
interface Button {
    void render();
}
class WindowsButton implements Button {
    public void render() { System.out.println("Rendering Windows Button"); }
}
class MacButton implements Button {
    public void render() { System.out.println("Rendering Mac Button"); }
}

// Product Family 2: Checkbox
interface Checkbox {
    void render();
}
class WindowsCheckbox implements Checkbox {
    public void render() { System.out.println("Rendering Windows Checkbox"); }
}
class MacCheckbox implements Checkbox {
    public void render() { System.out.println("Rendering Mac Checkbox"); }
}

// Abstract Factory interface
interface GUIFactory {
    Button createButton();
    Checkbox createCheckbox();
}

// Concrete Factory 1 - creates Windows family
class WindowsFactory implements GUIFactory {
    public Button createButton() { return new WindowsButton(); }
    public Checkbox createCheckbox() { return new WindowsCheckbox(); }
}

// Concrete Factory 2 - creates Mac family
class MacFactory implements GUIFactory {
    public Button createButton() { return new MacButton(); }
    public Checkbox createCheckbox() { return new MacCheckbox(); }
}

// Client
public class AbstractFactoryDemo {

    private Button button;
    private Checkbox checkbox;

    public AbstractFactoryDemo(GUIFactory factory) {
        button = factory.createButton();
        checkbox = factory.createCheckbox();
    }

    public void renderUI() {
        button.render();
        checkbox.render();
    }

    public static void main(String[] args) {
        String osName = "Windows"; // ye runtime pe decide hota hai

        GUIFactory factory = osName.equals("Windows") ? new WindowsFactory() : new MacFactory();
        AbstractFactoryDemo app = new AbstractFactoryDemo(factory);
        app.renderUI();
    }
}
```

### Factory Method vs Abstract Factory (Interview favourite question)

| Point | Factory Method | Abstract Factory |
|---|---|---|
| Kya banata hai | Ek single product | Related products ka family |
| Mechanism | Inheritance (subclass overrides method) | Composition (factory object inject hota hai) |
| Complexity | Simple | Zyada complex, multiple factories |
| Example | Notification (Email/SMS) | UI Toolkit (Windows/Mac) |

---

## 4. Builder Pattern

### Kya hai?
Jab kisi object ke paas **bohot saare optional fields** hote hain (telescoping constructor problem), tab Builder pattern step-by-step, readable tarike se object construct karne deta hai.

### Real-world analogy
Subway sandwich banwana — bread choose karo, phir veggies, phir sauce, phir toasting. Har step optional/customizable hai, aur end me ek complete sandwich (object) milta hai. Tumhe ek hi constructor call karke sab kuch ek saath nahi bolna padta.

### Problem jo Builder solve karta hai (Telescoping Constructor)

```java
// BAD approach - too many constructor overloads
public class User {
    public User(String name) {}
    public User(String name, int age) {}
    public User(String name, int age, String email) {}
    public User(String name, int age, String email, String address) {}
    // ... aur bhi combinations - unmanageable
}
```

### Code — Builder Pattern (Static Nested Builder, Effective Java style)

```java
public class User {
    // required fields
    private final String name;
    private final String email;

    // optional fields
    private final int age;
    private final String address;
    private final String phoneNumber;

    // private constructor - only Builder can access
    private User(UserBuilder builder) {
        this.name = builder.name;
        this.email = builder.email;
        this.age = builder.age;
        this.address = builder.address;
        this.phoneNumber = builder.phoneNumber;
    }

    @Override
    public String toString() {
        return "User{name='" + name + "', email='" + email + "', age=" + age +
               ", address='" + address + "', phone='" + phoneNumber + "'}";
    }

    // Static nested Builder class
    public static class UserBuilder {
        private final String name;   // required
        private final String email;  // required

        private int age;             // optional - default 0
        private String address;      // optional
        private String phoneNumber;  // optional

        public UserBuilder(String name, String email) {
            this.name = name;
            this.email = email;
        }

        public UserBuilder age(int age) {
            this.age = age;
            return this;             // returning 'this' enables method chaining
        }

        public UserBuilder address(String address) {
            this.address = address;
            return this;
        }

        public UserBuilder phoneNumber(String phoneNumber) {
            this.phoneNumber = phoneNumber;
            return this;
        }

        public User build() {
            return new User(this);
        }
    }
}

// Client usage - readable, fluent chaining
public class BuilderDemo {
    public static void main(String[] args) {
        User user = new User.UserBuilder("Ashutosh", "ashutosh@mail.com")
                        .age(25)
                        .address("Mumbai")
                        .phoneNumber("9999999999")
                        .build();

        System.out.println(user);
    }
}
```

### Real-world usage jo tum already jaante ho
`StringBuilder`, `Lombok @Builder`, `okhttp3.Request.Builder()` — sab isi pattern pe based hain. Spring Boot me `UriComponentsBuilder` bhi Builder pattern hi hai.

### Interview Tip
Agar interviewer bole "Immutable object kaise banaoge with 10 optional fields?" — seedha Builder Pattern ka naam lo aur ye code likho. Ye ekdum classic LLD question hai (jaise tum Parking Lot System me `TicketBuilder` bana sakte ho).

---

## 5. Prototype Pattern

### Kya hai?
Naya object banane ke bajaye, **existing object ko clone (copy)** karke naya object banate hain — jab object creation costly ho (DB call, heavy computation, etc.)

### Real-world analogy
Photocopy machine — original document se milta-julta copy nikaal lo, bina dobara se pura document type kiye. Chahiye to copy me thoda modification kar do.

### Kab use karo?
- Jab object creation expensive ho (e.g., DB se data fetch karke object banana)
- Jab tumhe same base object ki multiple slightly-different copies chahiye

### Code — Shallow vs Deep Copy (dono interview me pucha jaata hai)

```java
import java.util.ArrayList;
import java.util.List;

class Address {
    String city;
    Address(String city) { this.city = city; }
}

class Employee implements Cloneable {
    String name;
    int age;
    Address address;              // reference type - shallow copy issue yahin aayega
    List<String> skills;

    Employee(String name, int age, Address address, List<String> skills) {
        this.name = name;
        this.age = age;
        this.address = address;
        this.skills = skills;
    }

    // SHALLOW COPY - default clone(), reference fields share same object
    @Override
    public Employee clone() throws CloneNotSupportedException {
        return (Employee) super.clone();
    }

    // DEEP COPY - manually copy nested objects too
    public Employee deepClone() {
        Address newAddress = new Address(this.address.city);
        List<String> newSkills = new ArrayList<>(this.skills);
        return new Employee(this.name, this.age, newAddress, newSkills);
    }

    @Override
    public String toString() {
        return name + " | " + age + " | " + address.city + " | " + skills;
    }
}

public class PrototypeDemo {
    public static void main(String[] args) throws CloneNotSupportedException {
        Employee original = new Employee("Ashutosh", 25,
                new Address("Mumbai"), new ArrayList<>(List.of("Java", "Spring")));

        Employee shallowCopy = original.clone();
        Employee deepCopy = original.deepClone();

        // modifying original's nested address object
        original.address.city = "Pune";

        System.out.println("Original: " + original);
        System.out.println("Shallow Copy: " + shallowCopy);  // city also changes to Pune! (shared reference)
        System.out.println("Deep Copy: " + deepCopy);        // city stays Mumbai (independent copy)
    }
}
```

### Shallow vs Deep Copy — Quick Table

| Copy Type | Primitive fields | Object references |
|---|---|---|
| Shallow Copy | New copy | **Same reference shared** — changes reflect in both |
| Deep Copy | New copy | **Independent new object** — changes isolated |

---

## Quick Recap Table — Sab Patterns Ek Nazar Me

| Pattern | Problem Solve Karta Hai | Real Example |
|---|---|---|
| Singleton | Ek hi instance chahiye globally | DB Connection, Logger |
| Factory Method | Object creation ko subclass pe delegate karo | Notification type based creation |
| Abstract Factory | Related objects ka family banao | Cross-platform UI toolkit |
| Builder | Complex object with many optional fields | User/Order object with StringBuilder-jaisi chaining |
| Prototype | Expensive object ko clone karke reuse karo | Employee object cloning, cache-based objects |

---

## Interview Prep Tip (LLD ke liye)

Tumhare Parking Lot System practice ke context me:
- **Singleton** → `ParkingLot` class khud singleton honi chahiye (ek hi parking lot instance).
- **Factory Method** → `VehicleFactory` jo `Car`, `Bike`, `Truck` object return kare based on vehicle type.
- **Builder** → `Ticket` object banate waqt (entry time, vehicle, slot, rate — sab optional/required mix).
- **Strategy pattern** (Behavioral) ke saath combine hoga jab fee calculation different vehicle types ke liye alag ho — wo agla topic hai (Structural/Behavioral README chahiye ho to bata dena, wo bhi bana dunga).

Agla step: chaho to isi tarah **Structural Patterns** (Adapter, Decorator, Facade, Proxy, Composite) ka README bhi bana doon?
