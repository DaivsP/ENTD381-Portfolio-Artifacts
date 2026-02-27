# Artifact C: Sanitized GitHub Code Snippets

## Narrative
Creating Object-Oriented Programs is the core of my technical expertise. I was initially trained in OOP principles during my early years as a developer, where I learned to model real-world entities into reusable code components. I quickly mastered the four pillars of OOP: encapsulation, inheritance, polymorphism, and abstraction.

In a recent enterprise project, I designed a transaction processing system that relied heavily on these principles. I utilized encapsulation to protect sensitive financial data within private class variables, exposing them only through strictly controlled getter and setter methods. This ensured that the state of our financial objects could not be maliciously or accidentally altered.

Inheritance and polymorphism were critical in keeping the codebase DRY (Don't Repeat Yourself). I created a base `Transaction` class that contained common properties like timestamps and transaction IDs. I then extended this class to create specific objects like `DepositTransaction` and `WithdrawalTransaction`. Through polymorphism, my systems could process lists of various transaction types uniformly, drastically reducing code complexity.

As a manager, I now enforce these OOP standards across my development teams. I conduct rigorous architectural reviews to ensure our Java applications are modular, reusable, and maintainable. Included below are sanitized code snippets from a repository I built, demonstrating the creation of class hierarchies, the implementation of interfaces, and the use of access modifiers to build a secure Object-Oriented Program.

---

## Technical Implementations of Object-Oriented Programming

### 1. Abstraction & Encapsulation

**Abstraction** is the process of hiding the complex implementation details and showing only the essential features of an object. **Encapsulation** binds the data (attributes) and the methods that manipulate that data into a single unit (class), while restricting unauthorized access using access modifiers like `private`.

In the following snippet, the abstract `Transaction` class provides a generalized blueprint for all financial transactions, hiding the inner complexity of the attributes from the outside world. The sensitive fields like `id`, `amount`, and `timestamp` are encapsulated as `private`, meaning they can only be accessed or modified through the controlled, public `getId()`, `getAmount()`, and `getTimestamp()` getter methods.

```java
import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.util.UUID;

// Abstraction: This abstract class defines the blueprint for all transactions
public abstract class Transaction {
    
    // Encapsulation: Sensitive data fields are carefully hidden (private)
    private final String id;
    private final BigDecimal amount;
    private final LocalDateTime timestamp;

    public Transaction(BigDecimal amount) {
        this.id = UUID.randomUUID().toString();
        this.amount = amount;
        this.timestamp = LocalDateTime.now();
    }

    // Encapsulation: Read-only access exposed via public getter methods
    public String getId() {
        return id;
    }

    public BigDecimal getAmount() {
        return amount;
    }

    public LocalDateTime getTimestamp() {
        return timestamp;
    }

    // Abstraction: Subclasses must provide their own specific implementation
    public abstract void executeTransaction();
}
```

### 2. Inheritance

**Inheritance** is an OOP mechanism where a new class (subclass) derives the properties and behaviors of an existing class (superclass). This promotes code reusability (keeping the code DRY).

Below, `DepositTransaction` and `WithdrawalTransaction` inherit the standard attributes and behaviors of the `Transaction` class (like calculating IDs and saving timestamps), while allowing us to define their specific internal logic. The subclasses only need to implement the abstract `executeTransaction()` method.

```java
import java.math.BigDecimal;

// Inheritance: DepositTransaction "is-a" Transaction
public class DepositTransaction extends Transaction {

    public DepositTransaction(BigDecimal amount) {
        super(amount); // Calls the constructor of the base Transaction class
    }

    @Override
    public void executeTransaction() {
        // Specific logic for depositing funds
        System.out.println("Processing Deposit of $" + this.getAmount() + 
                           " for Transaction ID " + this.getId());
        // E.g., Database call to credit account...
    }
}

// Inheritance: WithdrawalTransaction "is-a" Transaction
public class WithdrawalTransaction extends Transaction {

    public WithdrawalTransaction(BigDecimal amount) {
        super(amount); 
    }

    @Override
    public void executeTransaction() {
        // Specific logic for withdrawing funds
        System.out.println("Processing Withdrawal of $" + this.getAmount() + 
                           " for Transaction ID " + this.getId());
        // E.g., Database call to debit account and check for overdraft limits...
    }
}
```

### 3. Polymorphism

**Polymorphism** allows objects of different types to be treated as instances of the same class through a common interface or superclass. It lets one method name be used for different types, enabling dynamic method dispatch during runtime.

In the example below, the `TransactionProcessor` takes a `List<Transaction>`. Because of polymorphism, this list can contain `DepositTransaction`, `WithdrawalTransaction`, or any other subclass of `Transaction`. The processor iterates through uniformly and calls `executeTransaction()`. The Java Virtual Machine dynamically resolves the correct overloaded method to invoke based on the actual real-time object type (Deposit or Withdrawal).

```java
import java.util.List;

public class TransactionProcessor {

    // Polymorphism: Accepts any object that inherits from the abstract Transaction class
    public void processAllTransactions(List<Transaction> transactions) {
        System.out.println("--- Starting Batch Transaction Processor ---");
        
        for (Transaction transaction : transactions) {
            // JVM dynamically determines whether to call Deposit or Withdrawal execution logic
            transaction.executeTransaction(); 
        }
        
        System.out.println("--- Batch Processing Complete ---");
    }
}

// Example Execution
import java.util.Arrays;
import java.math.BigDecimal;

public class Main {
    public static void main(String[] args) {
        TransactionProcessor processor = new TransactionProcessor();

        // Adding different types of transactions into a unified list
        Transaction t1 = new DepositTransaction(new BigDecimal("1500.00"));
        Transaction t2 = new WithdrawalTransaction(new BigDecimal("200.50"));
        Transaction t3 = new DepositTransaction(new BigDecimal("50.00"));

        // Polymorphism in action
        processor.processAllTransactions(Arrays.asList(t1, t2, t3));
    }
}
```
