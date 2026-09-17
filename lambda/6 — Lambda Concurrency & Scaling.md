🧠 Topic 6 — Knowledge Check

Q1) **Basic**

What does concurrency mean in Lambda?

If 20 Lambda invocations are executing at the same time, what is the concurrency?

Ans) Concurrenty is the number of executions happening concurrently. This is not th count of warm execution server or number of lambda invocations/requests.
```
concurrency == no of concurrent executions currently in progress.
```

Hence, 20 lambda invocations are executing at the same time means, concurrency = 20.

---

Q2) **Execution environments**

Suppose we have:
```
10 concurrent invocations
```
Can you normally expect one execution environment to process all 10 simultaneously?

If not, what happens?

Ans 2) No, 10 concurrent invocations require 10 concurrent execution environments/capacity units for that function.

AWS may reuse existing warm environments where possible and create new ones where necessary.

So:
```
10 concurrent invocations
        ↓
10 concurrent execution environments
        ↓
Some may already be warm
Some may require creation
```
The important rule is:

**One execution environment doesn't simultaneously process multiple Lambda invocations.**

---

Q3) **Sequential vs concurrent**

Consider:
```
Invocation 1 → finishes
Invocation 2 → starts
Invocation 3 → starts
```
Could the same execution environment potentially handle all three?

Now consider:
```
Invocation 1 ──────────────── still running
Invocation 2 ──────────────── arrives
```
Why might Lambda need another execution environment?

Ans 3) No, in first case the same execution environment will not handle all three becuause invocation 1 create an execution environment, after invocation 1 is finished, we got two concurrent invocations 2,3, but we have only 1 warm environment which can be reused so we need 1 more environement to handle invocation 3.

The Lambda need another execution environment because it does not work like a normal nodejs process,  where all request are taken care by single nodejs process using the event loop.

### Mindmap
```
Sequential
──────────────►
Same environment may be reused


Concurrent
──────────────►
Multiple environments/capacity required
```

---

Q4) **State**

Suppose:
```
let counter = 0;
```
and Lambda has:
```
Environment A
Environment B
```
Environment A has:
```
counter = 10
```
Can Environment B automatically see counter = 10?

Why?

Ans 4) No Environment B is totally different isolated execution environment and therefore will not have the same state for counter variables (for both global and local).

The scope of variables are limited to execution environments.

---

Q5) **Database — important**

You have:
```
Lambda concurrency = 100
```
and each execution environment creates:
```
5 database connections
```
What is the potential maximum number of database connections?

Why is this dangerous for RDS?

Ans 5) The potential maximum number of database connections =  500;
This is dangerous becuase the Lambda can scale but the underlying dependencies like RDS might have limitations, this will create problem in production.

```
Lambda can scale faster than RDS can handle the resulting workload.
```

---

Q6) **Reserved vs provisioned concurrency**
Explain the difference in your own words:
```
Reserved concurrency
```
versus:
```
Provisioned concurrency
```
Try to explain each in one sentence.

Ans 6) **Reserved concurrency** = maximum number of concurrent executions allowed for a function, while also reserving that amount of account concurrency for that function.

**Provisioned concurrency** keeps a configured number of execution environments initialized and ready to respond, reducing cold-start latency.


### Mindmap
```
Reserved concurrency
        ↓
"How many can run?"


Provisioned concurrency
        ↓
"How many should be ready?"
```

Q7) **Real-world architecture**

You have:
```
API Gateway
      ↓
Payment Lambda
      ↓
Payment Provider API
```

The payment provider allows only:
```
100 requests/second
```
but Lambda can potentially process much more.

What problem could occur?

What Lambda mechanism might you consider to help control the traffic?

Don't worry if you don't know the complete answer yet.

Ans 7) We use the "Reserved Concurrenry" mechanism to help control the traffic.
so if we set it to 100, the max concurrent requests sent to the payment provider is 100.

Q8) **Senior-level** ⭐

Consider:
```
                    Lambda
                      │
                      ▼
                     RDS
```
Traffic suddenly increases from:
```
10 concurrent requests
```
to:
```
1,000 concurrent requests
```
Explain the chain of events you would be concerned about.

Try to connect:
```
Concurrency
→ Execution environments
→ DB connections
→ RDS
→ Application behavior
```

