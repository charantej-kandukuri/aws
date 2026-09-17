
# Lambda Handler


Keep this mental shortcut:

event
  ↓
"What happened / what data was sent?"

context
  ↓
"Information about this Lambda execution"

```JavaScript
export const handler = async (event, context) => {

    console.log(event);                  // Input
    console.log(context.awsRequestId);   // Execution information

};
```

## What is Handler?

Handler is the __exported function name__, configured as Lambda's entry point.

## This is an important Lambda pattern
```JavaScript
// Initialization
const db = createDatabaseConnection();

export const handler = async (event) => {

    // Invocation-specific logic
    return db.query(...);

};
```

_Execution-environment reuse is an optimization, not persistent storage._


## Knowledge Check
Q1) What is the difference between: event and context?

A1) 

event &rarr; "What happened / what data was sent?"

context &rarr; "Information about this Lambda execution"

Q2) Why does AWS need a handler? What does this configuration mean?
`index.handler`

A2) Handler is needed beacuse AWS lambda needs to know the extry point to execute the code.

index.handler => index is the file name and handler is the method or entry point of lambda code execution.

```
index
  ↓
index.js / index.mjs
  ↓
handler
  ↓
exported function

```
handler is the __exported function name__, configured as Lambda's entry point.

Q3) If API Gateway sends:
```json
{
    "name": "Charan"
}
```
to Lambda, where would you expect "Charan" to appear?

A3) If API Gateway sends, we can find the data in the "event" object.

For example, HTTP request information can appear in fields such as:

```javascript
event.body
event.queryStringParameters
event.pathParameters
event.headers
```


Q4) Why do we generally prefer:
```Javascript
export const handler = async (...) => {
    return result;
};
```
over the older:
```javascript
callback(null, result);
```

A4) we generally perfer the modern JavaScript ES2025 syntax to handle promises with async and await, because it is more readable.


Q5) Q5 — Important
Consider:
```javascript
const db = createDatabaseConnection();

export const handler = async (event) => {
    await db.query("...");
};
```
Why can this be more efficient than:
```javascript
export const handler = async (event) => {
    const db = createDatabaseConnection();

    await db.query("...");
};
```
A5) It is more efficient becuase in the first case the initialization happens when module loaded, while in the second case the it happens on every handler invocation.

Limitation:

Execution environments can disappear, so the state isn't permanent.

This is an important Lambda pattern:
```javascript
// Initialization
const db = createDatabaseConnection();

export const handler = async (event) => {

    // Invocation-specific logic
    return db.query(...);

};
```

Think
```
// Initialization
const db = createDatabaseConnection();

export const handler = async (event) => {

    // Invocation-specific logic
    return db.query(...);

};
```
But AWS can eventually destroy that environment:
```
Environment A
     ↓
   destroyed
     ↓
Environment B
     ↓
New DB connection
```

So:

__Execution-environment reuse is an optimization, not persistent storage.__

_That's an important senior-level distinction._

