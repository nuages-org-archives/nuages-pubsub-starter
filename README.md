# Nuages PubSub Starter

Nuages.PubSub is a WebSocket services based on API Gateway WebSocket. It provides all the building block to easily deploy the backend to Lambda using Cloud Formation.

This repository is for **deployment** only. Full source code available here https://github.com/nuages-io/nuages-pubsub

### At a glance:

- Similar to Azure Web PubSub
- Include WebSocket Service + Client API
- .NET Core 6 SDK included
- Open API endpoint available (/swagger)
- Many database options are available: DynamoDB, MongoDB and MySQL. More can be added by providing additional provider trough DI.
- Use the following AWS services
  - API Gateway
  - Lambda
  - CloudFormation (CDK)
  - IAM
  - CloudWatch
  - Route 53 for custom domain (optional)
  - Certificate Manager (optional)
  - System Manager (Parameter Store, App Config, optional)
  - DynamoDB (optional)



## Prerequisites

**IMPORTANT!** You need to have CDK configured on your machine

**IMPORTANT!** You need .NET 6. 

https://docs.aws.amazon.com/cdk/v2/guide/work-with.html#work-with-prerequisites



## Getting Started

Everything is ready to be used as-is. All you need to do is deploy using the CDK from you machine. 

**The easy way:**

1. Install the CDK
1. git clone https://github.com/nuages-io/nuages-pubsub-starter
2. cdk deploy 

That's it. Your web socket services and API are ready to go.

**The harder way:**

Some benefits require some work. Follow the steps in the sections below to customize the stack.



## Customizing the deployment stack

You can customize most of the settings to fit you needs. Just follow the instructions below.


### 1. Clone this Github repository on your machine

```
git clone https://github.com/nuages-io/nuages-pubsub-starter
```
### 2. Setup CDK deployment options

