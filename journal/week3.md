# Week 3 — Decentralized Authentication

# Required Homework

### Video Review

* Watched: [Week 3 - Live Streamed Video – Decentralized Authentication](https://www.youtube.com/live/9obl7rVgzJw)
* Watched: [Week 3 - Cognito Custom Pages](https://youtu.be/T4X4yIzejTc)
* Watched: [Week 3 - Cognito – JWT Server Side Verify](https://youtu.be/d079jccoG-M)
* Watched: [Week 3 - Exploring JWTs](https://youtu.be/nJjbI4BbasU)
* Watched: [Week 3 - Improving UI Contrast and Implementing CSS Variables for Theming](https://youtu.be/m9V4SmJWoJU)
* Watched: [Week 3 - Security Considerations - Decentralized Authentication](https://youtu.be/tEJIeII66pY)
* 
# #1 Live Session 
## AWS Cognito
In this live session, I first created a UserPool in AWS Cognito.

**Steps to setup UserPool in AWS Cognito**
- Login to your AWS Console
- Check your region in which you want to use your service. I personally prefer `us-east-1` region as in that region most of the services works well.
- Search for **Cognito** service and you will find **UserPool** tab in your left side panel.
- After clicking on **UserPool** -> **Create UserPool**.
- You will be displayed with a **Authentication providers** page where I chose **Username** and **Email** for Cognito user pool sign-in options -> click **Next**
- Password Policy I kept it as **Cognito Default**.
- Under Multi-factor authentication -> I selected **No MFA** -> **Next**
- In User account recovery -> checkbox **Email only** -> **Next**
- Under Required attributes -> I selected **Name** and **preferred username** -> **Next**
- Then I chose **Send email with Cognito** for first time -> **Next**
- After that you will be asked to give your User Pool Name , I gave it as **crddur-user-pool** -> under Initial app client I kept it as **Public client** -> enter app client name **(eg: cruddur)** -> **Next**
- You will get a chance to verify all the filled details and then click on **Create User Pool**, your usepool is being created.

## Gitpod Code Working 
I installed AWS Amplify as it is development platform and provides you set of pre-built UI components and Libraries. 
```
npm i aws-amplify --save
```
After installing this I found `"aws-amplify": "^5.0.16",` in my frontend-react-js directory's `package.json` file.

**Note: make sure you are running these commands in your `frontend-react-js` directory.**

### Configure Amplify
I added this code in `app.js` of frontend-react-js directory.
```js
import { Amplify } from 'aws-amplify';

Amplify.configure({
  "AWS_PROJECT_REGION": process.env.REACT_AWS_PROJECT_REGION,
  "aws_cognito_identity_pool_id": process.env.REACT_APP_AWS_COGNITO_IDENTITY_POOL_ID,
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

In the above code set these below env vars in `docker-compose.yml`.
```js
REACT_APP_AWS_PROJECT_REGION= ""
REACT_APP_AWS_COGNITO_IDENTITY_POOL_ID= ""
REACT_APP_AWS_COGNITO_REGION= ""
REACT_APP_AWS_USER_POOLS_ID= ""
REACT_APP_CLIENT_ID= ""
```
### Then to check the **Authentication Process** I added this code in my `HomeFeedPage.js`
```js
import { Auth } from 'aws-amplify';

// set a state
const [user, setUser] = React.useState(null);

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

// check when the page loads if we are authenicated
React.useEffect(()=>{
  loadData();
  checkAuth();
}, [])
```
### To render two React components: `DesktopNavigation` and `DesktopSidebar`, passing some properties to each of them.
```js
<DesktopNavigation user={user} active={'home'} setPopped={setPopped} />
<DesktopSidebar user={user} />
```
### Then added this code in `DesktopNavigation.js` which helps you to check whether you are logged in or not by passing the`user` to `ProfileInfo`.
```js
import './DesktopNavigation.css';
import {ReactComponent as Logo} from './svg/logo.svg';
import DesktopNavigationLink from '../components/DesktopNavigationLink';
import CrudButton from '../components/CrudButton';
import ProfileInfo from '../components/ProfileInfo';

export default function DesktopNavigation(props) {

  let button;
  let profile;
  let notificationsLink;
  let messagesLink;
  let profileLink;
  if (props.user) {
    button = <CrudButton setPopped={props.setPopped} />;
    profile = <ProfileInfo user={props.user} />;
    notificationsLink = <DesktopNavigationLink 
      url="/notifications" 
      name="Notifications" 
      handle="notifications" 
      active={props.active} />;
    messagesLink = <DesktopNavigationLink 
      url="/messages"
      name="Messages"
      handle="messages" 
      active={props.active} />
    profileLink = <DesktopNavigationLink 
      url="/@andrewbrown" 
      name="Profile"
      handle="profile"
      active={props.active} />
  }

  return (
    <nav>
      <Logo className='logo' />
      <DesktopNavigationLink url="/" 
        name="Home"
        handle="home"
        active={props.active} />
      {notificationsLink}
      {messagesLink}
      {profileLink}
      <DesktopNavigationLink url="/#" 
        name="More" 
        handle="more"
        active={props.active} />
      {button}
      {profile}
    </nav>
  );
}
```

### In `ProfileInfo.js`

This code defines a function called `signOut` that uses the `Auth` object from the `aws-amplify` library to sign out the currently authenticated user from an AWS Amplify application.
```js
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
Overall, this code provides a simple and straightforward way to sign out a user from an AWS Amplify application by using the `Auth` object from the `aws-amplify` library.

### Signin Page, Signout Page and Confirmation Page
Added code to handle Errors like if the username or email is wrong then it should display a error message.
**Signin Page**
```js
import { Auth } from 'aws-amplify';

const [cognitoErrors, setCognitoErrors] = React.useState('');

const onsubmit = async (event) => {
  setCognitoErrors('')
  event.preventDefault();
  try {
    Auth.signIn(username, password)
      .then(user => {
        localStorage.setItem("access_token", user.signInUserSession.accessToken.jwtToken)
        window.location.href = "/"
      })
      .catch(err => { console.log('Error!', err) });
  } catch (error) {
    if (error.code == 'UserNotConfirmedException') {
      window.location.href = "/confirm"
    }
    setCognitoErrors(error.message)
  }
  return false
}

let errors;
if (cognitoErrors){
  errors = <div className='errors'>{cognitoErrors}</div>;
}

// just before submit component
{errors}
```

**Signout Page**
```js
import { Auth } from 'aws-amplify';

const [cognitoErrors, setCognitoErrors] = React.useState('');

const onsubmit = async (event) => {
  event.preventDefault();
  setCognitoErrors('')
  try {
      const { user } = await Auth.signUp({
        username: email,
        password: password,
        attributes: {
            name: name,
            email: email,
            preferred_username: username,
        },
        autoSignIn: { // optional - enables auto sign in after user is confirmed
            enabled: true,
        }
      });
      console.log(user);
      window.location.href = `/confirm?email=${email}`
  } catch (error) {
      console.log(error);
      setCognitoErrors(error.message)
  }
  return false
}

let errors;
if (cognitoErrors){
  errors = <div className='errors'>{cognitoErrors}</div>;
}

//before submit component
{errors}
```
**Confirmation Page**
```js
const resend_code = async (event) => {
  setCognitoErrors('')
  try {
    await Auth.resendSignUp(email);
    console.log('code resent successfully');
    setCodeSent(true)
  } catch (err) {
    // does not return a code
    // does cognito always return english
    // for this to be an okay match?
    console.log(err)
    if (err.message == 'Username cannot be empty'){
      setCognitoErrors("You need to provide an email in order to send Resend Activiation Code")   
    } else if (err.message == "Username/client id combination not found."){
      setCognitoErrors("Email is invalid or cannot be found.")   
    }
  }
}

const onsubmit = async (event) => {
  event.preventDefault();
  setCognitoErrors('')
  try {
    await Auth.confirmSignUp(email, code);
    window.location.href = "/"
  } catch (error) {
    setCognitoErrors(error.message)
  }
  return false
}
```
In the above code I have added the try catch block and used `setCognitoErrors("Email is invalid or cannot be found.") ` to catch the error.

### Recovery Page
```js
const onsubmit_confirm_code = async (event) => {
  event.preventDefault();
  setCognitoErrors('')
  if (password == passwordAgain){
    Auth.forgotPasswordSubmit(username, code, password)
    .then((data) => setFormState('success'))
    .catch((err) => setCognitoErrors(err.message) );
  } else {
    setCognitoErrors('Passwords do not match')
  }
  return false
}
```
Added the above code to check password macthing 

**This is how it displayed the error after entering wrong email/password**

![week-3](https://user-images.githubusercontent.com/57486368/226545822-29ac1a07-c2c2-471f-a91a-e569f8852759.jpg)

## #2 Implemented Server side verification using JWT
In this task I have learnt how to code for verification, authentication and authorization purpose. I have used **JWT(JSON Web Token)** because then, I will not have to go to any other external resource. Andrew, the organizer has decided to keep it simple, small, readable, maintainable and with no complexity. So he chose JWT. I was been instructed to create a user token. Which helped to fetch user data when the user logs in and if user logs out then for that the token was unset. 

## #3 Security Consideration Video by Ashish Rajan.
Understood the concept of Authentication and Authoriazation. 
- **SAML (Security Assertion Markup Language)** : You need to have a single point of entry into any application. [uses face ID]
- **Open ID Connect** : Allows yyou to use your social credentials like Google credentials, LinkedIn, Facebook and so on.. (ONLY Authentication, NO Authorization).
#### For Authorization

- **O Auth** : Usually tag team together with Open ID.

***Real Life Example***

When we post on WhatsApp Status it asks us to whether wnt to share on Facebook as well ?. This workflow essentially is the O Auth and Open ID workflow.
Exchange of token happens at the end.

## #4 AWS Cognito Concept
Understood the concept of AWS Cognito : Amazon Cognito is your service which allows authentication with users that it stores locally in amazon account.
Basically, it is a **user directory** with context of AWS.
**There are 2 types of Amazon Cognito**
- **Cognito User Pool** : Granting access to Application. 
- **Cognito Identity Pool** : Granting access to Amazon services.

## #5 Changed the UI of the Cruddur Application by implementing some CSS.

## #6 Troubleshooting
This week was more of coding and the more you code the more errors you will come across. So I was facing this **Type Error** after adding that code line to unset token `localStorage.removeItem(access_token)` in my `profileInfo.js`file and when I viewed logs it showed **KeyError: keys**

![](https://user-images.githubusercontent.com/115455157/224488089-67f99565-c8ce-4fb6-863c-7adc6432fc20.jpg)

![](https://user-images.githubusercontent.com/115455157/224488210-78459475-73f4-4905-b7af-d6b2ea79e950.jpg)

### Solution
After viewing logs and repeatedly restarting my environment I found that there was a spelling mistake. Instead of `app.logger.debug(e)` I had written it as `app.logger.deug(e)`. So finally after correcting that spelling mistake everyting was running well. 


## Summary

| Homework      | Completed     | Not Completed  |
| ------------- |:-------------:| -----:|
| Watched Week 3 - Live Streamed Video   | ✔ |  |
| Watched Ashish's Week 3 - Decenteralized Authentication | ✔      |   
| Setup Cognito User Pool|✔      |   |
| Implement Custom Signin, Signup page| ✔      |   |
| Implement Custom Confirmation Page| ✔   |   |
| Implement Custom Recovery Page |✔      |   |
| Verify JWT token server side | ✔      |   |




















