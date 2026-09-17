# Lambda configuration

Q1) **Memory**

You have a Lambda using:
```
512 MB
```
and it takes 5 seconds.

You increase memory to:
```
2048 MB
```
and it now takes 1.5 seconds.

Why can increasing **memory** make a Lambda function faster?

Ans 1) Increasing Lambda memory also increases the amount of CPU/compute capacity allocated to the function.

Increase memory doesn't mean increasing the RAM it also increases compute memory (CPU).
We need to identify the bottleneck of application before dealing with the memory configuration.
If the performance issues is with DB queries, even if we increase the memory it may not improve the performance.


Q2) **Timeout**

Your Lambda is taking 20 seconds because an external API is very slow.

The timeout is:
```
10 seconds
```
What happens?

And does changing the timeout from 10 → 30 seconds actually make the external API faster?

Ans 2) 
A slow dependency can cause Lambda invocations to remain occupied for longer, which can increase concurrent executions and resource pressure.

As the external API is taking 20 sec to give response, the lambda function gets terminated after 10 sec and gives timeout error before receiving the response from the external API.
If we increase the timout configuration from 10 to 30 sec the function goes through and retruns the response but it doesn't mean it makes API faster but the resources wait till the API response is received for 30 sec. The drawback here is that when the new requests are triggered as it will wait for more time new resources will get created.


Q3) **Environment Variables**

Why would you use:
```
process.env.API_URL
```

instead of:
```
const API_URL = "https://production.example.com";
```

Give me two reasons.

Ans 3) We should not hardcode configuration variables, instead define in the environment variables for easy maintainability.
If there is any change in the variable we need not deploy the code, we can just make change in the enironment configurations.
Also, there can be different configuration for different environments (staging, dev, prod) so if we hardcode such variables we may face issues if forget to make necessary updates in different environments.

**The Major benefits**:
```
Configuration separated from code
          +
Different configuration per environment
```

**ChatGpt:**
```
One nuance we'll cover later: changing Lambda environment variables normally results in a function configuration update and can affect execution environments; don't think of it simply as "change a variable with zero deployment/runtime implications."
```


Q4) **Security**

Where would you prefer to keep:
```
API_URL
```

versus:
```
DATABASE_PASSWORD
```
Would you treat both exactly the same?

Why?

Ans 4) API_URL, DATABASE_PASSWORD.
I would configure the API_URL in the environment configuration while DATABASE_PASSWORD in the secrets manager.
We can configure the DATABASE_PASSWORD in the environment configuration but it will not be secured.
Passwords should be secured, it should be known to only specific group of users.
If we configure it in the environment configuration, who ever has the access to the environment conf will know the passwords.
Any secured configurations should be configured in the secrets manager.


**ChatGpt**:

Environment variables can be **encrypted at rest**, so saying "environment variables are not secure" is too absolute.

The better production statement is:

Don't treat Lambda environment variables as the preferred secret-management system for sensitive credentials. Use a dedicated secrets-management solution when appropriate.

**Memory Map**
```
Normal configuration
        ↓
Environment variables

Sensitive secrets
        ↓
Secrets Manager / Parameter Store
```

Q5) **IAM**

Your Lambda needs to read a file from S3.

Does Lambda automatically have permission to read any S3 bucket?

If not, what provides the permission?
Ans 5)  No, Lambda will not have permissions to read any S3 buckets by default. We need to attach an IAM Role that has policy attached giving permissions like Read, Delete etc for a specific S3 bucket.

Q6) **Versions & aliases**

You have:
```
Version 10 → stable
Version 11 → new release
```

and:
```
production → Version 10
```

What happens when you change:
```
production → Version 11
```

And why would aliases be useful for rollback?
Ans 6) The versions and aliases are useful duing deployments.

Suppose if we deploy the new release(version 11) to production and we find a major bug to be worked up, then we can roll back to the previous stable version (version 10).

The aiases help us identity the which version is pointing to stage/production etc.

**We'll eventually discuss how this leads to blue/green and canary deployments.**


Q7) **Architecture**

Why can Lambda architecture (x86_64 vs arm64) become important when your Node.js application uses native dependencies?
Ans 7) We need to check if the native dependencies belongs to x86_64 or arm64, if this is not configured correctly while creation of lambda it will give problems in production.

**ChatGpt**:
Especially important for Node.js packages containing native binaries.

**Mind Map**
```

Your machine
    ↓
npm install
    ↓
native module compiled for x86_64
    ↓
Lambda configured as arm64
    ↓
💥 compatibility problem
```
The exact behavior depends on how the dependency/package was built, but your architectural understanding is right.



Q8) **Senior-level architecture**

Suppose your Lambda receives a sudden spike and AWS creates:

```
100 execution environments
```

Each environment creates:

```
5 database connections
```

Approximately how many database connections could you potentially end up with?

And why could this be a serious problem for RDS?
Ans 8) Appoximately we will end up with 5 * 100 = 500 databse connections. 
This could be a serious problem for because the RDS might not support that many connections there will be some limitation which we need to consider.

I know this can be handled by concurrency, but we have stil not discussed on this in great detail.