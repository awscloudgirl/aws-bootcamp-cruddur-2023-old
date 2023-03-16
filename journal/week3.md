# Week 3 — Decentralized Authentication

# Setup Cognito User Group

## In AWS Console:

• Go to Cognito

• Click on `User Pools`

• Click `Create User Pool`

• Check `Cognito User Pool` `Username` `Email`

• Click `Next`

• Check `No MFA`

• Click `Next`

• Scroll down to the section `Required Attributes` click on the dropdown and select `name`

• Click `Next`

• Select `Send email with Cognito` 

• Click `Next`

• Enter `cruddur-user-pool` in `User pool name`

• Scroll down to `Initial app client` and in `App client name` enter `cruddur`

• Click `Next`

• Scroll down and click `Create user pool`

# Install AWS Amplify

## In the terminal:

```bash
cd frontend-react-js`
npm i aws-amplify --save
```

## Add to `app.js`:

```javascript
import { Amplify } from 'aws-amplify';

Amplify.configure({
  "AWS_PROJECT_REGION": process.env.REACT_AWS_PROJECT_REGION,
  "aws_cognito_region": process.env.REACT_APP_AWS_COGNITO_REGION,
  "aws_user_pools_id": process.env.REACT_APP_AWS_USER_POOLS_ID,
  "aws_user_pools_web_client_id": process.env.REACT_APP_CLIENT_ID,
  "oauth": {},
  Auth: {
    // We are not using an Identity Pool
    // identityPoolId: process.env.REACT_APP_IDENTITY_POOL_ID, // REQUIRED - Amazon Cognito Identity Pool ID
    region: process.env.REACT_AWS_PROJECT_REGION,           // REQUIRED - Amazon Cognito Region
    userPoolId: process.env.REACT_APP_AWS_USER_POOLS_ID,         // OPTIONAL - Amazon Cognito User Pool ID
    userPoolWebClientId: process.env.REACT_APP_AWS_USER_POOLS_WEB_CLIENT_ID,   // OPTIONAL - Amazon Cognito Web Client ID (26-char alphanumeric string)
  }
});
```

## Add Env Vars to `docker-compose.yml`:

```yaml
      REACT_APP_AWS_PROJECT_REGION: "${AWS_DEFAULT_REGION}"
      REACT_APP_AWS_COGNITO_REGION: "${AWS_DEFAULT_REGION}"
      REACT_APP_AWS_USER_POOLS_ID: ""
      REACT_APP_CLIENT_ID: ""
```
User pool ID and Client ID found under the "cruddur" User pool in AWS console.

## Add conditional components to `HomeFeedPage.js`:

```javascript
import { Auth } from 'aws-amplify';

// check if we are authenicated
const checkAuth = async () => {
  Auth.currentAuthenticatedUser({
    // Optional, By default is false. 
    // If set to true, this call will send a 
    // request to Cognito to get the latest user data
    bypassCache: false 
  })
  .then((user) => {
    console.log('user',user);
    return Auth.currentAuthenticatedUser()
  }).then((cognito_user) => {
      setUser({
        display_name: cognito_user.attributes.name,
        handle: cognito_user.attributes.preferred_username
      })
  })
  .catch((err) => console.log(err));
};
```

## Update `ProfileInfo.js`:

```javascript
import { Auth } from 'aws-amplify';

const signOut = async () => {
  try {
      await Auth.signOut({ global: true });
      window.location.href = "/"
  } catch (error) {
      console.log('error signing out: ', error);
  }
}
```



