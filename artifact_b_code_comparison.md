# Artifact B: Code Review and Language Comparison

**Context:** Early in the development of the FinPay application, a junior developer initially implemented complex, data-heavy validation logic for financial transactions entirely within the frontend JavaScript client. Mentoring this developer through a code review became a profound learning experience for my own architectural understanding. While guiding them to migrate this logic to a secure, backend Java service, I solidified my comprehension of the fundamental, practical differences between scripting languages and compiled programming languages.

I learned that while JavaScript's dynamic typing and browser execution offer rapid UI feedback, they are inherently insecure for business logic because the execution environment is dictated by the client. Conversely, translating this logic into Java taught me to appreciate how the JVM provides a controlled, secure environment where compile-time static type-checking drastically reduces runtime errors. This experience shifted my perspective from simply "writing code that works" to "designing systems that are secure and robust by architecture."

Below is the side-by-side comparison of the initial JavaScript implementation and the refactored Java backend solution.

---

### 1. Initial Implementation: Frontend JavaScript (Scripting Language)

*This code was originally written to run in the user's web browser.*

```javascript
// transactionValidation.js (Client-side Script)
// Receives an untyped payload from the UI form

function validateTransaction(payload) {
    let errors = [];

    // JavaScript is dynamically typed. We must manually check types at runtime.
    if (typeof payload.amount !== 'number' || payload.amount <= 0) {
        errors.push("Invalid amount. Must be a positive number.");
    }

    // Checking complex business logic on the client side exposes rules to the user
    // and relies on the browser's JavaScript engine (V8, SpiderMonkey) to interpret the code.
    if (!payload.accountId || payload.accountId.length !== 10) {
        errors.push("Account ID must be exactly 10 characters.");
    }

    if (payload.currency !== "USD" && payload.currency !== "EUR") {
         errors.push("Unsupported currency type.");
    }

    if (errors.length > 0) {
        console.error("Validation failed:", errors);
        return { valid: false, errors: errors };
    }

    return { valid: true };
}
```

### 2. Refactored Implementation: Backend Java (Compiled Programming Language)

*This updated code runs on the Java Virtual Machine (JVM) within our Spring Boot microservice.*

```java
// TransactionValidator.java (Server-side Java)
package net.finpay.core.service;

import net.finpay.core.model.TransactionPayload;
import org.springframework.stereotype.Service;
import java.util.ArrayList;
import java.util.List;

@Service
public class TransactionValidator {

    // Java utilizes strict, static typing. The 'TransactionPayload' object 
    // guarantees the data structure before the method is even executed.
    public ValidationResult validate(TransactionPayload payload) {
        List<String> errors = new ArrayList<>();

        // Type checking is handled at compile-time. Here, we only need to verify the business constraints.
        if (payload.getAmount() == null || payload.getAmount().compareTo(java.math.BigDecimal.ZERO) <= 0) {
            errors.add("Invalid amount. Must be a positive number.");
        }

        if (payload.getAccountId() == null || payload.getAccountId().length() != 10) {
            errors.add("Account ID must be exactly 10 characters.");
        }

        if (!"USD".equals(payload.getCurrency()) && !"EUR".equals(payload.getCurrency())) {
            errors.add("Unsupported currency type.");
        }

        return new ValidationResult(errors.isEmpty(), errors);
    }
}
```

---

### Analysis: Scripting vs. Programming Languages

Moving this logic from JavaScript to Java highlights the architectural and linguistic differences between the two technologies:

1. **Execution Environment (Browser vs. JVM):**
   * **JavaScript:** The original script was interpreted at runtime directly in the user's web browser. While this allows for rapid, asynchronous UI feedback, running financial validation on the client is insecure, as the code can be inspected and manipulated by the end-user.
   * **Java:** The refactored Java code is compiled into bytecode (`.class` files) and executed persistently on the highly secure Java Virtual Machine (JVM) on our backend servers.

2. **Typing Systems (Dynamic vs. Static):**
   * **JavaScript:** JavaScript relies on dynamic typing. The `validateTransaction` function accepts any object (`payload`), forcing us to write explicit, runtime checks (e.g., `typeof payload.amount !== 'number'`) to prevent errors. 
   * **Java:** Java enforces strict, static typing. The payload must be mapped to a strongly-typed `TransactionPayload` class containing precisely defined fields (like `BigDecimal amount`). If the wrong data type is passed, the program fails safely at compile-time, rather than failing unexpectedly during execution.

3. **Architectural Use Cases:** 
   * **Scripting Languages:** JavaScript excels at DOM manipulation, client-side routing, and providing immediate interactivity on the frontend.
   * **Programming Languages:** Java is designed for robust, secure, and multithreaded backend processing. By moving our data-heavy validation to Java, we protect our business logic, ensure data integrity before persistence (via Hibernate/PostgreSQL), and leverage the enterprise capabilities of the Spring Framework.
