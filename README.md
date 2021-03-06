# lambda-routing
A Simple but complete pattern for using decorators to map REST and GrqphQL endpoints to separate functions
within the same lambda. You can use SAM and the template.yaml file to deploy a fully functional API to AWS.  I was careful to favor resources
that are only "pay for what you use" so there should be little or no reoccurring costs for this deployment.

# Example

### Rest

`http://myapi.com/`

Routes to:
```
@router.rest("GET", "/students")
def list_students(args: dict) -> dict:
    ...
    return student_list
```

### GraphQL
```
query listStudents{
  listStudents
  {
    studentId
    firstName
    lastName
  }
}
```
Routes to:
```
@router.graphql("Query", "listStudents")
def list_students(args: dict) -> dict:
    ...
    return student_list
```

### Direct Invoke
```
client = boto3.client('lambda')
response = client.invoke(
        FunctionName='simple-serverless-routing-dev',
        InvocationType='RequestResponse',
        LogType='None',
        Payload=bytes(request_payload, 'UTF-8')
    )
```
Routes to:
```
@router.direct("list_students")
def list_students(args: dict) -> dict:
    ...
    return student_list
```

# Requirements

- Python 3.8
- Pip 3
- AWS cli url here
- SAM cli url here
- make
- An AWS account with permissions to deploy Lambda, API Gateway, AppSync 
and other resources they depend on. You can always remove API Gateway or AppSync if you're only intrested in REST or GraqphQL.
- A shell configured with your AWS credentials AWS_DEFAULT_REGION, AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY...
doc url here


# Deploy
```
git clone git@github.com:SimpleServerless/simple-routing.git
cd simple-routing.git
make deploy
```

# What if I only want REST or GraphQL not both?
I've tried to keep everything related to REST or GraphQL grouped together so you can easily remove one or the other.
Nothing will break if you do or do not include a decorator for REST or GrqphQL in lambda_function, so feel free to delete the 
decorators you don't want to use.
If you only want to deploy examples for REST or GraphQL find the resource sections in template.yaml that start with
the comments "# API Gateway (REST stuff) starts here" or "#  Appsync (GraphQL stuff) starts here" and remove every resource
in that section. You will also need to delete only the relevant items in the "Outputs" section.


# Files Explanation
**/src**

&nbsp;&nbsp;**lambda_function:** Contains the lambda handler and all functions the endpoints are routed to.

&nbsp;&nbsp;**requirements.txt:** Contains a list of any dependancies that needed to be included in the deploy.

&nbsp;&nbsp;**schema.graphql:** GraphQL schema only used if grqphQL routes are declared**

&nbsp;&nbsp;**utils.py:** Contains supporting functions for lambda_handler.py**

&nbsp;&nbsp;**Makefile:** Make targets for deploying, testing and iterating. See Make Targets for more infofmation.**

&nbsp;&nbsp;**run_local.py:** Helper script for testing and iterating locally in a shell or IDE.**

&nbsp;&nbsp;**template.yaml:** SAM (CloudFormation) template used to deploy resources to AWS.**


# Make Targets
**build:** Uses SAM to build and prepare for packaging

**package:** Uses SAM to package application for deployment

**clean:** Removes artifacts that are created by testing and deploying

**deploy:** Uses SAM and `template.yaml` to deploy the function and supporing infrastructure to AWS.

**invoke:** Uses the AWS CLI to invoke the deployed function.

**run-local:** Uses run_local.py to execute the handler locally. This can be used to quickly iterate and demonstrate
how run_local.py can be used as a cood to run and debug the function from an IDE.

**tail:** Uses the AWS CLI to tail the logs of the deployed function in realtime.



# Iterate in a local environment
You'll need to have your AWS credentials set up in your shell to access AWS resources like SecretsManager.
doc url here

The `run_local` make target demonstrates how to use the run_local.py script to iterate locally, or as something you can 
run in an IDE allowing you so set breakpoints and debug. 
