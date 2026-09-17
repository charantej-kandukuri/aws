🧠 Knowledge Check — Topic 1

Q1) What does serverless mean in AWS Lambda? Does it mean there are no servers?

Ans 1) No, underneath AWS manages everthing at serverslevel, we dont have to bother about it. we just focus on function logic.

Q2) What is the difference between a Lambda function and a Lambda invocation?

Ans 2) lamda function - it is a hanlder function with 2 paramteres event and context.

lambda invocation - it is executing the handler code by creating the execution environment if not exists. A lambda function can be invoked by resources like s3, http, SQS etc.

Q3) What is an execution environment?
Ans 3) An execution environment is something that gets created when a lambda function is invoked, this includes:
- execution environment
- runtime environment
- initialization 
- handler

Q4) Suppose Lambda receives 100 requests at roughly the same time. Can all 100 requests necessarily execute inside one Node.js process?

Why?

Ans 4) Since all requests are recieved at roughly the same time, the execution environment will persist and be able to handle the requests.

But if there is a possibility that all 100 requets are not handled by single execution enivronment then multiple execution environements might get created to handle requests as each request is a separate invocation.

Q5) Consider:

let count = 0;

export const handler = async () => {
    count++;
    console.log(count);
};

Can we use count as reliable persistent application state?

Why or why not?

Ans 5) No, we can't not use count as reliable persistent application state, because the count will reset upon cold starts and hence we should consider persistant storages like s3, rds, redis etc.

Q6 — Think like a senior developer

Why would we put something like a database connection outside the handler instead of creating it inside the handler every time?

Ans 6) we should put database connection outside the handler for the purpose of reusability. The handler can just reuse the existing db connection in the same execution environment.