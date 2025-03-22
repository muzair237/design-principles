# Software Design Principles

## What Are Software Design Principles?

Software Design Principles are **guidelines** that help developers create maintainable, scalable, and robust software. These principles **do not enforce specific implementations** but provide a **conceptual framework** to make better design decisions.

Instead of focusing on "how" something should be coded, design principles focus on "why" and "what" makes the design **good** or **bad**.

## Why Do We Need Design Principles?

Without well-defined principles, software tends to become:

- **Hard to maintain** → Small changes break multiple parts of the system.
- **Difficult to scale** → New features introduce unintended side effects.
- **Bug-prone** → Poor structure leads to hard-to-debug issues.
- **Inflexible** → Difficult to extend without modifying existing code.

Applying design principles **reduces technical debt** and **improves code longevity**.

---

## SOLID Principles

SOLID is an acronym for five key **object-oriented design principles** that help developers create **scalable, maintainable, and robust** software. These principles were introduced by **Robert C. Martin (Uncle Bob)** and are fundamental for writing **clean and flexible** code.

SOLID stands for:

1. **S** - Single Responsibility Principle (SRP)
2. **O** - Open/Closed Principle (OCP)
3. **L** - Liskov Substitution Principle (LSP)
4. **I** - Interface Segregation Principle (ISP)
5. **D** - Dependency Inversion Principle (DIP)

These principles prevent common software design issues such as **tight coupling, rigidity, fragility, and unnecessary complexity**.

---

### 1️⃣ Single Responsibility Principle (SRP)

**Definition:**  
A class should have **only one reason to change**. In simpler terms it should have **only one responsibility**.

❌ **Bad Example: User Management in an E-Commerce System**

```ts
class UserService {
  registerUser(email: string, password: string) {
    // Register user in the database
  }

  sendWelcomeEmail(email: string) {
    // Send a welcome email to the user
  }

  generateReport(userId: string) {
    // Generate user activity report
  }
}
```

🔴 **Issues:** :
1) This class handles multiple responsibilities (user registration, email notifications, and report generation).
2) If we modify the reporting logic, we might introduce bugs that affect user registration because both functionalities exist in the same class. A failure in generating reports could potentially crash or disrupt the entire UserService, impacting user-related operations.

✅ **Better Approach: Separate Responsibilities into Different Classes:** :

```ts
class UserRepository {
  registerUser(email: string, password: string) {
    // Handles database operations
  }
}

class EmailService {
  sendWelcomeEmail(email: string) {
    // Handles email notifications
  }
}

class ReportService {
  generateReport(userId: string) {
    // Handles user reports
  }
}
```

✅ **Benifits:** :
1) Each class has a single responsibility, making changes more manageable.
2) Loosely coupled components that can be modified independently.

### 2️⃣ Open/Closed Principle (OCP)

**Definition:**  
Software components should be **open for extension** but **closed for modification**. In simple terms, A module, class, or function should allow **new functionality** to be added **without altering existing code**

❌ **Bad Example: User Management in an E-Commerce System**

```ts
class PaymentService {
  processPayment(type: string, amount: number) {
    if (type === "creditCard") {
      // Process credit card payment
    } else if (type === "paypal") {
      // Process PayPal payment
    }
  }
}
```

🔴 **Issues:** :
1) Every time we add a new payment method (e.g., Crypto), we have to modify the ```PaymentService``` class.
2) This violates OCP since changes are required for every new feature.

✅ **Better Approach: Separate Responsibilities into Different Classes:** :

```ts
interface PaymentProcessor {
  process(amount: number): void;
}

class CreditCardPayment implements PaymentProcessor {
  process(amount: number) {
    // Process credit card payment
  }
}

class PayPalPayment implements PaymentProcessor {
  process(amount: number) {
    // Process PayPal payment
  }
}

class PaymentService {
  processPayment(paymentMethod: PaymentProcessor, amount: number) {
    paymentMethod.process(amount);
  }
}
```

✅ **Benifits:** :
1) We can add new payment methods without modifying ```PaymentService```.
2) The system is easier to extend and maintain.

### 3️⃣ Liskov Substitution Principle (LSP)

**Definition:**  
Objects of a subclass should be replaceable for objects of the superclass **without breaking the application**. In simpler terms, a subclass **should behave in a way that does not break expectations** set by its parent class.

❌ **Bad Example: A Broken Payment System**

```ts
class PaymentMethod {
  deductAmount(amount: number) {
    console.log(`$${amount} deducted from your account.`);
  }
}

class CreditCardPayment extends PaymentMethod {
  deductAmount(amount: number) {
    console.log(`$${amount} charged to your credit card.`);
  }
}

class PostpaidPayment extends PaymentMethod {
  deductAmount(amount: number) {
    throw new Error("Postpaid payment cannot deduct money immediately!");
  }
}
```

