+++
title = 'Playwright Architecture'
date = '2026-09-07'
draft = false
description = 'My understanding of Playwright architecture, client libraries, automation processes, protocols, CDP, and browser communication.'
tags = ['Playwright', 'Architecture', 'CDP', 'Automation']
categories = ['Playwright Fundamentals']
+++

# Playwright Architecture --- Corrected Notes

## How Playwright Automation Works

-   Using Playwright Client Libraries, we write our automation logic.

    -   Example: `page.goto()`, `locator.click()`, `locator.fill()`.

-   Playwright Test Runner executes our tests and our Playwright API
    calls are handled through the Client Library.

-   When we run Playwright, Playwright automatically starts and manages
    the required browser automation processes.

-   The Client Library tells the Playwright automation process what
    operation we want to perform on the browser.

    -   Example: `locator.click()` → "I want to click this element."

-   Playwright communicates the request using its internal
    protocol/communication mechanism.

    -   We can think of the information as structured/JSON-like data.
    -   Important: Don't say that Playwright always communicates using
        WebSocket.

------------------------------------------------------------------------

## WebSocket Protocol

-   WebSocket provides a persistent, two-way communication channel
    between a client and server.

-   Once the connection is established, both sides can exchange multiple
    messages through that connection until it is closed.

-   WebSocket is about maintaining communication, not authorization.

------------------------------------------------------------------------

## HTTP

-   HTTP follows a request → response model.

-   The client sends a request and the server returns a response.

-   Authentication is application-dependent.

    -   It is not correct to say that every HTTP request must be
        separately authorized.
    -   Cookies, session IDs, tokens, etc. may be used depending on the
        application.

-   The server does not necessarily "forget" the client after every
    response.

------------------------------------------------------------------------

## Playwright Automation Process

-   Playwright's automation process receives and understands the
    requested operation.

-   It uses the appropriate browser-specific implementation to
    communicate with the target browser.

    -   Chromium → Chromium implementation
    -   Firefox → Firefox implementation
    -   WebKit → WebKit implementation

-   For Chromium, CDP (Chrome DevTools Protocol) can be used for browser
    communication in relevant scenarios.

    -   Don't think of Playwright as simply a CDP wrapper.

-   The browser receives the instruction and performs the actual action.

-   The result comes back through the Playwright automation layer to the
    Client Library and finally to our test.

------------------------------------------------------------------------

## What Is a Protocol?

> A protocol is a set of agreed rules that tells two parties **HOW**
> they should communicate with each other.

The protocol defines how the two systems communicate, and through that
communication one system can invoke functionality exposed by the other
system.

------------------------------------------------------------------------

## Your Software Example

``` text
My Software
    ↓
HTTP protocol
    ↓
Other application's API
    ↓
Application logic
    ↓
Result
```

------------------------------------------------------------------------

## Playwright + Chromium Example

``` text
Playwright
    ↓
CDP protocol
    ↓
Chromium's CDP interface / commands
    ↓
Chromium internal logic
    ↓
Browser action
```

------------------------------------------------------------------------

# Chromium + CDP

-   For Chromium, Playwright's Chromium-specific implementation can
    communicate with Chromium using CDP.

-   CDP = Chrome DevTools Protocol.

-   CDP is a protocol, meaning it defines rules, commands, parameters,
    responses and events for communication with Chromium.

## Example of a CDP Message

``` json
{
  "id": 42,
  "method": "Input.dispatchKeyEvent",
  "params": {
    "type": "keyDown",
    "key": "K"
  }
}
```

### Understanding the CDP Message

-   `"method": "Input.dispatchKeyEvent"` → CDP command/method

-   `"params"` → Information/arguments for that command

-   `"id": 42` → Identifier for the request

Chromium receives the CDP command through its CDP implementation and
executes the corresponding internal browser logic.

------------------------------------------------------------------------

# Browsers

-   Playwright supports Chromium-based browsers such as Chrome, Edge,
    Opera, and Brave through the Chromium browser engine.

-   Since Firefox and Safari are not Chromium-based browsers, Playwright
    uses Playwright-supported Firefox and WebKit browser builds for
    Firefox and Safari-related browser automation.

-   When we install Playwright browsers using:

``` bash
npx playwright install
```

Playwright downloads these browser binaries:

-   Chromium

-   Firefox

-   WebKit

-   Playwright supports Chromium-based branded browsers such as Chrome,
    Edge, Opera, and Brave, and also provides three browser binaries:

    -   Chromium
    -   Firefox
    -   WebKit

------------------------------------------------------------------------

# Playwright Browser Mental Map

``` text
                    Playwright
                        │
          ┌─────────────┼─────────────┐
          ↓             ↓             ↓
      Chromium       Firefox        WebKit
          │
          │
    Chromium-based
      browsers
          │
     ┌────┼────┬────┐
     ↓    ↓    ↓    ↓
  Chrome Edge Opera Brave
```

------------------------------------------------------------------------

# Complete Playwright Communication Flow

``` text
┌─────────────────────────────┐
│       Playwright Test       │
│        / Test Code          │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│     Playwright Client       │
│          Library            │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│   Playwright Automation     │
│          Process            │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│ Browser-specific            │
│ implementation              │
└──────────────┬──────────────┘
               ↓
        Target Browser
               ↓
        Browser Action
               ↓
             Result
               ↓
     Playwright Automation
             Layer
               ↓
      Client Library
               ↓
            Test
```

