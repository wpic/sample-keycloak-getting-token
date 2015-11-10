##							Obtain Token and Invoke Service

Keycloak allows you to make direct REST invocations to obtain an access token. To use it you must also have registered a valid Client to use as the "client_id" for this grant request.

1.So first we need to create a client that can be used to obtain the token. Go to the Keycloak admin console and create a new client. Give the Client ID a name and select public for access type. 

Then in Settings, set "Direct Grants Only" on, and This switch is for clients that only use the Direct Access Grant protocol to obtain access tokens. Under Web Origins URIs enter you keycloak root (for example, https://auth.wpic-tools.com/auth)

As we are going to manually obtain a token and invoke the service let's increase the lifespan of tokens slightly. In production access tokens should have a relatively low timeout, ideally less than 5 minutes. To increase the timeout go to the Keycloak admin console again. This time click on Realm Settings then on Tokens. Change the value of Access Token Lifespan to 15 minutes. That should give us plenty of time to obtain a token and invoke the service before it expires.

In order to get a token, The REST URL to invoke on is /{keycloak-root}/realms/{realm-name}/protocol/openid-connect/token

Now we're ready to get our first token using CURL. To do this run:

```bash
RESULT=`curl --data "grant_type=password&client_id=curl&username=yourusername&password=*****" https://auth.wpic-tools.com/auth/realms/wpic/protocol/openid-connect/token`
```

Basically what we are doing here is invoking Keycloaks OpenID Connect token endpoint with grant type set to password which is the Resource Owner Credentials flow that allows swapping a username and a password for a token.
Take a look at the result by running:
echo $RESULT

(For public client's, the POST invocation requires form parameters that contain the username, credentials, and client_id of your application. For example:

    POST /auth/realms/demo/protocol/openid-connect/token
    Content-Type: application/x-www-form-urlencoded

    username=bburke&password=geheim&client_id=curl&grant_type=password)

2.The result is a JSON document that contains a number of properties. There's only one we need for now though so we need to parse this output to retrieve only the value we want. To do this run:

```bash
TOKEN=`echo $RESULT | sed 's/.*access_token":"//g' | sed 's/".*//g'`
```

This command uses sed to strip out everything before and after the value of the access token property.

3.Now that we have the token we can invoke the secured service. To do this run:
curl https://squirrel-staging.wpic-tools.com/rest/tasks -H "Authorization: bearer $TOKEN"