🔴 **Issues:** :
1) ```CreditCardPayment``` works fine since credit cards allow immediate deductions.
2) But ```PostpaidPayment``` violates LSP because calling ```deductAmount()``` throws an error instead of behaving like a ```PaymentMethod```.
3) If we substitute ```PaymentMethod``` with ```PostpaidPayment```, the program breaks unexpectedly.

✅ **Better Approach: Refactor with a More Suitable Abstraction:** :

```ts
interface PrepaidPayment {
  deductAmount(amount: number): void;
}

interface PostpaidPayment {
  billAmount(amount: number): void;
}

class CreditCard implements PrepaidPayment {
  deductAmount(amount: number) {
    console.log(`$${amount} charged to your credit card.`);
  }
}

class MonthlyBilling implements PostpaidPayment {
  billAmount(amount: number) {
    console.log(`$${amount} added to your monthly bill.`);
  }
}
```

✅ **Benifits:** :
1) Now, ```CreditCard``` and MonthlyBilling follow their respective behaviors.
2) We no longer force postpaid payments to implement ```deductAmount()```.

### 4️⃣ Interface Segregation Principle (ISP)

**Definition:**  
A class should **not be forced to depend on interfaces they do not use**. In simpler terms, **large, bloated interfaces should be broken down** into **smaller, specific interfaces** so that **implementing classes only need to define the methods they actually use**.


❌ **Bad Example: A Single Interface for All User Actions**

```ts
interface ReportService {
  createReport(): void;
  editReport(): void;
  deleteReport(): void;
  viewReport(): void;
}

class AdminReportService implements ReportService {
  createReport() {
    console.log("Admin created a report.");
  }
  editReport() {
    console.log("Admin edited a report.");
  }
  deleteReport() {
    console.log("Admin deleted a report.");
  }
  viewReport() {
    console.log("Admin viewed a report.");
  }
}

class BasicUserReportService implements ReportService {
  createReport() {
    throw new Error("Basic users cannot create reports!");
  }
  editReport() {
    throw new Error("Basic users cannot edit reports!");
  }
  deleteReport() {
    throw new Error("Basic users cannot delete reports!");
  }
  viewReport() {
    console.log("Basic user viewed a report.");
  }
}
```

🔴 **Issues:** :
1) ```BasicUserReportService``` implements methods it should not have which is ```viewReport```.
2) Unnecessary methods force developers to handle errors for invalid actions.

✅ **Better Approach: Split into Smaller Interfaces:** :

```ts
interface ReportViewer {
  viewReport(): void;
}

interface ReportEditor {
  createReport(): void;
  editReport(): void;
  deleteReport(): void;
}

class AdminReportService implements ReportViewer, ReportEditor {
  createReport() {
    console.log("Admin created a report.");
  }
  editReport() {
    console.log("Admin edited a report.");
  }
  deleteReport() {
    console.log("Admin deleted a report.");
  }
  viewReport() {
    console.log("Admin viewed a report.");
  }
}

class BasicUserReportService implements ReportViewer {
  viewReport() {
    console.log("Basic user viewed a report.");
  }
}
```

✅ **Benifits:** :
1) ```BasicUserReportService``` implements only what it needs.
2) No unnecessary empty or error-throwing methods.
3) The codebase is cleaner, easier to maintain, and more flexible.

### 5️⃣ Dependency Inversion Principle (DIP)

**Definition:**  
High-level modules should **not depend on low-level modules**. Both should depend on **abstractions**. Accordingly, abstractions should not depend on details. Details should depend on abstractions. 
In simpler terms:  
- **High-level modules** (business logic) should not be tightly coupled to **low-level modules** (database, APIs, libraries).  
- Both should depend on **interfaces (abstractions)** instead of **concrete implementations**.  
- This allows **flexibility, maintainability, and testability**.

❌ **Bad Example: Logging System in a Backend Application**

```ts
class FileLogger {
  log(message: string): void {
    console.log(`Writing log to file: ${message}`);
  }
}

class UserService {
  private logger: FileLogger;

  constructor() {
    this.logger = new FileLogger();
  }

  createUser(name: string): void {
    this.logger.log(`User ${name} created.`);
  }
}
```

🔴 **Issues:** :
1) ```UserService``` is hardcoded to use ```FileLogger```.
2) Switching to a database logger or cloud logging requires modifying ```UserService```.
3) Testing is difficult because logs are directly written to a file.

✅ **Better Approach: Introduce an Abstraction (Interface):** :

```ts
interface Logger {
  log(message: string): void;
}

class FileLogger implements Logger {
  log(message: string): void {
    console.log(`Writing log to file: ${message}`);
  }
}

class CloudLogger implements Logger {
  log(message: string): void {
    console.log(`Sending log to cloud: ${message}`);
  }
}

class UserService {
  private logger: Logger;

  constructor(logger: Logger) {
    this.logger = logger;
  }

  createUser(name: string): void {
    this.logger.log(`User ${name} created.`);
  }
}
```

✅ **Benifits:** :
1) ```UserService``` does not depend on a concrete logger.
2) We can switch logging implementations easily.
3) Unit testing is simpler by injecting a mock logger.