------------------------------------------------------------------------

# Example: `locator.click()`

When we write:

``` typescript
await page.locator('#login').click();
```

A simplified way to understand the flow is:

``` text
My Test
    ↓
locator.click()
    ↓
Playwright Client Library
    ↓
Playwright Automation Process
    ↓
Browser-specific implementation
    ↓
Target Browser
    ↓
Element is clicked
    ↓
Result
    ↓
Playwright Automation Process
    ↓
Client Library
    ↓
My Test
```

The important point is:

> **Our test code does not directly control the browser.**

The Playwright automation layer handles the communication and browser
automation.

------------------------------------------------------------------------

# Important Understanding About WebSocket

WebSocket and the protocol used for browser automation are concepts that
should not be confused.

``` text
WebSocket
    ↓
A communication channel / transport mechanism

Protocol
    ↓
Rules for how messages are structured and understood
```

Therefore:

> **Do not say that Playwright always communicates using WebSocket.**

WebSocket describes a way of maintaining two-way communication between
systems. It does not itself define what the messages mean.

------------------------------------------------------------------------

# Important Understanding About HTTP

HTTP follows a:

``` text
Request
   ↓
Response
```

model.

However, authentication is application-dependent.

For example, an application may use:

``` text
Cookies
Session IDs
Tokens
Other authentication mechanisms
```

Therefore:

> **It is incorrect to say that every HTTP request must be separately
> authorized.**

Also:

> **The server does not necessarily "forget" the client after every
> response.**

Application state can be maintained through mechanisms such as cookies,
sessions, and tokens.

------------------------------------------------------------------------

# Protocol --- My Simple Understanding

The easiest way I remember a protocol is:

``` text
Protocol
    ↓
Agreed rules
    ↓
Define HOW two systems communicate
```

For example:

``` text
My Software
     ↓
HTTP
     ↓
Other Application API
```

Here:

-   **API** → What functionality can I request?
-   **Protocol** → How do we communicate the request?

------------------------------------------------------------------------

# CDP --- My Simple Understanding

CDP stands for:

**Chrome DevTools Protocol**

It is a protocol used for communication with Chromium-based browser
functionality.

A CDP message can contain:

``` text
ID
 ↓
Identifies the request

Method
 ↓
Specifies the command

Parameters
 ↓
Provides information required by the command
```

Example:

``` json
{
  "id": 42,
  "method": "Input.dispatchKeyEvent",
  "params": {
    "type": "keyDown",
    "key": "K"
  }
}
```

------------------------------------------------------------------------

# Important: Playwright Is Not Simply a CDP Wrapper

For Chromium, CDP can be involved in browser communication in relevant
scenarios.

However:

> **Playwright should not be thought of as simply a CDP wrapper.**

Playwright provides a higher-level automation API and handles browser
automation across multiple browser engines.

``` text
Playwright
    │
    ├── Chromium
    │
    ├── Firefox
    │
    └── WebKit
```

This is one reason Playwright is different from simply using CDP
directly.

------------------------------------------------------------------------

# Final Mental Map

The simplest way I remember Playwright architecture is:

``` text
                  MY TEST
                     ↓
          PLAYWRIGHT CLIENT
               LIBRARY
                     ↓
        PLAYWRIGHT AUTOMATION
                PROCESS
                     ↓
        BROWSER-SPECIFIC
          IMPLEMENTATION
                     ↓
          ┌──────────┼──────────┐
          ↓          ↓          ↓
      Chromium    Firefox     WebKit
          │
          ↓
         CDP
   (relevant scenarios)
          │
          ↓
     BROWSER ACTION
          │
          ↓
        RESULT
          │
          ↓
       MY TEST
```

------------------------------------------------------------------------

# Key Takeaways

1.  **Client Library**

    -   We write our Playwright automation using Client Library APIs.
    -   Examples:
        -   `page.goto()`
        -   `locator.click()`
        -   `locator.fill()`

2.  **Playwright Test Runner**

    -   Executes our tests.

3.  **Playwright Automation Process**

    -   Receives and processes the requested browser operation.

4.  **Browser-specific Implementation**

    -   Handles communication with the appropriate browser engine.

5.  **Protocol**

    -   Defines the rules for communication between systems.

6.  **WebSocket**

    -   Provides a persistent, two-way communication channel.
    -   It is a communication mechanism, not an authorization mechanism.

7.  **HTTP**

    -   Uses a request → response model.
    -   Authentication is application-dependent.

8.  **CDP**

    -   Stands for Chrome DevTools Protocol.
    -   Defines commands, parameters, responses, and events for
        communication with Chromium.

9.  **Playwright is not simply a CDP wrapper.**

10. **Playwright supports three main browser binaries:**

    -   Chromium
    -   Firefox
    -   WebKit

11. **Chromium-based branded browsers include:**

    -   Chrome
    -   Edge
    -   Opera
    -   Brave

------------------------------------------------------------------------

# One Sentence to Remember

> **I write automation using Playwright Client APIs; Playwright's
> automation process handles the request through the appropriate
> browser-specific implementation, which ultimately causes the target
> browser to perform the requested action.**
