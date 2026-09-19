# Intro to XSS

## Types of XSS

There are three main types of XSS vulnerabilities:

| Type                             | Description                                                                                                                                                                                                                                  |
| -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Stored (Persistent) XSS`        | The most critical type of XSS, which occurs when user input is stored on the back-end database and then displayed upon retrieval (e.g., posts or comments)                                                                                   |
| `Reflected (Non-Persistent) XSS` | Occurs when user input is displayed on the page after being processed by the backend server, but without being stored (e.g., search result or error message)                                                                                 |
| `DOM-based XSS`                  | Another Non-Persistent XSS type that occurs when user input is directly shown in the browser and is completely processed on the client-side, without reaching the back-end server (e.g., through client-side HTTP parameters or anchor tags) |

# DOM-Based XSS

This type of XSS is completely processed on the frontend side, nothing goes to the backend.

## Source & Sink

The **Source** is the JavaScript object that takes the user input, and it can be input parameter like a URL parameter or an input field.

**Sink** is the function that writes the user input to DOM Object on the page. If the **Sink** function doesn't properly , it would be vulnerable to **DOM XSS** attack.

*Some commonly used JavaScript functions to write to DOM objects are:*

1. `document.write()`
2. `DOM.innerHTML`
3. `DOM.outerHTML`

Furthermore, some of the `jQuery` library functions that write to DOM objects are:

- `add()`
- `after()`
- `append()`

