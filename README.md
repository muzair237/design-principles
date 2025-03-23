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

## 1) SOLID Principles

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

1. This class handles multiple responsibilities (user registration, email notifications, and report generation).
2. If we modify the reporting logic, we might introduce bugs that affect user registration because both functionalities exist in the same class. A failure in generating reports could potentially crash or disrupt the entire UserService, impacting user-related operations.

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

1. Each class has a single responsibility, making changes more manageable.
2. Loosely coupled components that can be modified independently.

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

1. Every time we add a new payment method (e.g., Crypto), we have to modify the `PaymentService` class.
2. This violates OCP since changes are required for every new feature.

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

1. We can add new payment methods without modifying `PaymentService`.
2. The system is easier to extend and maintain.

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

1. `CreditCardPayment` works fine since credit cards allow immediate deductions.
2. But `PostpaidPayment` violates LSP because calling `deductAmount()` throws an error instead of behaving like a `PaymentMethod`.
3. If we substitute `PaymentMethod` with `PostpaidPayment`, the program breaks unexpectedly.

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

1. Now, `CreditCard` and MonthlyBilling follow their respective behaviors.
2. We no longer force postpaid payments to implement `deductAmount()`.

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

1. `BasicUserReportService` implements methods it should not have which is `viewReport`.
2. Unnecessary methods force developers to handle errors for invalid actions.

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

1. `BasicUserReportService` implements only what it needs.
2. No unnecessary empty or error-throwing methods.
3. The codebase is cleaner, easier to maintain, and more flexible.

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

1. `UserService` is hardcoded to use `FileLogger`.
2. Switching to a database logger or cloud logging requires modifying `UserService`.
3. Testing is difficult because logs are directly written to a file.

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

1. `UserService` does not depend on a concrete logger.
2. We can switch logging implementations easily.
3. Unit testing is simpler by injecting a mock logger.

---

## 2) DRY (Don't Repeat Yourself) Principle

Every piece of knowledge must have a single, unambiguous, and authoritative representation within a system. In simpler terms:

- **Avoid duplication** of logic, data, and code.
- **Centralize reusable components**.
- If a concept changes, it should **only require modification in one place**.

By following **DRY**, we improve:

- **Maintainability** – Changes in one place reflect everywhere.
- **Scalability** – Code is modular and easier to extend.
- **Readability** – Less clutter and reduced redundancy.

❌ **Bad Example: Repeating Validation Logic in Multiple Places**

```ts
class UserController {
  registerUser(email: string, password: string): void {
    if (!email.includes("@")) {
      throw new Error("Invalid email format");
    }
    if (password.length < 6) {
      throw new Error("Password must be at least 6 characters long");
    }
    console.log("User registered");
  }

  updateUser(email: string, password: string): void {
    if (!email.includes("@")) {
      throw new Error("Invalid email format");
    }
    if (password.length < 6) {
      throw new Error("Password must be at least 6 characters long");
    }
    console.log("User updated");
  }
}
```

🔴 **Issues:** :

1. The same validation logic is repeated in both methods.
2. If the validation rules change (e.g., passwords must be 8+ characters), we need to modify both places.
3. High risk of inconsistency if one method is updated but the other isn’t.

✅ **Better Approach: Extract Validation into a Separate Function:** :

```ts
class UserValidator {
  static validate(email: string, password: string): void {
    if (!email.includes("@")) {
      throw new Error("Invalid email format");
    }
    if (password.length < 6) {
      throw new Error("Password must be at least 6 characters long");
    }
  }
}

class UserController {
  registerUser(email: string, password: string): void {
    UserValidator.validate(email, password);
    console.log("User registered");
  }

  updateUser(email: string, password: string): void {
    UserValidator.validate(email, password);
    console.log("User updated");
  }
}
```

✅ **Benifits:** :

1. Validation is now centralized in `UserValidator`.
2. Future changes (e.g., stricter password rules) are made in ONE place.
3. Code is easier to read and maintain.

---

## 3) YAGNI (You Ain’t Gonna Need It) Principle

You should not add functionality until it is necessary. In simpler terms:

- **Don’t write code for features you think you might need in the future**.
- **Build only what is required right now**.
- **Avoid over-engineering** by adding unnecessary complexity.

By following **YAGNI**, we improve:

- **Simplicity** – Less code to manage and maintain.
- **Faster Development** – Focus on what matters.
- **Easier Refactoring** – No unnecessary complexity.

❌ **Bad Example: Creating Role-Based Access Before It’s Needed**

