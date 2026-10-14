# Command Injection

A collection of beginner-friendly notes on OS command injection, covering core concepts, testing methods, and prevention techniques.

## Topics

- [Command Injection Basics](Command-Injection-Basics.md)
- [Shell Operators](Shell-Operators.md)
- [Blind Command Injection](Blind-Command-Injection.md)
- [Prevention](Prevention.md)

## Learning Objectives

- Understand how command injection vulnerabilities occur.
- Learn how to identify and verify command injection safely.
- Understand the difference between verbose and blind command injection.
- Learn secure coding practices to prevent command injection.

## Disclaimer

These notes are for educational purposes and authorized security testing only.




# OS Command Injection

## What Is It?

OS Command Injection occurs when an application passes user-controlled input to an operating system command without proper protection.

Example vulnerable PHP code:

```php
exec("ping -c 4 " . $_GET["ip"]);
```

Normal input:

```text
127.0.0.1
```

Injected input:

```text
127.0.0.1; whoami
```

Here, `;` separates commands, so the application may execute both `ping` and `whoami`.

## Common Operators

- `;` — Executes the next command regardless of the previous command's result.
- `&&` — Executes the next command only if the previous command succeeds.
- `||` — Executes the next command if the previous command fails.
- `|` — Passes one command's output to another command.
- `&` — In common Unix shells, runs a command in the background.
- `$(command)` — Substitutes a command's output in supported shells.

## Common Payloads and Their Uses

Use these examples only in authorized labs. Replace `127.0.0.1` with the application's expected input when appropriate.

**Linux payloads:**

- `127.0.0.1; whoami` — Identifies the user running the command.
- `127.0.0.1; id` — Displays the current user and group IDs.
- `127.0.0.1; pwd` — Shows the current working directory.
- `127.0.0.1; ls` — Lists files and directories.
- `127.0.0.1; cat /path/to/file` — Reads a file if the user has permission.
- `127.0.0.1 && id` — Runs `id` only if the first command succeeds.
- `127.0.0.1; sleep 5` — Introduces a five-second delay to test blind injection.
- `127.0.0.1; ping -c 4 127.0.0.1` — Runs a Linux ping command four times.

**Windows payloads:**

- `127.0.0.1 & whoami` — Identifies the current user.
- `127.0.0.1 & dir` — Lists files and directories.
- `127.0.0.1 & ipconfig` — Displays network configuration.
- `127.0.0.1 & timeout /t 5` — Tests for a five-second delay.

Payload syntax depends on the operating system and the shell used by the application.

## Verbose and Blind Injection

**Verbose command injection:** The output of the injected command appears in the application's response. For example, `127.0.0.1; whoami` may return `www-data`.

**Blind command injection:** The command executes, but its output is not displayed. A time-delay payload such as `127.0.0.1; sleep 5` may help identify it. Compare response times with normal requests and repeat the test to reduce false positives.

## Understanding Results

- `www-data` — A common Linux account used by web services.
- `Permission denied` — The command may have executed, but the current account lacks permission.
- No visible output — Does not necessarily mean the injection failed.
- A response delay — Possible evidence of blind injection, but timing alone is not conclusive.

## Testing Workflow

1. Submit normal input and observe the application's response.
2. Test a harmless command such as `whoami`.
3. Check whether the output or application behavior changes.
4. If output is hidden, test a short delay.
5. Verify the result against normal behavior and document the evidence.

## Prevention

Avoid executing OS commands when a safer library or API can achieve the same result. If execution is necessary, use safe process APIs without invoking a shell, validate input on the server, apply strict allowlists, and run the application with least privilege.

**Important terms:** Input validation checks whether input follows expected rules. Sanitization cleans or transforms input. Allowlisting accepts only explicitly permitted values. Least privilege limits the application's permissions to what it actually needs.

Client-side HTML validation alone is insufficient because it can be bypassed.
