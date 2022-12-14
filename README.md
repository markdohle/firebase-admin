# firebase-admin

## MIT xPro Week 26 - Authentication of Express Routes with Firebase Tokens

<img id="tokens" src="image004.png" width='400' onclick="Run()" style="position:absolute"> </img>

### Authentication

Authentication determines a user’s identity. Firebase makes it easy for you to integrate authentication into your application. You can use pre-built authentication solutions for logging in with your email and password or more robust solutions that allow for authentication with a Gmail or Facebook account.

### Routing

Routing is a little different from authentication, in that routing is an internal process that allows access to different areas of a web page through API endpoints. Now, routing and authentication work together so that only authenticated users can access certain routes (this is also known as authorization), while others cannot. 

### How Firebase Works

If I log in to an app that uses Google Firebase services, the server gives my browser an access token. This token is passed back to the server with any new routing request I make as long as I am logged in, and then the server looks at my token, sees if it can authenticate that particular route that I want to access with it, and then will let me access that resource. Admin users are authenticated to access any resource. 

### Instructions:

In this exercise, you will authenticate certain routes with Firebase. Download the starter files Links to an external site.of a back end and a front end form that you will need to a) configure to your own firebase instance and b) authenticate several express routes for different users of the app.

- Take the token presented to browser client
- Add the token to the HTTP header
- Pass the token to the back end
- Use a Firebase package to verify the token

1. Goto Firebase Project Settings. Select Service accounts. Generate a new private key object.

```
//=============Private Key Example==============
{
  "type": "service_account",
  "project_id": "kogs-94e74",
  "private_key_id": "5773e02e412e41ed99ec4c0af682ab306fd2dbfa",
  "private_key": "-----BEGIN PRIVATE KEY-----\xxx\n-----END PRIVATE KEY-----\n",
  "client_email": "firebase-adminsdk-yfi86@kogs-94e74.iam.gserviceaccount.com",
  "client_id": "108406486084031070012",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/firebase-adminsdk-yfi86%40kogs-94e74.iam.gserviceaccount.com"
}
````

2. Create an admin.js for the pk

```
const admin = require('firebase-admin');

// firebase service account pk
const type = "service_account";
const project_id = "fkogs-94e74";
const private_key_id = "5773e02e412e41ed99ec4c0af682ab306fd2dbfa";
const private_key = "-----BEGIN PRIVATE KEY-----\xxx\n-----END PRIVATE KEY-----\n";
const client_email = "irebase-adminsdk-yfi86@kogs-94e74.iam.gserviceaccount.com";
const client_id = "108406486084031070012";
const auth_uri = "https://accounts.google.com/o/oauth2/auth";
const token_uri = "https://oauth2.googleapis.com/token";
const auth_provider_x509_cert_url = "https://www.googleapis.com/oauth2/v1/certs";
const client_x509_cert_url = "https://www.googleapis.com/robot/v1/metadata/x509/firebase-adminsdk-ujt4h%40fir-practice-c5dc2.iam.gserviceaccount.com";


// credential grants access to Firebase services
admin.initializeApp({
  credential: admin.credential.cert({
      type,
      project_id,
      private_key_id,
      private_key:
        private_key.replace(/\\n/g,'\n'),
      client_email,
      client_id,
      auth_uri,
      token_uri,
      auth_provider_x509_cert_url,
      client_x509_cert_url
  }),
});

module.exports = admin;
```

3. Create Express Server with index.js.

```
const express = require('express');
const app     = express();
const admin   = require('./admin');

// used to serve static files from public directory
app.use(express.static('public'));

app.get('/open', (req,res) => res.send('Open Route!'));

// verify token at the root route
app.get('/auth', function(req,res){
    // read token from header
    const idToken = req.headers.authorization
    console.log('header:', idToken);

    // verify token
    admin.auth().verifyIdToken(idToken)
        .then(function(decodedToken) {
            console.log('decodedToken:',decodedToken);
            res.send('Authentication Sucess!');
        }).catch(function(error) {
            console.log('error:', error);
            res.send('Authentication Fail!');
        });
})