```ts
class User {
  constructor(public name: string, public role: "author" | "admin") {}
}

class BlogService {
  createPost(user: User, content: string) {
    if (user.role === "admin") {
      console.log("Admins can create posts!");
    } else if (user.role === "author") {
      console.log("Authors can create posts!");
    }
  }
}
```

🔴 **Issues:** :

1. The `admin` role isn’t needed right now.
2. Unnecessary role-checking logic adds complexity.
3. Future-proofing too early – requirements might change.

✅ **Better Approach: Implement Only What’s Required:** :

```ts
class User {
  constructor(public name: string) {}
}

class BlogService {
  createPost(user: User, content: string) {
    console.log(`${user.name} created a post!`);
  }
}
```

✅ **Benifits:** :

1. Only supports the existing `author` role.
2. Less code, easier to modify later.
3. When admins are needed, we can refactor properly.

---

## 4) KISS (Keep It Simple, Stupid) Principle

Most systems work best if they are kept simple rather than made complex. Therefore, simplicity should be a key goal in design, and unnecessary complexity should be avoided. In simpler terms:

- **Write straightforward, clear, and easy-to-understand code**.
- **Avoid unnecessary abstractions, over-engineering, and convoluted logic**.
- **Prioritize readability and maintainability**.

By following **DRY**, we improve:

- **Easier debugging and maintenance** – Simple code is easier to fix.
- **Faster development** – No wasted time on unnecessary complexity.
- **Better collaboration** – Other developers can understand your code easily.

❌ **Bad Example: Using a Complex Regular Expression for a Simple Validation**

```ts
function isNumeric(str: string): boolean {
  return /^[0-9]+$/.test(str);
}
```

🔴 **Issues:** :

1. Overuse of regex for something simple.
2. Harder to read for someone unfamiliar with regex.

✅ **Better Approach: Use a Straightforward Approach:** :

```ts
function isNumeric(str: string): boolean {
  return !isNaN(Number(str));
}
```

✅ **Benifits:** :

1. Simple and intuitive.
2. Easier to understand.
3. No unnecessary regex complexity.

---

## 5) Law of Demeter (LoD) – "Principle of Least Knowledge"

An object should only talk to its immediate friends and not to strangers. In other words a class should only interact with:

- Itself (**its own methods and properties**).
- Its **direct dependencies** (objects it directly owns).
- Objects passed **explicitly** to it as parameters.

A class **should NOT:**

- Depend on the internals of other objects.
- Call methods on objects returned by other methods (**"train wreck" or "method chaining" violation**).

Why Follow **LoD**?

- **Increases Encapsulation** – Objects don’t depend on internal details of others.
- **Improves Maintainability** – Fewer dependencies mean easier refactoring.
- **Reduces Coupling** – Changes in one class don’t break unrelated parts of the system.

❌ **Bad Example: Exposing Internal Data Structures**

```ts
class User {
  private email: string;

  constructor(email: string) {
    this.email = email;
  }

  getEmail(): string {
    return this.email;
  }
}

class UserService {
  private user: User;

  constructor(user: User) {
    this.user = user;
  }

  getUser(): User {
    return this.user;
  }
}

const userData = new User("dynamic@example.com");
const userService = new UserService(userData);

console.log(userService.getUser().getEmail());
```

🔴 **Issues:** :.

1. The caller (console.log) is directly fetching the User object and then calling `getEmail()`.
1. The external code knows too much about `User` internal structure.
1. If `User` changes, every caller needs modification.

✅ **Better Approach: Return Only What’s Needed:** :

```ts
class User {
  private email: string;

  constructor(email: string) {
    this.email = email;
  }

  getEmail(): string {
    return this.email;
  }
}

class UserService {
  private user: User;

  constructor(user: User) {
    this.user = user;
  }

  getUserEmail(): string {
    return this.user.getEmail();
  }
}

const userData = new User("dynamic@example.com");
const userService = new UserService(userData);
console.log(userService.getUserEmail());

```

✅ **Benifits:** :

1. Encapsulation of `User` internals.
2. Only required data is exposed.
3. Less dependency on internal structures.

---

## 6) Composition Over Inheritance Principle

**Composition over Inheritance** is a design principle that suggests **favoring composition (object delegation) over class inheritance (extending classes)** to achieve code reuse and flexibility. In simpler terms, instead of building a deep **class hierarchy** where behavior is inherited from parent classes, we **compose objects** by combining small, reusable behaviors.

Why Prefer Composition Over Inheritance?

- **Encapsulation** – Implementation details are hidden inside components.
- **Flexibility** – Components can be changed at runtime, unlike fixed inheritance.
- **Loose Coupling** – Objects depend only on required behaviors, not full parent classes.
- **Avoids the Fragile Base Class Problem** – A change in a parent class won’t break many child classes.
- **Better Code Reusability** – Components can be shared across different objects.

