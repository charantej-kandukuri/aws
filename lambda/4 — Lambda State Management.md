# Lambda State Management


## State storage decision
| Storage         | Best for                    | Persistent? | Shared between environments? |
| --------------- | --------------------------- | ----------: | ---------------------------: |
| Local variable  | Current invocation          |           ❌ |                            ❌ |
| Global variable | Reusable temporary state    |           ❌ |                            ❌ |
| `/tmp`          | Temporary files             |           ❌ |                            ❌ |
| Redis           | Shared cache/state          |          ✅* |                            ✅ |
| DynamoDB        | Application data            |           ✅ |                            ✅ |
| RDS/MySQL       | Relational application data |           ✅ |                            ✅ |
| S3              | Files/objects               |           ✅ |                            ✅ |


`Redis` is persistent enough to be shared, but cache contents can be evicted depending on configuration, so don't automatically treat a cache as your system of record.

## Four levels of state

### Level 1 — Invocation state
```javascript
Level 1 — Invocation state
```
Exists during the current invocation.
```
Invocation ends
      ↓
State gone
```

### Level 2 — Execution environment state
``` javascript
const cache = new Map();
```
Potentially survives:
```
Invocation 1
      ↓
Invocation 2
      ↓
Invocation 3
```
But disappears when the environment disappears.

### Level 3 — Shared external state
```
Redis
DynamoDB
RDS
```
Multiple Lambda **environments** can access it.

### Level 4 — Durable object/file storage
```
S3
```
Useful for durable files and objects.

### MindMap

```
                  Lambda
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
      Memory      Redis      Database
       │            │            │
   temporary      cache       source of
   optimization               truth
```
This distinction is extremely important.

## Testing Lambda
Testing Lambda with sequential requests can hide **concurrency/state** problems.

## The complete mental model
```
                         Lambda
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
          Env A          Env B          Env C
             │             │             │
        ┌────┴────┐   ┌────┴────┐   ┌────┴────┐
        │ Memory  │   │ Memory  │   │ Memory  │
        │ /tmp    │   │ /tmp    │   │ /tmp    │
        │ Modules │   │ Modules │   │ Modules │
        └─────────┘   └─────────┘   └─────────┘
             │             │             │
             └─────────────┼─────────────┘
                           │
                           ▼
                  External Storage
                           │
               ┌───────────┼───────────┐
               ▼           ▼           ▼
            DynamoDB      RDS        S3/Redis
```

The environments are temporary.

External services provide shared/persistent state.

## TakeAways
1. 
Don't say:

```
Lambda doesn't have state.
```

That's too simplistic.

Instead say:
```
Lambda execution environments can have temporary/reusable state, but application state should not depend on the lifetime of an execution environment.
```

That's a much better senior-level explanation.

2. 
Execution-environment state is reusable but not reliable. Persistent application state lives outside the Lambda environment.

## 🧠 Topic 4 — Knowledge Check
Q1) What's the difference between: Execution-environment state and:Persistent application state.

A1) **Execution environment state**: includes memory, /tmp, modules cache, global variables, connections. All this belongs to the execution environments and hence if the enviroment is distroyed by AWS all the environment state is also gone. Also the state will be different in different execution environments for that lambda.

**Persistent applicatoin state**: We use resources like RDS, DyanamoDB, s3 and Redis*. The makes the application share the data between execution environments. The data is also persistent. Redis is actually used as a central cache store and it also depends on its configurations.

We should use them based on the requirement. 

we should check like this, if the execution environment is distroyed and the application is still working as expected then you can do for execution environment state else go for applicaton persistent state.

#### ChatGPT:
```
Can my application tolerate losing this data at any moment?
             │
        ┌────┴────┐
       YES        NO
        │          │
        ▼          ▼
   Memory/cache   Persistent
   /tmp           storage
```

Redis is shared external state, but not necessarily durable source-of-truth storage.

For example:
```
Database → source of truth
Redis    → fast shared cache
```

If Redis loses a cached value, the application should generally be able to reconstruct it from the source of truth.

Q2) Suppose we have:
```
let cache = new Map();
```
Why might this cache be available during invocation #2?

And why might it be completely empty during invocation #2?

Ans) The cache might be available during invocagtion #2 if the same execution environment is used.

The cahe might completely be empty during invocation #2 if a new execution environment is used or created due to concurrent requests.

Q3) You have:
```
let totalOrders = 0;

export const handler = async (event) => {
    totalOrders++;
};
```
Why is this not a reliable way to maintain the total number of orders in your application?

Ans) This is not a reliable way to maintain the total number of orders in your application because this information will be lost if the execution environment is gone, also if a lambda has multiple execution environments diff environments will be different state and hence the application will not work as expected.

Q4) Why is /tmp useful in Lambda?

Give me one real-world example where /tmp makes sense.

Ans4) The /tmp directory is a temporary directory used with in an execution environment.

Real world usage:
```
S3
 │
 ▼
Lambda
 │
 ▼
/tmp/input
 │
 ▼
Process
 │
 ▼
/tmp/output
 │
 ▼
S3
```

The original file is not effected as it is in persistant s3 storage.

But the file in /tmp folder is not guarenteed to be availble in the subsequent request because the invocation might go to diff execution environemnt wher e this file is not present (execution environment state) or the execution environment might have been distroyed already.

Q5) You have:
```
Lambda A
Lambda B
Lambda C
```
and each environment has:
```
const cache = new Map();
```
If Lambda A puts:
```
user123 → user data
```
into its Map, can Lambda B directly access it?

Why?

Ans 5) No, Lambda B can not acccess it, bacause Lambda A will have its own execution environment which is not sharable.

The important mental model is:

**Global/module-level memory is local to an execution environment, not global to the Lambda function.**

Q6 — Architecture question

Ans6) For each item below, tell me where you would store it and why:
| Data                                    | Your choice |
| --------------------------------------- | ----------- |
| Current invocation's temporary variable |local varible|
| Large temporary file during processing  | /tmp        |
| User profile                            | DynamoDB    |
| Shared cache                            | Redis       |
| Permanent uploaded image                | S3          |
| MySQL relational business data          | RDS/MySQL   |


Q7 — Senior-level

Imagine your Lambda works perfectly when you test it manually because all requests are going to the same warm environment.

Then production traffic arrives and AWS creates 20 execution environments.

What kinds of bugs could appear if your application incorrectly relies on in-memory state?

Think about:

Caches
Counters
Sessions
Configuration
Database connections

Ans 7) In production beacuse we have different execution environments:
1. we have cache issues because the in-memory state of environment is not shared, but its ok if its used only for the optimization purpose. 
2. The counters/session will be in different state in different environments it should be in a central cache like redis. 
3. If configurations/database connections are loaded into a global varibles (module level initialization) even then the scope is limited that execution envirionment it is not shared between environments.


## 🧠 Your Lambda mental model so far
```

                       Lambda Function
                              │
                 ┌────────────┼────────────┐
                 ▼            ▼            ▼
              Env A         Env B        Env C
                 │            │            │
              Memory        Memory       Memory
              /tmp          /tmp         /tmp
              Modules       Modules      Modules
              DB conn       DB conn      DB conn
                 │            │            │
                 └────────────┼────────────┘
                              │
                              ▼
                     External Services
                              │
             ┌────────────────┼────────────────┐
             ▼                ▼                ▼
          DynamoDB           S3              RDS
                              │
                            Redis
```
And the core rule:

**Never make application correctness depend on an execution environment surviving.**

That's a senior-level Lambda principle.