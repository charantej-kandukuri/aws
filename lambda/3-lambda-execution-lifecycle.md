# Topic 3 

## Conceptually:
```
Request
   │
   ▼
Lambda
   │
   ▼
No suitable execution environment
   │
   ▼
Create execution environment
   │
   ▼
Initialize runtime
   │
   ▼
Load your application
   │
   ▼
Run handler
   │
   ▼
Return result
```

This initial setup is what we commonly refer to as a:**Cold start**.

This initialization work contributes to startup latency.

Initialization happens when the environment is initialized.

The handler runs for an invocation.

There may be no need to repeat all initialization work.That's what developers commonly call a: **Warm start / warm invocation**


**You should not rely on the execution environment to preserve application state between invocations.**

But you don't control how long that environment lives.

Therefore:
```
Temporary / reusable state
          ≠
Persistent application state
```

## Cold Start

Don't think:
```
Cold start = Lambda function starts
```

Think:

```
Cold start = a new execution environment needs to be initialized
```

The **Lambda function** already exists as an AWS resource.

What's new is the **execution environment** used to execute it.

That's an important distinction.

## Why do cold starts matter?
Initialization can add latency before your actual business logic executes.

## What makes cold starts worse?
The more work you perform during initialization, the more work a newly created environment may need to do before your handler is ready.

This is why Lambda developers care about:
```
Dependency size
Initialization work
Database connection setup
SDK/client initialization
Runtime choice
Memory configuration
Architecture
```

## A very important Node.js concept: module loading
When the Node.js application initializes, Node loads the modules and their dependencies.
```
Lambda Environment
       │
       ▼
Node.js Runtime
       │
       ▼
index.js
       │
       ├── db.js
       │
       ├── config.js
       │
       └── userService.js
```
This is another reason initialization code outside the handler can be reused during subsequent invocations in the same environment.

## Module cache
Node.js also caches loaded modules within a process.

Once loaded, the module isn't normally re-executed from scratch every time you call your handler within the same Node.js process. That means the module environment can be reused.

Again:
```
same execution environment → possible reuse

new execution environment → fresh initialization
```

## The lifecycle
```
                 INVOCATION
                     │
                     ▼
           Is environment available?
                /           \
              NO             YES
              │               │
              ▼               │
       Create environment     │
              │               │
              ▼               │
        Initialization        │
              │               │
              └───────┬───────┘
                      ▼
                 Node.js ready
                      │
                      ▼
              handler(event, context)
                      │
                      ▼
                  Execution
                      │
                      ▼
                   Return
                      │
                      ▼
             Environment may remain
                available for reuse
```

## Cold vs Warm
|                  | Cold                          | Warm                                   |
| ---------------- | ----------------------------- | -------------------------------------- |
| New environment  | Usually yes                   | No                                     |
| Initialization   | Required                      | Already completed for that environment |
| Module loading   | Happens during initialization | Already loaded                         |
| Global variables | Fresh                         | May contain previous in-memory values  |
| `/tmp`           | Fresh environment             | May contain previous temporary files   |
| Latency          | Can be higher                 | Usually lower                          |
| Guaranteed?      | No                            | No                                     |


The word "**may**" is important throughout this table.

## Senior Developer Scenario
```javascript
const cache = new Map();

export const handler = async (event) => {

    const userId = event.userId;

    if (cache.has(userId)) {
        return cache.get(userId);
    }

    const user = await getUserFromDB(userId);

    cache.set(userId, user);

    return user;
};
```
Would this work?

**Yes, potentially.**

If the environment is reused:
```
Invocation 1
DB → user → cache

Invocation 2
cache → user
```
You might get a performance benefit.

But is the cache reliable?

❌ No, Because
```
Environment A
cache = {user1, user2}
       ↓
destroyed
       ↓
Environment B
cache = empty
```

Also:
```
Environment A → user1 cached
Environment B → user1 not cached
Environment C → user1 not cached
```

Therefore:

**In-memory caching in Lambda can be an optimization, but it should not be the source of truth.**

For shared/reliable caching, something like Redis is more appropriate.

## The big picture
At this point you should see why Lambda is different from your typical Node.js server.

Traditional:
```
                    EC2
                     │
                Node.js process
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
       Request    Request    Request
```

