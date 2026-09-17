# AWS IAM

## IAM Users vs IAM Groups
In AWS 

IAM Groups  = Teams

IAM Users = Team members

### IAM User: 
A unique identity (with username and password) that represents a person or application.

### IAM Group:
A collection of users. You attach policies(permissions) to the Group and every user in that group inherits those permissions.

### Best practices:
You should never attach permissions directly to a single user. Always put user into a group and attach permissions to the group.