app.listen(3000, () => {
    console.log('Running on port: 3000');
})
```

4. Create Node App and install dependencies

```npm init -y```

```npm install express```

5. Create a Public Folder with the login.js and login.html

```
<!DOCTYPE html>
<html>
<head>
    <style>.card{background-color:gainsboro;padding:10px;display:inline-block;margin: 10px;height:100px;width:250px;}</style>
    <!-- The core Firebase JS SDK is always required and must be listed first -->
    <script src="https://www.gstatic.com/firebasejs/7.24.0/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/7.24.0/firebase-auth.js"></script>    
</head>

<!-- login/logout box -->
<div class="card">
    <input id="email" type="email" placeholder="Email"><br>
    <input id="password" type="password" placeholder="Password"><br><br>
    <button id="login">Login</button>
    <button id="signup">SignUp</button>
    <button id="logout" style="display:none;">Logout</button>
    <div id="loginMsg"></div>
</div><br>

<!-- call open/secure route box -->
<div class="card">
    <button onclick="callOpenRoute()">Call Open Route</button>
    <button onclick="callAuthRoute()">Call Auth Route</button>
    <div id="routeMsg"></div>    
</div>

<script src="login.js"></script>

</html>
```

```
// Your web app's Firebase configuration
    // For Firebase JS SDK v7.20.0 and later, measurementId is optional
    var firebaseConfig = {
        apiKey: "AIzaSyCF6sUFJpZ2SztqKDR-exYXC-TOGP07XyU",
        authDomain: "fir-practice-c5dc2.firebaseapp.com",
        databaseURL: "https://fir-practice-c5dc2-default-rtdb.firebaseio.com",
        projectId: "fir-practice-c5dc2",
        storageBucket: "fir-practice-c5dc2.appspot.com",
        messagingSenderId: "150296159467",
        appId: "1:150296159467:web:586573c70a870c355a4aae"
    };
    // Initialize Firebase
    firebase.initializeApp(firebaseConfig);

    // get elements
    const email    = document.getElementById('email');
    const password = document.getElementById('password');
    const login    = document.getElementById('login');
    const signup   = document.getElementById('signup');
    const logout   = document.getElementById('logout');
    const loginMsg = document.getElementById('loginMsg');
    const routeMsg = document.getElementById('routeMsg');

    // login
    login.addEventListener('click', e => {
        const auth  = firebase.auth();      
        const promise = auth.signInWithEmailAndPassword(email.value, password.value);
        promise.catch(e => console.log(e.message));
    });

    // signup
    signup.addEventListener('click', e => {
        const auth  = firebase.auth();
        const promise = auth.createUserWithEmailAndPassword(email.value,password.value);
        promise.catch(e => console.log(e.message));
    });

    // logout
    logout.addEventListener('click', e => {
        firebase.auth().signOut();
    });

    // login state
    firebase.auth().onAuthStateChanged(firebaseUser => {
        if(firebaseUser){
            console.log(firebaseUser);
            logout.style.display = 'inline';
            login.style.display  = 'none';
            signup.style.display = 'none';
            loginMsg.innerHTML   = 'Authentication Success!';
        }
        else{
            console.log('User is not logged in');
            logout.style.display = 'none';          
            login.style.display  = 'inline';
            signup.style.display = 'inline';
            loginMsg.innerHTML   = 'Not Authenticated!';
        }
    });

    function callOpenRoute(){
        (async () => {
            let response = await fetch('/open');
            let text     = await response.text();
            console.log('response.text:', response);
            routeMsg.innerHTML = text;
        })();
    }

    function callAuthRoute(){
        // call server with token
        firebase.auth().currentUser.getIdToken()
            .then(idToken => {
                console.log('idToken:', idToken);

                (async () => {
                    let response = await fetch('/auth', {
                        method: 'GET',
                        headers: {
                            'Authorization': idToken
                        }
                    });
                    let text = await response.text();
                    console.log('response:', response);
                    routeMsg.innerHTML = text;
                })();

            }).catch(e => console.log('e:',e));
    }    
```
