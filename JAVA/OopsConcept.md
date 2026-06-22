# Method overloading

## what is method overloading?
Method overloading allows you to use the same method name for different tasks as long as their parameter types or numbers differ. 

## why we need this concept??
Improved Readability & Clean APIs: Instead of naming methods calculateAreaInt, calculateAreaDouble, or calculateAreaCircle, you can simply name all of them calculateArea. This makes APIs intuitive and highly expressive.

Reduced Complexity: You only have to memorize one method name for a specific action, reducing the cognitive load on developers.

Code Reusability: It saves memory and lines of code. For example, the System.out.println() method is heavily overloaded, allowing you to print strings, integers, and booleans without writing distinct printing logic for every data type.

Compile-Time Polymorphism: The compiler determines exactly which method to execute at compile-time by matching the arguments you pass, ensuring optimal execution speed without runtime performance penalties.

## Example 

Consider a scenario where you want to log messages. Instead of creating different method names like logMessage, logError, and logWarning, you can overload a single log method:

public void log(String message) { ... }
public void log(String message, int errorCode) { ... }
public void log(String message, Exception e) { ... }


# Encapsulation

## what is encapsulation ?
Encapsulation in Java is an object-oriented principle that binds data and methods into a single unit, typically a class. It restricts direct access to data by hiding implementation details. This ensures controlled interaction with the data through defined methods.

## Why there is a need of encapsulation ?

Protection: It stops outside code from accidentally breaking or ruining your data.

Control: You decide exactly how people can see or change your data.

Rules (Validation): You can set strict rules. 

Read-Only or Write-Only Access: You can make a class read-only (by omitting setters) or write-only (by omitting getters).

For example, you can block someone from setting a user's age to -5 or a bank balance to a negative number.Easy to Change: If you want to change how your code works inside the class, you can do it without breaking anyone else's code outside.


## Example

public class BankAccount {
    // 1. Private variables hide the data from direct external access
    private String accountNumber;
    private double balance;

    // Constructor
    public BankAccount(String accountNumber, double initialBalance) {
        this.accountNumber = accountNumber;
        if (initialBalance >= 0) {
            this.balance = initialBalance;
        }
    }

    // 2. Getter method provides controlled read access
    public double getBalance() {
        return this.balance;
    }

    // 3. Setter method provides controlled write access with validation
    public void deposit(double amount) {
        if (amount > 0) { // Validation logic
            this.balance += amount;
            System.out.println("Successfully deposited: $" + amount);
        } else {
            System.out.println("Invalid deposit amount!");
        }
    }
}


# Constructors

## What is constructor?

A constructor in Java is a special member that is called when an object is created. It initializes the new object’s state. It is used to set default or user-defined values for the object's attributes

A constructor has the same name as the class.
It does not have a return type, not even void.
It can accept parameters to initialize object properties.

## why constructor uses same name as class? what is the reason behind it??

In Java the constructor name matches the class name because a constructor is a special member whose purpose is to produce and initialize instances of that particular class. Making the constructor name identical to the class name provides a simple, unambiguous syntactic signal to the compiler and to programmers that the method is not an ordinary method but the initializer for objects of that class.

## Example

public class Smartphone {
    private String brand;
    private String color;

    // This is the CONSTRUCTOR
    // Rules: It has the EXACT SAME NAME as the class, and NO return type (like void).
    public Smartphone(String startingBrand, String startingColor) {
        brand = startingBrand;
        color = startingColor;
        System.out.println("A new " + color + " " + brand + " phone was just built!");
    }
}



public class Main {
    public static void main(String[] args) {
        // Creating objects calls the constructor automatically
        Smartphone phone1 = new Smartphone("Apple", "Silver");
        Smartphone phone2 = new Smartphone("Samsung", "Black");
    }
}


A new Silver Apple phone was just built!
A new Black Samsung phone was just built!

# Static keyword

## what is static keyword?

static means "belongs to the class itself, not to individual objects."Normally, when you create a class, each object you make gets its own separate copy of data. But if you mark a variable or method as static, it becomes a shared resource. There is only one single copy of it in the entire program, and all objects share it.

## Why Do We Need It?

To Share Data: If you need a variable that tracks information across all objects (like a counter), static keeps them all synchronized.

To Save Memory: Instead of 1,000 objects creating 1,000 copies of the same variable, a static variable takes up memory only once.

For Utility Functions: If a method just does a calculation and doesn't depend on an object's specific data (like Math.sqrt()), making it static lets you use it instantly without creating an object first.

## Example

public class Student {
    // 1. Normal Variable: Every student gets their own unique name
    public String name;

    // 2. Static Variable: Shared by ALL students. Only 1 copy exists.
    public static String schoolName = "Hogwarts";

    // Constructor
    public Student(String studentName) {
        this.name = studentName;
    }
}

public class Main {
    public static void main(String[] args) {
        // You can access static variables directly through the class
        System.out.println(Student.schoolName); // Prints: Hogwarts

        // Creating two different student objects
        Student s1 = new Student("Harry");
        Student s2 = new Student("Ron");

        // Both objects share the same static variable
        System.out.println(s1.name + " goes to " + s1.schoolName); // Harry goes to Hogwarts
        System.out.println(s2.name + " goes to " + s2.schoolName); // Ron goes to Hogwarts

        // 3. Changing it once changes it for everyone!
        Student.schoolName = "Magic Academy";

        System.out.println(s1.schoolName); // Prints: Magic Academy
        System.out.println(s2.schoolName); // Prints: Magic Academy
    }
}


Quick Rules to RememberStatic Methods: A static method (like public static void main) can be called without creating an object. 
Because of this, static methods cannot use non-static variables.Accessing: Always access static items using the Class name (ClassName.variable) rather than the object name. It keeps your code clear.

The Golden Rule: Static methods can only access other static variables or static methods. They cannot look at non-static variables because they don't belong to any specific object!


## Polymorphism = same method, different behavior. Example: shape.draw() behaves differently for Circle and Rectangle. Two types — overloading (same method name, different params) and overriding (child changes parent method).

## Abstraction = hiding complex implementation, showing only what's necessary. Example — you call list.add() without knowing how it works internally.

Pillar          One Line
Encapsulation   Hide data using private + getters/setters
Inheritance     Child class reuses parent class methods
Polymorphism    Same method, different behavior
Abstraction     Hide complexity, show only what's needed