# Simple Routing: Leason 1
This repo is a companion to **Recipe 1** in the "Simple Serverless" series and future lessons will build on the tools and patterns used here.
I hope you find something here helpful, and please give this repo a star if you do. Thanks for checking it out.

This repo contains a simple but complete pattern for using decorators to map REST and GrqphQL endpoints to a functions in lambdas. 
You can use SAM and the included template.yaml file to deploy a fully functional API to AWS. 
I was careful to favor resources that are only "pay for what you use" so there should be little or no reoccurring costs for this deployment.

I also use this repo as a toolbox of tricks I've learned over the years to make developing lambdas fast and easy. 

You will
find in this repo:
- A pattern for mapping REST or GrqphQL endpoints to functions via a decorator ex: `@router.rest("GET", "/students")`
- All the infrastructure as code needed to deploy fully functional APIs via SAM which is an AWS extension of CloudFormation
- A simple script that makes it easy to iterate and debug locally
- Commands to invoke a deployed lambda and tail its logs in realtime.



# Example

### Rest

`http://myapi.com/students`

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
        Payload=bytes('{"route": "list_students"}', 'UTF-8')
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
- AWS CLI [install docs](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
- SAM CLI [install docs](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html)
- make
- An S3 Bucket for uploading the lambda deployments as defined in the `S3_BUCKET` variable in the make file.
- An AWS account with permissions to deploy Lambda, API Gateway, AppSync 
and other resources they depend on. You can always remove API Gateway or AppSync resources from template.yaml if you're only intrested in REST or GraqphQL.
- A shell configured with your AWS credentials AWS_DEFAULT_REGION, AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY... 
  [docs](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html)


# Deploy
```
git clone git@github.com:SimpleServerless/simple-routing.git
cd simple-routing
make deploy
```

# What if I only want REST or GraphQL not both?
I've tried to keep everything related to REST or GraphQL grouped together so you can easily remove one or the other.
Nothing will break if you do or do not include a decorator for REST or GrqphQL in lambda_function.py, so feel free to delete the 
decorators you don't want to use.
If you only want to deploy examples for REST or GraphQL find the resource sections in template.yaml that start with
the comments "# API Gateway (REST stuff) starts here" or "#  Appsync (GraphQL stuff) starts here" and remove every resource
in that section. You will also need to delete only the referenced resources in the "Outputs" section.


# Files Explanation
**/src**

&nbsp;&nbsp;**lambda_function.py:** Contains the lambda handler and all CRUD or business logic functions the endpoints are routed to.

&nbsp;&nbsp;**requirements.txt:** Contains a list of any dependancies that needs to be included in the deploy.

&nbsp;&nbsp;**schema.graphql:** GraphQL schema only used if grqphQL routes are declared

&nbsp;&nbsp;**utils.py:** Contains supporting functions for lambda_handler.py

&nbsp;&nbsp;**Makefile:** Make targets for deploying, testing and iterating. See [Make Targets](#make-targets) for more information.

&nbsp;&nbsp;**run_local.py:** Helper script for testing and iterating locally in a shell or IDE. 
You can run this script in an IDE to execute lambda_function.handler locally, set break points...

&nbsp;&nbsp;**template.yaml:** SAM extended CloudFormation template used to deploy resources to AWS.


# Make Targets
**clean:** Removes artifacts that are created by testing and deploying

**build:** Uses SAM to build and prepare for packaging

**package:** Uses SAM to package application for deployment

**deploy:** Uses SAM and `template.yaml` to deploy the function and supporing infrastructure to AWS.

**invoke:** Uses the AWS CLI to invoke the deployed function.

**run-local:** Uses run_local.py to execute the handler locally. This target demonstrates
how run_local.py can be used as a wrapper to run and debug the function in a shell or from an IDE.

**tail:** Uses the AWS CLI to tail the logs of the deployed function in realtime.



# Iterate in a local environment
You'll need to have your AWS credentials set up in your shell to access AWS resources like SecretsManager. [docs](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html)

The `run_local` make target demonstrates how to use the run_local.py script to iterate locally, or as something you can 
run in an IDE allowing you so set breakpoints and debug. 