Ans 8) If the traffic suddenly increases for 10 to 10,000 to handle the this the lambda will scale horizontally meaning will have 10,000 execution evnvironments and hence will have 10,000 DB connections to RDS. Therefore the DB will give problem.
Not sure what kind of issue the RDS will throw and how it is handled.


Q9) **Bonus**

What's the difference between:
```
Concurrency = 0
```
and:
```
0 warm execution environments
```
Are they necessarily the same thing?

Ans 9) Concurrency = 0 means there are no executions in progress all invocations are finised.

0 warm execution environments means there are 0 execution environments, for the next invoation the AWS have to create a new execution environment which means a delay in request and response due to cold start (execution enviroment initialization).

---

Q10)

A Lambda receives:
```
50 requests/second
```
and each invocation takes approximately:
```
2 seconds
```
Approximately what concurrency should you expect?

Ans 10) Apporximately 50 * 2 = 100  concurrency.

Q11) Now the request rate remains:

50 requests/second

but the Lambda becomes slower:

10 seconds per invocation

Approximately what happens to concurrency?

Why?

Ans 11) 50 * 10 = 500 concurrency.

Because formula: Concurrency ≈ Request rate × Average duration

Q12 ⭐

Which is larger:
```
100 requests/second
×
0.1 seconds duration
```
or:
```
100 requests/second
×
5 seconds duration
```
Calculate the approximate concurrency for both.

Ans 12) 
100 requests/second × 0.1 = 10 concurrency
100 requests/second × 5 = 500 concurrency is larger

---

Q13) Important

True or false:

"If I configure reserved concurrency to 100, my Lambda can process a maximum of 100 requests per second."

Explain your answer.

Ans 13) False, because the concurrency also depends on the duration/time taken to complete one invocation.

eg:
if duration = 1 sec, concurrency = 100
if duration = 0.1 sec, concurrency = 10

---

Q14) **Architecture**
You have:
```
Lambda
   ↓
RDS
```
Your Lambda receives:
```
200 requests/sec
```
Average duration:
```
5 seconds
```
Each environment creates:
```
3 database connections
```
Approximately how many concurrent Lambda executions could you have?

And potentially how many DB connections?

Ans 14) 200 * 5 = 1000 concurrency
and 1000 * 3  = 3000 db connections.

Q15) ⭐ **Senior-level**

Your Lambda is currently:
```
100 requests/sec
1 second average duration
```
So concurrency is approximately:
```
100
```
Then an external API becomes slow and Lambda duration increases to:
```
10 seconds
```
The incoming traffic remains exactly:
```
100 requests/sec
```
Explain what happens to:
```
Concurrency
Execution environments
Database connections
Potential RDS pressure
```
This question is especially important because it connects performance → concurrency → infrastructure pressure.

Ans 15)
Concurrency: 100 * 10 = 1000
Execution environments: 1000
Database connections: 1000 (assuming 1 connetion per environment.)
Potential RDS pressure: Yes RDS might not scale as fast as lambda and this is reason We need to make sure our downstream dependencies can tolerate the concurrency that Lambda can generate. This can be handled by placing a SQS queue.


### TakeAway

You've now understood the most important relationship in Lambda scaling:

**Higher request duration → higher concurrency, even when request rate doesn't change.**

## 🧠 Part 3 Knowledge Check
Q16) Your Lambda has:
```
Reserved concurrency = 50
```
Currently:
```
50 invocations are running
```
A new invocation arrives.

What happens?

Ans 16) If 50 invocations are already running then the lamdba will throttle, as it already reached the maxmimum number of Maximum concurrent executions that this lambda can handle.

---

Q17)

Is this statement true or false?

**"Reserved concurrency of 100 means Lambda will always have 100 warm execution environments."**

Explain.

Ans) False, warm execution envitionments belongs to provisioned environments.

Reserved concurrency is how many execution envirionments a function is allowed to have.

Provisioned concurrency is the how many execution environments should be initialized and ready.

---

Q18) Is this statement true or false?

**"Provisioned concurrency of 20 means the Lambda can never execute more than 20 concurrent invocations."**

Why?

Ans 18) False, provisioned concurrency of 20 means we have 20 execution environments initialised and ready which will be reused for new invocations. But if a lambda gets more than 20 based on the reserved concurrency it can create more execution environments to handle those invocations.

