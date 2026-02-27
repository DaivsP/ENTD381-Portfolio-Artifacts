# Artifact E: Spring Boot Web Layer & Integration Tests

## Narrative
Developing web applications using Java has been a continuous thread throughout my career. I initially learned the mechanics of HTTP by building web applications using raw Java Servlets—manually parsing request streams and printing dynamic HTML responses. That granular experience laid the foundation for my transition into mastering modern, enterprise-grade frameworks.

I now oversee the development of RESTful web services utilizing the Spring Boot framework. Learning Spring Boot shifted my paradigm from "writing web servers" to "architecting decoupled APIs." I learned how to utilize powerful declarative annotations (`@RestController`, `@PostMapping`) to map complex URIs to specific backend methods, seamlessly handling JSON serialization and deserialization without manual boilerplate. Furthermore, integrating Object-Relational Mapping (ORM) tools securely connected these web layers to our underlying databases.

However, the most significant milestone in my web development journey was internalizing the absolute necessity of automated presentation-layer testing. I learned that an API contract isn't truly robust until it is programmatically verified. By mastering Spring's `MockMvc` framework, I learned how to simulate client HTTP requests to guarantee our web controllers return the correct JSON HTTP status codes (`2xxSuccessful`, `5xxInternalServerError`) without needing to launch a full web server or a physical browser. This test-driven approach ensures that our web applications remain highly available and contractually sound during rapid deployment lifecycles.

**Below are sanitized Java snippets from a production financial repository I manage.** The first snippet demonstrates a REST Controller handling complex routing and payload extraction. The second snippet acts as my artifact of learning, showing the integration test suite utilizing `MockMvc` to mathematically prove the web layer operates correctly.

---

## Technical Implementation 1: The Spring Boot REST Controller

This snippet demonstrates my ability to architect the communication layer between external clients and our internal services. It showcases dynamic URL path variable extraction (`@PathVariable`), JSON body deserialization (`@RequestBody`), and structured `ResponseEntity` returns.

```java
package net.finpay.core.controller;

import net.finpay.core.model.payment.PaymentDetail;
import net.finpay.core.model.payment.PaymentReversal;
import net.finpay.core.model.payment.Transaction;
import net.finpay.core.service.PaymentService;
import net.finpay.core.service.TransactionService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.math.BigInteger;
import java.time.ZonedDateTime;
import java.util.List;

// Declares that this class handles web requests and automatically serializes returns into JSON
@RestController
@RequestMapping("/api/v1") // Base routing path
public class PaymentController {

    @Autowired
    private PaymentService paymentService;

    @Autowired
    private TransactionService transactionService;

    /**
     * Endpoint to create a new financial payment.
     * Demonstrates capturing dynamic ID parameters from the URL path, alongside a complex JSON payload.
     */
    @PostMapping(value = "/patient/{patientId}/encounter/{encounterId}/payment")
    public ResponseEntity<PaymentDetail> createNewPayment(
            @PathVariable("patientId") BigInteger patientId,
            @PathVariable("encounterId") BigInteger encounterId,
            @RequestBody PaymentDetail paymentDetail) {

        // Internal business logic linking the payload to the URL identities
        paymentDetail.setPatientId(patientId);
        paymentDetail.setPatientEncounterId(encounterId);
        paymentDetail.setPaymentInitDt(ZonedDateTime.now());

        paymentService.createAndChargePayment(encounterId, paymentDetail);

        // Web logic: Returning appropriate HTTP Status Codes based on transaction success or failure
        if (paymentDetail.isPaymentStatusIsStripeException()) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(paymentDetail);
        } else {
            return ResponseEntity.ok(paymentDetail); // 200 OK
        }
    }

    /**
     * Endpoint to retrieve a history of transactions.
     * Demonstrates a GET request mapping.
     */
    @GetMapping(path = "/patient/{patientId}/encounter/{encounterId}/transactions")
    public List<Transaction> getEncounterTransactions(
            @PathVariable("patientId") BigInteger patientId,
            @PathVariable("encounterId") BigInteger encounterId) {
            
        // Spring Boot automatically converts the returned Java List into a JSON array for the client
        return transactionService.getEncounterTransactions(encounterId, patientId);
    }
}
```

---

## Technical Implementation 2: Integration Testing the Web Layer (MockMvc)

This snippet demonstrates my commitment to test-driven web architecture. Using `MockMvc`, we simulate incoming HTTP traffic to verify our REST APIs behave exactly as documented without requiring a physical network connection. 

```java
package net.finpay.core.controller;

import net.finpay.core.TestUtils;
import net.finpay.core.model.payment.PaymentDetail;
import net.finpay.core.service.PaymentService;
import net.finpay.core.service.TransactionService;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

// Narrows the test scope to only the Web layer, bypassing the database
@WebMvcTest(PaymentController.class)
class PaymentControllerTest {

    // Simulates an HTTP client
    @Autowired
    private MockMvc mockMvc;

    // Mocks the underlying business services, isolating the web layer for pure API testing
    @MockBean
    private PaymentService paymentService;

    @MockBean
    private TransactionService transactionService;

    /**
     * Programmatically tests the POST mechanism ensuring our controller correctly
     * accepts JSON payloads, maps them to the URI, and returns a successful 2xx status.
     */
    @Test
    void testCreateNewPaymentEndpoint() throws Exception {
        // Load a mock JSON payload from a test file
        PaymentDetail mockPaymentPayload = TestUtils.getPaymentDetailFromJson("create_payment_standard.json");

        // Simulate a POST request from a web client
        this.mockMvc.perform(post("/api/v1/patient/1/encounter/1/payment")
                .content(TestUtils.asJsonString(mockPaymentPayload))
                .contentType(MediaType.APPLICATION_JSON) // Sending JSON
                .accept(MediaType.APPLICATION_JSON))     // Expecting JSON back
                .andDo(print())                          // Log the exact HTTP interaction for debugging
                .andExpect(status().is2xxSuccessful());  // Assert that our API responded with HTTP 200/201
    }

    /**
     * Programmatically tests the GET mechanism.
     */
    @Test
    public void testGetEncounterTransactionsEndpoint() throws Exception {
        // Simulate a GET request
        this.mockMvc.perform(get("/api/v1/patient/1/encounter/1/transactions"))
                .andDo(print())
                .andExpect(status().is2xxSuccessful());
    }
}
```