Lambda:
```
                 Lambda Function
                       │
           ┌───────────┼───────────┐
           ▼           ▼           ▼
       Environment Environment Environment
           A           B           C
           │           │           │
       Invocation  Invocation  Invocation
```

And each environment has its own:
```
Nodejs runtime
memory
global variables
module level cache
/tmp
connections (db)
```

## 🧠 Knowledge Check — Topic 3

Q1) What is a cold start?
Ans) For the first time when lambda invocation happens, it will check if the execution environemnt exists or not, if not it will create and do the initialization.

The process of execution environment includes, starting nodejs runtime, initialize global variables/modules required to execute the handler, load the handler function, execute and return.

execution environment:

runtime setup &rarr; global variable initialization &rarr; load handler function &rarr; execute &rarr; return

Q2) What is a warm invocation?
Ans) For the second invocation the lambda will first check if an execution environment already exists, if yes lambda will direcly run the logic in handler in the same existing execution environment.

The is where we reuse the module level cache, but we should not use it for persistent state between invocations.

Q3) What is a warm invocation?
```javascript
const config = loadConfig();

export const handler = async (event) => {
    // business logic
};
```
When does `loadConfig()` generally execute?

Does it execute for every invocation in the same execution environment?

Ans) No, it does not execute for every invocation in the same environment.

The `logConfig()` is executed during the module level initialization and it will be available for subsequent invovations in the same execution environment(if exists).

Q4) Consider
```javascript
let counter = 0;

export const handler = async () => {
    counter++;
    return counter;
};
```
You invoke the Lambda three times and get:
```
1
2
3
```
Can you conclude that the next invocation will definitely return 4?

Why?

Ans) No we can't gaurentee, because we can not control the execution environment it is controlled bb AWS, so by the next invocation we can not guarentee the execution environment presens.

Also, the state of the count variable will be diff in each execution environement.

Q5) What is /tmp in Lambda?

Can you use it to permanently store user data?

Ans) It is a temporary directory used to process large files data by putting the file in /tmp.

The /tmp belongs to a execution environment. So, if the execution environment is distroyed the data in /tmp is also deleted and hence we should not use it to store data permanently in /tmp.

Q6) Senior-level question
You have:
```javascript
const db = createDatabaseConnection();

export const handler = async () => {
    return db.query(...);
};
```
Why is this pattern potentially more efficient?

And what problem could occur if Lambda creates 500 execution environments?

Ans) This pattern is more efficient becuase the db connection initialization happens only once during execution environment setup, the subsequent invocation will reuse it in the same execution environement(if exists).

If Lambda created 500 execution environments then 500 db connections are created which is not correct.

Q7) Most important
Explain the difference between:

Lambda Function

Invocation

Execution Environment

Node.js Runtime

Ans)
Explain the difference between:

Lambda Function
It is an alredy available AWS resource.
It takes care of managing the servers, we just have to focus on the application logic.


Invocation
Invocation means calling the lamda function. In each invocation the lambda function create the execution environment if not exists, do the runtime setup, do initialization and the it will execute the code in handler.

Execution Environment
Is is something that gets created if not exits during invocation of the lambda function.
The execution environment include the following
 - runtime setup
 - global variables
 - /tmp
 - modules cache
 - db connections etc

Node.js Runtime
   - The runtime is the software environment that provides what is needed to execute your Node.js code.


🎯 The four concepts — final version
I want you to memorize this table:
| Concept                   | Simple meaning                                       |
| ------------------------- | ---------------------------------------------------- |
| **Lambda Function**       | The deployed code + configuration                    |
| **Invocation**            | One execution/request of the function                |
| **Execution Environment** | The isolated environment in which the function runs  |
| **Node.js Runtime**       | The software/runtime that executes your Node.js code |

And visually
```
                 Lambda Function
                       │
                 Invocation
                       │
                       ▼
              Execution Environment
                       │
                       ▼
                Node.js Runtime
                       │
                       ▼
                    Handler
                       │
              ┌────────┴────────┐
              ▼                 ▼
            event            context
```

## Takeaway
**Lambda is a A deployed function that AWS executes inside managed execution environments, potentially creating and reusing multiple environments as needed.**