❌ **Bad Example: The Fragile Base Class Problem**

```ts
class Character {
  move() {
    console.log("Moving...");
  }
}

class Player extends Character {
  attack() {
    console.log("Player Attacks!");
  }
}

class Enemy extends Character {
  attack() {
    console.log("Enemy Attacks!");
  }
}
```

🔴 **Issues:** :

1. What if we now introduce Flying Enemies or Swimming Players?
2. We might need multiple levels of inheritance, leading to rigid, hard-to-change code.
3. If we add new behaviors like Stealth Mode, we either:
   - Modify the base class (risking breaking existing classes).
   - Create multiple subclasses (messy and redundant).

✅ **Better Approach: Use Composition Instead:** :

```ts
// Independent Behaviors
class MoveBehavior {
  move() {
    console.log("Moving...");
  }
}

class AttackBehavior {
  attack() {
    console.log("Attacking...");
  }
}

// Character now "has-a" behavior instead of "is-a"
class Character {
  constructor(
    private moveBehavior: MoveBehavior,
    private attackBehavior: AttackBehavior
  ) {}

  move() {
    this.moveBehavior.move();
  }

  attack() {
    this.attackBehavior.attack();
  }
}

const player = new Character(new MoveBehavior(), new AttackBehavior());
const enemy = new Character(new MoveBehavior(), new AttackBehavior());

player.attack();
enemy.move();
```

✅ **Benifits:** :

1. No deep inheritance hierarchy.
2. Easier to modify behaviors (e.g., replace `AttackBehavior` for a `StealthBehavior`).
3. We can attach different behaviors to different objects.

---

## 7) Encapsulate What Varies Principle

"Encapsulate What Varies" is a **core principle of software design** that suggests:

- Identify the **changing parts** of a system.
- Isolate them into **separate modules, classes, or strategies**.
- Keep the **stable parts independent** of the variations.

By doing this, we make the system **more flexible, maintainable, and extendable** without modifying existing code.

❌ **Bad Example: Hardcoding the Variations**

```ts
class NotificationService {
  sendNotification(message: string, type: string) {
    if (type === "email") {
      console.log(`Sending EMAIL: ${message}`);
    } else if (type === "sms") {
      console.log(`Sending SMS: ${message}`);
    } else if (type === "push") {
      console.log(`Sending PUSH NOTIFICATION: ${message}`);
    }
  }
}

const notifier = new NotificationService();
notifier.sendNotification("Order Shipped!", "email");
```

🔴 **Issues:** :

1. Violation of **Open-Closed Principle (OCP)** – Adding a new notification method (e.g., WhatsApp) requires modifying the existing class.
2. The class mixes notification logic, making changes risky.
3. If one method changes, it may break others.

✅ **Better Approach: Encapsulate What Varies (Strategy Pattern):** :

```ts
interface NotificationStrategy {
  send(message: string): void;
}

class EmailNotification implements NotificationStrategy {
  send(message: string) {
    console.log(`📧 Sending EMAIL: ${message}`);
  }
}

class SMSNotification implements NotificationStrategy {
  send(message: string) {
    console.log(`📩 Sending SMS: ${message}`);
  }
}

class PushNotification implements NotificationStrategy {
  send(message: string) {
    console.log(`🔔 Sending PUSH NOTIFICATION: ${message}`);
  }
}

class NotificationService {
  constructor(private strategy: NotificationStrategy) {}

  notify(message: string) {
    this.strategy.send(message);
  }
}

const emailNotifier = new NotificationService(new EmailNotification());
emailNotifier.notify("Your order has been shipped!");

const smsNotifier = new NotificationService(new SMSNotification());
smsNotifier.notify("Your order has been delivered!");

const pushNotifier = new NotificationService(new PushNotification());
pushNotifier.notify("Flash sale starts now!");
```

✅ **Benifits:** :

1. Each notification type is independent and can be modified without affecting others.
2. You can add a new notification method (e.g., WhatsApp) without modifying existing classes.
3. The code is cleaner, modular, and scalable.

---

## 8) Hollywood Principle

The **Hollywood Principle** is a **software design guideline** that states: "Don't call us, we'll call you."

It means that **lower-level** components should not call **higher-level** components directly.  
Instead, the **higher-level** components **control the flow** and decide **when** and **how** the lower-level components are used.

This principle **reduces coupling** and makes the system **more flexible and maintainable**.

Why do we need the **Hollywood Principle**?