The following context variable can be set to customize the deployment using the CDK. (See https://docs.aws.amazon.com/cdk/v2/guide/context.html for detail about context).

#### Infrastructure options

By default, the stack will be deployed with the current configuration

- Auto generated URL for WebSocket Endpoint and API Endpoint
- DynamoDB as internal storage
- Auto-generated API key
- Default values for Issuer, Audience and Secret.

You can customize this behavior by setting the following options.

| Name                    | Description                        | Instructions                                                 |
| ----------------------- | ---------------------------------- | ------------------------------------------------------------ |
| WebSocketDomain         | WebSocket Endpoint Domain Name     | DO NOT CREATE YOURSELF. Route53 recordset will be created with the stack |
| WebSocketCertificateArn | Certificate for WebSocket Endpoint | Required if WebSocketDomain is assigned.                     |
| ApiDomain               | API Endpoint Domain Name           | DO NOT CREATE YOURSELF. Route53 recordset will be created with the stack |
| ApiCertificateArn       | Certificate for API Endpoint       | Required if APIDomain is assigned                            |
| ApiApiKey               | API Gateway API Key                | Will be autogenerated if not provided                        |



#### Database options

| Name                 | Description                                  | Instructions                                                 |
| -------------------- | -------------------------------------------- | ------------------------------------------------------------ |
| DataStorage          | Database storage used. Default: DynamoDB     | DynamoDB, MySQL or MongoDB                                   |
| DataConnectionString | Connection string to connect to the database | May be a connection string, an AWS Secrets Arn or null if provided from application settings. |

**IMPORTANT!!!** 

- Only DynamoDB tables will be deployed as part of the stack. For other database engines, you need to do this deploy on your own and provide the connection string.
- It is recommended to setup a DatabaseProxy for MySQL (https://docs.aws.amazon.com/lambda/latest/dg/configuration-database.html)



#### Database Proxy Options

The following options applies if you choose to use a DatabaseProxy for your MySQL database. Both the VPC and DatabaseProxy must exists and will NOT be created during deployment. All options are required.

**IMPORTANT!** Adding the lambda to a VPC will prevent outbound access to the Internet if the VPC does not provided a public subnet and an Internet Gateway. Nuages PubSub does not require a Internet connection, so this might not be an issue unless you customize the code and add services that connect to the Internet.

| Name                       | Description                | Instructions                                                 |
| -------------------------- | -------------------------- | ------------------------------------------------------------ |
|                       |               |                 |
| VpcId                   | Id of the VPC.                                      | Must be the VPC of the database you connect to |
| SecurityGroupId         | Security group Id | The Lambdas will be added to this security group. This security group must have access to the database proxy and the database |
| DatabaseProxyUser          | Database user name         | Must be the same as in the connection string                 |
| DatabaseProxyArn      | ARN of the Database Proxy | Required. See Proxy properties. |
| DatabaseProxyEndpoint | Proxy Endpoint            | Required. See Proxy properties. |
| DatabaseProxyName     | Name of the proxy         | Required. See Proxy properties. |



#### Authentication settings

The following settings can be set at deployment time. You may also want to set the values using standard application settings (see following section).

| Name          | Description    | Instructions                                             |
| ------------- | -------------- | -------------------------------------------------------- |
|               |                |                                                          |
| Auth_Issuer   | Token issuer   | Ex: MyCompany or https://mycompany.com                   |
| Auth_Audience | Token audience | Ex: MyAudience                                           |
| Auth_Secret   | Token secret   | Ex. Oiwebwbehj439857948fekhekrht435 (any value you want) |

The following settings applies when you want to use External Authentication 

| Name                              | Description                                  | Instructions                                                 |
| --------------------------------- | -------------------------------------------- | ------------------------------------------------------------ |
|                                   |                                              |                                                              |
| ExternalAuth_Enabled              | Enabled when True                            |                                                              |
| ExternalAuth_ValidAudiences       | List of valid audiences                      | Comma separated                                              |
| ExternalAuth_ValidIssuers         | List of valid isseurs URL                    | Must be URLs. Comma separated                                |
| ExternalAuth_DisableSslCheck      | Disable SSL certificate validation when True | Might be useful if you are using ngrok                       |
| ExternalAuth_JsonWebKeySetUrlPath | Json Web Token Key Set Path.                 | Open OpenId configuration page for your issuer. Ex: https://myissuer.com/.well-known/openid-configuration . Look for the wks_url value. Enter only the Path element. |
| ExternalAuth_Roles                | Roles asisgned to connections                | Default to "SendMessageToGroup JoinOrLeaveGroup"             |



### 3. Application settings

Runtime option applies to **Nuages.PubSub.Starter.API** and **Nuages.PubSub.Starter.WebSocket** projects.

You have many options to set application settings:

- Change appsettings.json 
- Set the values directly in the code
- Set options from System Manager Parameter Store 
- Set options from System Manager AppConfig (recommended)


The expected appsettings.json configuration format is as followed.

```json
{
  "Nuages": {    
    "PubSub": {
      "Auth":
      {
        "Issuer": "https://issuer.example.com",
        "Audience": "YourAudience",
        "Secret": "YourSecret"
      },
      "ExternalAuth" :
      {
        "Enabled" : false,
        "ValidAudiences" : null,
        "ValidIssuers" : null,
        "JsonWebKeySetUrlPath" : null,
        "DisableSslCheck" : false,
        "Roles" : "SendMessageToGroup JoinOrLeaveGroup"
      }
    }    
  }
}
```



**System Manager AppConfig**

By default, AppConfig will be loaded using the following identifiers. Those values can be changed in appsettings.json.

- ApplicationId : NuagesPubSubStarter
- ConfigProfileId: Shared
- EnvironmentId: Prod

You can create this configuration in AppConfig using the same identifier and add your application settings file as a hosted configuration.



**System Manager Parameter Store**

By default, parameter store values are loaded from path root /NuagesPubSubStarter. That value can be changed in appsettings.json.

The expected Parameter Store values should have the following key format

*{$StackName}/{$Keys}*

Ex. For the "Nuages:PubSub:Auth:Issuer" settings above, the key would be "{$StackName}/Nuages/PubSub/Auth/Issuer"

More info on this can be found here https://github.com/aws/aws-dotnet-extensions-configuration

### 4. Deploy

**Deploy locally**

Run this command at the solution root

```
cdk deploy
```

**Continuous integration and delivery (CI/CD)** (using GitHub)

More information available here : https://docs.aws.amazon.com/cdk/v2/guide/cdk_pipeline.html

You can us this method to have the application redeployed when there is a push in your repository. 

**IMPORTANT!** This deployment method will only work if you host the source code in your own <u>forked repository</u>.

Add --pipeline to the "app" element in cdk.json. The command should look like

```
"app": "sh build.sh;dotnet run --project src/Nuages.PubSub.Cdk.Starter/Nuages.PubSub.Starter.Cdk.csproj --pipeline"
```

Add CDK configuration to your application settings (applicationsettings.json or AppConfig or Parameter Store)

```json
{
  "Settings" : 
  {
    "CDKPipeline": {
      "GitHubRepository": "OWNER/REPO",
      "GithubToken": "{your-github-token-with-required-roles}",
      "NotificationTargetArn" : "~your-notification-arn-chatbot-or-sns"
  	}
  }
}

```

The Github token must have repository access and must be able to create webhooks.

Run this command at the solution root

```
cdk deploy
```

The CDK pileline will be deployed. Then, the pipeline will deploy the stack. 

You can also be notified for pipeline events. Just add a notification target Arn to NotificationTargetArn in the configuration.

The stack will automatically redeployed when a Push occurs on the Github repository.

## External authentication

You may want to use external token instead of token generated by the API. 

To do so, you will have to call UseExternalAuthRoute and provide some settings.

```csharp
 serviceCollection
     .AddPubSubService()
     .AddPubSubLambdaRoutes(configuration)
      .UseExternalAuthRoute(options =>
      {
         //The following settings can also be set in the appsettings or AppConfig or Parameter Store
         options.Enabled = true;
            
         //Set options here or set application settings
         options.ValidAudiences = "...",
         options.ValidIssuers = "...",
         options.JsonWebKeySetUrlPath" : ".well-known/jwks" //With OpenIdDict
         //Identity Server
         // options.JsonWebKeySetUrlPath" : ".well-known/openid-configuration/jwks"
         //If you use Auth0
         //options.JsonWebKeySetUrlPath" : ".well-known/jwks.json"  
         //You can find the Url of your authority by navigating to Url with path .well-known/openid-configuration
         options.DisableSslCheck = false; //May be usefull if using ngrok
     });
      
```

Sample configuration section.

```json
{
  "Nuages":
  {
    "PubSub":
    {
      "ExternalAuth" :
      {
        "Enabled": true,
        "ValidAudiences" : "YourAudience",
        "ValidIssuers" : "https://issuer.example.com",
        "JsonWebKeySetUrlPath" : ".well-known/jwks",
        "DisableSslCheck" : false
      }
    }
  }
}
```

# How to use Nuages PubSub

Very quick intructions here...I will be working on something more complete soon.

- Services URL are available in the Output tab of the Cloud Formation Nuages PubSub stack page

  - One endpoint for WebSocket
  - One endpoint for the API
    

- You can get the WebSocket URL using the **API Endpoint** getclienturi method


The API endpoint is secured usingh the API Key you provided on deploy (or it has been automatically assign). You can retrieve the value from the API Keys section in the AWS Application Gateway page.

The API key value should be added to the **x-api-key** request header

```
curl --location --request GET 'https://websocket-api.nuages.org/api/auth/getclienturi \
?userId=martin \
&roles=SendMessageToGroup \
&roles=JoinOrLeaveGroup \
&hub=Hub1' \
--header 'x-api-key: IFyTB9Cqad4MuML20j2sG5XfeFvx4LwZ1pgCURS9'
```

Once you have the WebSocket URL, you can use it to connect using any standard WebSocket client

https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-how-to-call-websocket-api-wscat.html

```
wscat -c "wss://gwaxs2uio9.execute-api.ca-central-1.amazonaws.com/prod?hub=Hub1&access_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJtYXJ0aW4iLCJyb2xlcyI6WyJTZW5kTWVzc2FnZVRvR3JvdXAiLCJKb2luT3JMZWF2ZUdyb3VwIl0sIm5iZiI6MTY0NjY1Njk3OCwiZXhwIjoxNjQ2NzQzMzc4LCJpYXQiOjE2NDY2NTY5NzgsImlzcyI6Imh0dHBzOi8vcHVic3ViLm51YWdlcy5vcmciLCJhdWQiOiJOdWFnZXNQdWJTdWIifQ.RojpjqsmGI-e5uoDSssx_3D8L8MIGuRjUQxTV3NZP0E"
```

### Echo

```json
{ "type":"echo" }
```

### Join

```json
{ "type":"join","dataType" : "json","group" : "group1"}
```

### Leave

```json
{ "type":"leave", "dataType" : "json", "group" : "group1"}
```

### SendMessage

```json
{"type":"send", "dataType" : "json", "group" : "group1", "data": { "my_message" : "message sent to group" }}
```

Additional capabilities are offered using the API Endpoint.

## API / SDK

API Documentaton is available here https://app.swaggerhub.com/apis-docs/Nuages/NuagesPubSub/1.0.0#/

C# SDK is available here https://www.nuget.org/packages/Nuages.PubSub.API.Sdk/

