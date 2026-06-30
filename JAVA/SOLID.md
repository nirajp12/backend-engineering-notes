# S : Single responsibility principle (One class should have only one reason to change.)

S - Stands for Single responsibility principle  that means a class should have only 1 reason to change for e.g if there is a class called tenant service in that if i define all the operation for tenant we have to perform like create tenant, send mail, save tenant etc. so a class is doing lot of things lets say if i want to  change something in notifiaction then i have to change in tenant service rather we can have seperate  classes for different operations.

Starting With S — Single Responsibility Principle

One class should have only ONE reason to change.

Wrong way — your project:
javapublic class TenantService {
    public void createTenant() { }     // business logic
    public void sendEmail() { }        // notification logic
    public void saveToDatabase() { }   // database logic
    public void generateReport() { }   // reporting logic
}
This class does 4 different things. If email logic changes — you touch TenantService. If DB logic changes — you touch TenantService. Too many reasons to change.
Right way:
javapublic class TenantService { 
    public void createTenant() { } // ONLY business logic
}
public class NotificationService { 
    public void sendEmail() { }    // ONLY notifications
}
public class TenantRepository { 
    public void save() { }         // ONLY database
}

# O : Open/Closed Principle

Class should be OPEN for extension but CLOSED for modification.

Meaning — when new requirement comes, you should add new code, not change existing working code.

"Instead of modifying PaymentService every time a new gateway comes — I create a PaymentGateway interface. Each gateway like Razorpay, Stripe implements it separately. When a new gateway like PayPal comes — I just add a new class implementing the interface. Existing code is untouched."

Wrong way:
javapublic class PaymentService {
    public void processPayment(String type) {
        if(type.equals("RAZORPAY")) {
            // razorpay logic
        } else if(type.equals("STRIPE")) {
            // stripe logic
        }
        // every new gateway = modify this class ❌
    }
}
Every time you add a new payment gateway — you touch and risk breaking existing code.
Right way:
javapublic interface PaymentGateway {
    void processPayment(double amount);
}

public class RazorpayGateway implements PaymentGateway {
    public void processPayment(double amount) { }
}

public class StripeGateway implements PaymentGateway {
    public void processPayment(double amount) { }
}

// New gateway? Just ADD a new class. Don't touch existing code ✅
public class PaypalGateway implements PaymentGateway {
    public void processPayment(double amount) { }
}


# L : Liskov Substitution Principle

Liskov Substitution Principle states that a child class should be replaceable by its parent class without breaking the program.
In my multi-tenant platform I have a User class with a login() method. TenantAdmin extends User and overrides login() with additional admin checks — but it still logs in successfully. So anywhere I use User, I can replace it with TenantAdmin and program works fine. That's LSP followed.
LSP is violated when a child class breaks the parent's promise. For example if I create a SuspendedUser extending User but throw an exception inside login() — now my processLogin(User user) method crashes when it receives a SuspendedUser. Child broke the parent's promise — that's LSP violation.
The fix is to handle suspension logic inside the User class itself instead of creating a broken child class."

"The fix is to handle suspension logic inside the User class itself — check if account is suspended inside login() — rather than creating a child class that breaks the parent's promise."

magine you're building your multi-tenant platform. You have a User class with a login() method.
javapublic class User {
    public void login() {
        System.out.println("User logged in");
    }
}
Now you have two types of users:

TenantAdmin — can login normally
SuspendedUser — account is suspended, cannot login


Developer Makes This Mistake
javapublic class SuspendedUser extends User {
    public void login() {
        throw new RuntimeException("Account suspended"); // ❌
    }
}
Now somewhere in your code:
javapublic void processLogin(User user) {
    user.login(); // works fine for normal User
                  // CRASHES for SuspendedUser ❌
}
You passed a child class — program broke. That's LSP violation.

Why This Is A Problem
Your processLogin() method trusts that ANY User can login. That's the contract User class promises.
SuspendedUser breaks that promise.

The Fix
Don't use inheritance here. Model it differently:
javapublic class User {
    private boolean isSuspended;
    
    public void login() {
        if(isSuspended) {
            System.out.println("Account suspended");
            return;
        }
        System.out.println("Logged in successfully");
    }
}
Now suspension is handled inside the class — no broken promises.

Real World Analogy
Think of electrical sockets in your office.

Every socket promises — "plug in any device, it will get power."

Now imagine one socket that looks the same but gives electric shock instead of power.
It's a socket (child) but it broke the promise of what a socket does (parent contract). That's LSP violation.

One Line To Remember Forever

"Child class must KEEP the promises made by the parent class. Never break them."

# I : Interface Segregation principle

A class should not be forced to implement methods it doesn't need.


Wrong way:
javapublic interface UserActions {
    void login();
    void manageTeants();
    void generateReports();
    void processPayments();
}

// Regular user forced to implement everything ❌
public class RegularUser implements UserActions {
    public void login() { }
    public void manageTenants() { 
        throw new Exception("Not allowed"); // ❌ doesn't need this
    }
    public void generateReports() {
        throw new Exception("Not allowed"); // ❌ doesn't need this
    }
}

Right way — split interfaces:
javapublic interface Loginable {
    void login();
}

public interface TenantManageable {
    void manageTenants();
}

public interface Reportable {
    void generateReports();
}

// Each class implements only what it needs ✅
public class RegularUser implements Loginable {
    public void login() { }
}

public class TenantAdmin implements Loginable, TenantManageable {
    public void login() { }
    public void manageTenants() { }
}

Simple Way To Remember

"Don't force classes to implement what they don't need. Split fat interfaces into smaller ones."


