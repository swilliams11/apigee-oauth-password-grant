# OAuth 2.0 Password Grant Sample Proxy


This proxy implements the password grant.  You must make changes to the proxy as outlined below.


1. There is a service callout named ServiceCallout.Okta must be changed so that it calls out to Okta using the [Okta API](http://developer.okta.com/docs/api/resources/authn.html).
  * [Okta Authentication API](http://developer.okta.com/docs/api/resources/authn.html#authentication-operations)

2. The ExtractVariables.Okta request must be updated to extract the variables from the Okta response.

3. Modify the ExtractVariables.TokenRequest policy so that extracts the appropriate values from password grant payload.
The current implementation is shown below; however, if you don't need the other parameters, such as the enterpriseid, then  you can remove it.

```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<ExtractVariables name="ExtractVariables.TokenRequest" continueOnError="true">
    <Source>request</Source>
    <JSONPayload>
        <Variable name="grantType" type="string">
            <JSONPath>$.grant_type</JSONPath>
        </Variable>
        <Variable name="username" type="string">
            <JSONPath>$.username</JSONPath>
        </Variable>
        <Variable name="password" type="string">
            <JSONPath>$.password</JSONPath>
        </Variable>
        <Variable name="enterpriseid" type="integer">
            <JSONPath>$.enterpriseid</JSONPath>
        </Variable>
    </JSONPayload>
    <IgnoreUnresolvedVariables>true</IgnoreUnresolvedVariables>
    <VariablePrefix>flow.request</VariablePrefix>
</ExtractVariables>
```


## Sample Request
Once you deploy the proxy and you make the changes above, then send the following request.  

```
curl -X POST \
-H 'Content-Type: application/json' \
-H 'Authorization: Basic {Base64encode(clientid:secret)' \
-d '{"grant_type":"password", "username":"the-user-name", "password":"the-users-password", "enterpriseid":"enterpriseid"}' \
https://{hostname}/oauth2-password/token -d ""
```

## Other Items

An important step is to validate that the service callout was successful before you generate an access token.

```
<Step>
    <Name>RaiseFault.InvalidUsernamePassword</Name>
    <Condition>flow.login.result != "success"</Condition>
</Step>
```

## Fault Handling
This proxy does not have any fault handling included, so it should be implemented by including a Fault Rule on the
proxy and target endpoints.