---

Q19) ⭐

Explain the difference:
```
Reserved concurrency = 100
Provisioned concurrency = 20
```
What does each number represent?

Ans 19) Reserved concurrency = 100 means max 100 concurrent invocations can be handeled at a point of time for a function.
Provisioned concurrency = 20 means 20 execution environments will be in warm state(initialized and ready) to decrease the latency. 

---

Q20) **Architecture**

You have:
```
API Gateway
      ↓
Lambda
      ↓
RDS
```
RDS starts failing when Lambda reaches approximately:
```
200 concurrent executions
```
Would it make sense to configure:
```
Reserved concurrency = 100
```
Why?

What trade-off does this introduce?

Ans 20) So if we reduce reserved concurrency to 100, this will solve the underlying dependency (RDS) issue as it can not scale too fast, but we cant randomly put any number to reserved concurrency becuase it depends on the duration and rate limit of underlying dependencies like a payment API etc.

---

Q21) **Rate vs concurrency**

Your external API allows:

100 requests/sec

You configure:

Lambda reserved concurrency = 100

Can you guarantee that Lambda sends no more than 100 requests/sec to the external API?

Explain why or why not.

Ans 21) No, we can't guarantee it depends on the duration, if duration is less than 0.1 sec then the lamdba can make 10 req/sec for each environment.

Assuming the duration is 0.1 sec per invocation, if we get more than 100 concurrent invocations then as we have reserved concurrency set to 100, the lambda can spin 100 execution environments, as the first set of 100 request will complete within a sec and next set of invocations will also be executed in the same sec so the external API may get more then 100 requerts/sec.
```
Concurrency ≈ Request rate × Average duration
```

Therefore:
```
Request rate ≈ Concurrency / Average duration
```
For example:
```
Reserved concurrency = 100
Average duration = 0.1 sec

≈ 100 / 0.1
≈ 1,000 requests/sec
```

So a function capped at **100 concurrent executions could potentially generate ~1,000 requests/sec** if each invocation lasts only 100 ms.

That's exactly why concurrency is not a rate limiter.

**Think of it this way**
```
Concurrency = how many workers are busy

Request rate = how quickly work is being completed
```
100 workers doing tiny jobs can process vastly more requests/sec than 100 workers doing slow jobs.

---

Q22 ⭐ **Senior-level**

You have:
```
100 requests/sec
```
Average Lambda duration:
```
10 seconds
```
Therefore approximately:
```
1,000 concurrency
```
But your downstream RDS can safely handle only:
```
100 concurrent database operations
```
What architectural options would you consider?

Think about:
```
Reserved concurrency
SQS
RDS Proxy
Connection pooling
Reducing Lambda duration
```
You don't need to know exactly how each works yet. I want to see your architectural thinking.

Ans 22)
As the RDS can handle only 100 concurrent requests we should think if a system can handle on a whole, so to map with RDS limits we need to reduce the Reserved concurrency = 100.

But the trade off is if we get more the 100 requests, then the requets will throttle, to handle this we can add SQS queue, now because the maxmimum execution environments will be 100 so the RDS will not receive more then 100 concurrent requests.

Yes, we can reduce the lamda duration by adding memory by several means like add more momory to the lambda function(more compute capacity), but this will make lamdba execute more request/sec but RDS performance will remain the same.

The DB performance can be improved by put a cache layer for read operations like redis, this will reduce the db read operations by this we can again increase the reserved concurrency.

But one important distinction:

**Caching improves the workload reaching RDS; it doesn't increase RDS's intrinsic capacity.**


## 🧠 Your Lambda concurrency mental model is now ready
You should now be able to reason about this:
```
                Traffic
                   │
                   ▼
             Request rate
                   │
                   ▼
              Lambda
                   │
          ┌────────┴────────┐
          │                 │
    Concurrency         Duration
          │                 │
          └────────┬────────┘
                   ▼
       Execution environments
                   │
                   ▼
          Downstream systems
          ┌────────┼────────┐
          ▼        ▼        ▼
         RDS     Redis    APIs

```

And the critical engineering principle:

**Lambda can scale quickly, but your downstream dependencies may not scale at the same rate.**

That's one of the most important concepts in serverless architecture.