- **Prevents Tight Coupling** – Lower-level components should not depend directly on higher-level components.
- **Promotes Inversion of Control (IoC)** – Higher-level modules dictate the behavior, allowing more flexibility.
- **Improves Scalability** – New features can be added without modifying existing classes.
- **Enhances Maintainability** – Changes in one module do not impact others significantly.

❌ **Bad Example: Without Hollywood Principle (Tightly Coupled) and every observer manually asking for updates:**

```ts
class NewsAgency {
  getNews() {
    return "Breaking News: Observer Pattern Explained!";
  }
}

class Subscriber {
  constructor(private agency: NewsAgency) {}

  checkNews() {
    console.log("Subscriber checking news:", this.agency.getNews());
  }
}

const agency = new NewsAgency();
const user1 = new Subscriber(agency);
const user2 = new Subscriber(agency);

user1.checkNews();
user2.checkNews();

```

🔴 **Issues:** :

1. Subscribers actively call NewsAgency instead of waiting for updates.
2. If nothing changes, subscribers still keep asking.
3. Subscribers directly depend on the news agency.

✅ **Better Approach: With Hollywood Principle (Loosely Coupled) and news agency decided when to notify:** :

```ts
class NewsAgency {
  private subscribers: Subscriber[] = [];

  subscribe(subscriber: Subscriber) {
    this.subscribers.push(subscriber);
  }

  publishNews(news: string) {
    console.log("NewsAgency: Publishing news...");
    this.subscribers.forEach(sub => sub.update(news));
  }
}

class Subscriber {
  constructor(private name: string) {}

  update(news: string) {
    console.log(`${this.name} received news: ${news}`);
  }
}

const agency = new NewsAgency();
const user1 = new Subscriber("User1");
const user2 = new Subscriber("User2");

agency.subscribe(user1);
agency.subscribe(user2);

agency.publishNews("Breaking News: Observer Pattern Explained!");
```

✅ **Benifits:** :

1. Subscribers register, but the agency decides when to send updates.
2. Subscribers don’t need to constantly check for updates.
3. Only updates when new information is available.
4. This is how news apps, event listeners, and pub-sub systems work.

---

## 9) Program Against Abstractions Principle

"Program Against Abstractions" encourages **coding against abstract interfaces rather than concrete implementations**.
This principle aligns closely with **Dependency Inversion Principle (DIP)** and **Open-Closed Principle (OCP)** from **SOLID**.  
It helps in building **loosely coupled, flexible, and maintainable software**.

Why Do We Need to **Program Against Abstractions**?
- **Flexibility & Extensibility** – Easily swap implementations without modifying existing code.  
- **Loose Coupling** – Components depend on abstractions, not concrete classes.  
- **Scalability** – Supports future changes with minimal effort.  
- **Testability** – Makes it easier to mock dependencies in unit tests.

❌ **Bad Example: Directly Programming Against a Concrete Class**

```ts
class PayPalPayment {
  processPayment(amount: number) {
    console.log(`💰 Processing $${amount} via PayPal`);
  }
}

class Checkout {
  private paymentGateway: PayPalPayment;

  constructor() {
    this.paymentGateway = new PayPalPayment();
  }

  completeOrder(amount: number) {
    this.paymentGateway.processPayment(amount);
  }
}

const checkout = new Checkout();
checkout.completeOrder(100);
```

🔴 **Issues:** :

1. Changing from PayPal to Stripe requires modifying the Checkout class.
2. Can't extend functionality without modifying existing code.
3. Hard to mock the payment processor in unit tests.

✅ **Better Approach: Programming Against an Abstraction:** :

```ts
interface PaymentGateway {
  processPayment(amount: number): void;
}

class PayPalPayment implements PaymentGateway {
  processPayment(amount: number) {
    console.log(`💰 Processing $${amount} via PayPal`);
  }
}

class StripePayment implements PaymentGateway {
  processPayment(amount: number) {
    console.log(`💳 Processing $${amount} via Stripe`);
  }
}

class Checkout {
  private paymentGateway: PaymentGateway;

  constructor(paymentGateway: PaymentGateway) {
    this.paymentGateway = paymentGateway;
  }

  completeOrder(amount: number) {
    this.paymentGateway.processPayment(amount);
  }
}

const paypalCheckout = new Checkout(new PayPalPayment());
paypalCheckout.completeOrder(100);

const stripeCheckout = new Checkout(new StripePayment());
stripeCheckout.completeOrder(150);
```

✅ **Benifits:** :

1. New payment methods can be added without modifying existing code.
2. High-level modules (Checkout) depend on abstractions (PaymentGateway), not concrete implementations.
3. Can switch to a different payment provider without modifying the Checkout class.
4. We can mock the PaymentGateway interface in tests.

---
