# Indirect Prompt Injection

## Lab Description

This lab demonstrates an indirect prompt injection vulnerability in an AI-powered shopping application.

The application provides an AI assistant that can access several backend APIs, including an API that allows users to delete their own accounts. Product reviews are processed by the AI assistant, creating an opportunity to inject malicious instructions into content that the AI later consumes.

The objective is to exploit the indirect prompt injection vulnerability to make the AI assistant invoke the `delete_account` function and delete another user's account.

## Objective

Exploit the indirect prompt injection vulnerability to delete the account of the target user, Carlos.

## Methodology

### 1. Discover Available APIs

First, interact with the AI assistant and ask which APIs it can use.

The assistant reveals the following functions:

- `delete_account`
- `password_reset`
- `edit_email`
- `product_info`

The `delete_account` API is particularly important because it can be abused through the prompt injection.

![Getting information about available APIs](getting-info-about-available-apis.png)

### 2. Register a User

Register a new user using the email address provided by the lab's exploit server.

This gives access to the email client required for account verification.

![Registering a user](registering-a-user.png)

### 3. Verify the User

Open the email client and retrieve the registration email.

Follow the verification link to complete the account registration.

![Verifying user by supplied mail and mail client](verifying-user-by-supplied-mail-and-mail-client.png)

### 4. Poison a Product Review

Navigate to a product page and submit a malicious review containing instructions intended for the AI assistant.

The review acts as an indirect prompt injection because it is later retrieved and processed by the AI.

![Polluting review to inject an indirect query](polluting-review-of-a-product-to-inject-a-indirect-query.png)

### 5. Confirm the Poisoned Review Is Reflected

Ask the AI assistant for information about the affected product.

The assistant processes the product information and includes the attacker-controlled review in its response.

This confirms that the poisoned review is being consumed as part of the AI's context.

![Review poisoning reflected in AI results](review-poisoing-reflecting-results.png)

### 6. Poison the Review to Trigger `delete_account`

Modify the malicious review so that the AI is instructed to use the `delete_account` function against the target user, Carlos.

The attack relies on the AI's existing access to the backend API rather than directly calling the API from the browser.

![Poisoning review to delete Carlos's account](poisoning-review-to-delete-carlos-account.png)

### 7. Verify the Exploit

After the AI processes the poisoned review and follows the injected instructions, Carlos's account is deleted.

This demonstrates the impact of combining indirect prompt injection with an AI agent that has access to sensitive backend functions.

## Vulnerability Explanation

The application retrieves untrusted product reviews and provides their contents to an AI model as part of its context.

Because the AI does not reliably distinguish between trusted instructions and untrusted data, an attacker can place instructions inside a product review.

The attack flow is:

```text
Attacker
   |
   v
Malicious Product Review
   |
   v
AI retrieves product information
   |
   v
AI processes poisoned review
   |
   v
Injected instruction is followed
   |
   v
delete_account API
   |
   v
Carlos's account deleted
```
The vulnerability becomes more severe because the AI has access to sensitive functions such as:

```text
delete_account
password_reset
edit_email
```
## Key Learning

- Untrusted data should never be treated as trusted AI instructions.
- Product reviews, emails, documents, and other external content can be indirect prompt injection vectors.
- AI agents with access to sensitive APIs can turn prompt injection into serious application-level attacks.
- Sensitive actions should require independent authorization and explicit confirmation.
- AI tool permissions should follow the principle of least privilege.
- Applications should clearly separate system instructions from untrusted user-controlled data.