"In my multi-tenant platform, instead of one fat UserActions interface with login, manage tenants, generate reports — I split it into Loginable, TenantManageable, Reportable. RegularUser only implements Loginable. TenantAdmin implements Loginable and TenantManageable. Each class gets only what it needs."

# D : Dependency Inversion Control

ou're building your multi-tenant platform. Your TenantService needs to save data to a database.
Day 1 — Your company uses MySQL.
javapublic class TenantService {
    private MySQLRepository repository = new MySQLRepository(); // hardcoded ❌
    
    public void createTenant() {
        repository.save();
    }
}
This works fine.

Day 30 — Boss says "We're switching to MongoDB."
Now you have to:

Open TenantService
Delete MySQLRepository
Add MongoRepository
Test everything again
Pray nothing breaks

You changed a high level class because a low level class changed. That's the problem DIP solves.

Why Is This Bad?
Imagine this in your platform:

TenantService depends on MySQLRepository
UserService depends on MySQLRepository
PaymentService depends on MySQLRepository

Switch to MongoDB → change 10 classes. High risk of breaking things.

The Fix — Depend On Interface, Not Implementation
Step 1 — Create an interface (abstraction)
javapublic interface TenantRepository {
    void save(Tenant tenant);
    Tenant findById(Long id);
}
Step 2 — Both MySQL and Mongo implement it
javapublic class MySQLTenantRepository implements TenantRepository {
    public void save(Tenant tenant) { 
        // MySQL specific logic
    }
    public Tenant findById(Long id) { 
        // MySQL specific logic
    }
}

public class MongoTenantRepository implements TenantRepository {
    public void save(Tenant tenant) { 
        // MongoDB specific logic
    }
    public Tenant findById(Long id) { 
        // MongoDB specific logic
    }
}
Step 3 — TenantService depends on interface only
javapublic class TenantService {
    private TenantRepository repository; // interface, not implementation ✅
    
    public TenantService(TenantRepository repository) {
        this.repository = repository; // injected from outside
    }
    
    public void createTenant(Tenant tenant) {
        repository.save(tenant);
    }
}
Now switch MySQL to MongoDB:

TenantService — zero changes ✅
Just swap which implementation you inject


Real Life Analogy
Think of a power socket in your house.
Your socket doesn't care if you plug in:

A phone charger
A laptop charger
A fan

It just provides power through a standard interface (3 pin socket).

Socket = TenantService (high level)

3 pin standard = TenantRepository (interface/abstraction)

Phone charger, laptop charger = MySQLRepository, MongoRepository (implementations)

Change the device — socket doesn't change. That's DIP.

How Spring Boot Does This Automatically
Without Spring — you manually inject:
javaTenantRepository repo = new MySQLTenantRepository();
TenantService service = new TenantService(repo);
With Spring — @Autowired handles it:
java@Service
public class TenantService {
    
    @Autowired
    private TenantRepository repository; // Spring injects correct impl
}
Spring looks at which class implements TenantRepository and injects it automatically at runtime.
That's DIP + DI working together.

Full Picture
WITHOUT DIP:
TenantService → MySQLRepository (tightly coupled ❌)
Change DB = change TenantService

WITH DIP:
TenantService → TenantRepository (interface ✅)
     ↑
MySQLRepository OR MongoRepository
Change DB = just swap implementation, TenantService unchanged

One Line To Remember

"Depend on interfaces not concrete classes — so changing implementation never breaks your high level code."

repository.save(tenant);
It depends on which implementation Spring injected.

How Spring Decides
If you have only one implementation of TenantRepository:
java@Repository
public class MySQLTenantRepository implements TenantRepository {
    public void save(Tenant tenant) { }
}
Spring automatically injects MySQLTenantRepository. No confusion.

What If You Have Two Implementations?
java@Repository
public class MySQLTenantRepository implements TenantRepository { }

@Repository
public class MongoTenantRepository implements TenantRepository { }
Spring gets confused — which one to inject? It throws an error.
Fix — use @Primary or @Qualifier:
java// Option 1 — mark one as default
@Repository
@Primary
public class MySQLTenantRepository implements TenantRepository { }

// Option 2 — specify which one you want
@Autowired
@Qualifier("mySQLTenantRepository")
private TenantRepository repository;

Simple Way To Remember
repository.save(tenant)
     ↓
Spring checks → which class implements TenantRepository?
     ↓
Injects that class → calls that class's save() method

You write repository.save() — Spring decides whose save() gets called at runtime.


That's runtime polymorphism + DIP + DI all working together in one line of code.

# ###################################################################################

"I built a centralized API Gateway in Spring Boot. The main challenge was that each vendor had a different request format — one needed query params, another needed form-encoded body. Initially I thought of writing separate classes per vendor but that wasn't scalable.
So I designed a standardized 4-argument format — arg1 for query params, arg2 for headers, arg3 for body, arg4 for form-encoded. The gateway automatically translates this into whatever the vendor needs. This way adding a new vendor requires zero code changes in the gateway — just configuration. That's the Adapter Pattern and Open/Closed Principle in practice.
The gateway also authenticated tenants via API keys, logged every request and response for audit purposes, and deducted from the tenant's wallet on each successful API call for usage-based billing."



"Authentication means verifying who the user is — proving identity. Authorization means checking what that authenticated user is allowed to do — their permissions.
In my API Gateway, when a request comes with x-api-key, I first decrypt it to extract the tenant name — this confirms the tenant's identity, that's authentication. Then I check the subscription table to see if this tenant has access to this specific API — that's authorization. So authentication answers 'who are you', authorization answers 'what can you do'. A tenant could be authenticated with a valid key but still get a 403 if they're not subscribed to that particular API — that's authorization failing even though authentication passed."





