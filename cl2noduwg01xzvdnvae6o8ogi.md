## How to Send Push Notifications from Backend Server using Firebase Cloud Messaging

# Introduction
Have you ever wondered or got into a situation where you have created an entire app and now all you have to do is send notifications to the users for example someone has liked your photo, new order received or if you are a foodie like me, how do you get notifications about at what stage your food is at, is it getting prepared, is it out for delivery yet?

I was in one of these situations for a very long time. I created a 3-way app, the only thing missing was I was unable to send notifications because of which I never thought my app was complete. 

Why was I stuck on it for so long? Did I never hear about Firebase Cloud Messaging(FCM)? Well I kind of did. Firebase is quite popular for it's not so good documentation and I wasn't able to find any good article/ tutorial to fully understand how the Notifications works on Android.

In this article I will explain everything and will provide you with further links for a better understanding so make sure to read it till the very end. 

# Let's get started

What you are seeing below is the 3-tier architecture of most apps in the market. Agree that most apps in the market have quite complex architecture but it is a good starting point for any app and you can then scale it according to your requirements.


![3-teir architecture.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650655055371/NXvaev-OI.png)

Now let's add Firebase Cloud Messaging to this architecture - 


![3-teir architecture with firebase.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650700855511/7lvRDefeh.png)

This is how our new architecture will look like after adding the FCM.

# What is Firebase Cloud Messaging (FCM)

> Firebase Cloud Messaging (FCM) provides a reliable and battery-efficient connection between your server and devices that allows you to deliver and receive messages and notifications on iOS, Android, and the web at no cost.

In simple words, Firebase Cloud Messaging allows you to - 

- Send notifications to Android, iOS and Web devices
- Free of cost
- Send notification to a specific user (Important)

# Prerequisite

1. Client App with Firebase initialized (could be any android app, iOS app, web app or Flutter app)
2. FCM integration with your app for displaying notifications in open state, background state & terminated state. If you don't have this click on this [link](https://www.youtube.com/watch?v=p7aIZ3aEi2w) and follow the tutorial step by step.
3. A RESTful API written in any language preferably in Python, NodeJS or Go.

I will use a Flutter app and NodeJS for the backend for the sake of this tutorial and I am assuming that you have all the prerequisites and are able to send a general notification from FCM console like shown below.

![prerequisite.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1650706394678/KntTL_y9L.gif)

# Aim
Our main aim is essentially to somehow trigger Firebase to send a notification when a particular route is requested from our API and send custom payload.

What Firebase does is it calls a device id which is a unique id for every device where your app is installed and through that id, we can send notification to that particular device.
We can obtain this device id by just adding small code on the client side.

```  
  String? token = await FirebaseMessaging.instance.getToken();
  print("Token = " + token.toString());
``` 

Just add the above two lines in the main function of Flutter code and the main function should look like this - 


```
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  LocalNotificationService.initialize();
  await Firebase.initializeApp();
  await FirebaseMessaging.instance.requestPermission();
  FirebaseMessaging.onBackgroundMessage(backgroundHandler);
  SystemChrome.setPreferredOrientations(
      [DeviceOrientation.portraitUp, DeviceOrientation.portraitDown]);
  
 // The statement below will fetch the device token and 
 // store it in the token variable.
  String? token = await FirebaseMessaging.instance.getToken();
 // The statement below will print the device token id
  print("Token = " + token.toString());

  runApp(const MyApp());
}
``` 
Now that we have the device id what should we do about it? Let's first understand what device id is.
>  Device ID/Token is a unique key for the **app-device** combination which is issued by the Apple or Google push notification gateways.

It's important to focus that the id is made up of combination of app and device. It's not the raw device token. We need to store this device id somewhere and associate it with the user it belongs to so when we need to send the user a notification we can just search the device id of that user and send notification to that device id.

This can be performed in multiple ways, either we can have our own database and store everything on our own or we can use Firebase Firestore to store the user and it's device id. One thing to make sure is that a user can have the app installed on various devices so when we store the data we need to make sure we store in 1-N relationship. 

Let's jump into Backend and start the fun stuff.

# Backend

For backend we will be using NodeJS and express to create a REST API. Essentially, when the user first opens the app or log into his account, we need to store the device token with his account on the database. We can achieve this by creating a simple endpoint which will take device id and user unique id as data and will store it on the database. 

Let's start doing it then, First we need to initialize a space where we can work. Navigate to the place where you work and run the command below.


```
npm init -y | npm i express cors firebase-admin
``` 
It should give you an output like shown below.


![Screenshot (15).png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651343474092/UHDo3ii5q.png)

The command you just ran did 2 things,

1. Initialized a new node project with all the default settings (you can change them anytime you want in the package.json)
2. Install three packages - Express, Cors & Firebase Admin

Now this is not a NodeJS tutorial so I won't be explaining every line of code but we will simply create 2 routes

1. A route for saving the device id/token to our backend server
2. Another route that will mimic making a new pizza order (I am a little hungry)

Now follow the steps below - 

**Step 1 - **  Create a new file and name it ```index.js```

**Step 2 - **  Open ```index.js``` and paste the code given below


```
const express = require('express')
const cors = require('cors')

const app = express()

app.use(cors())
app.use(express.json())

let userData = []

app.post('/saveinfo', (req, res) => {
    userData.push({ "userid": req.body.userid, "deviceid": req.body.deviceid })
    res.json({ "message": "Device Code Saved" }).status(201)
    console.table(userData)
})

app.post('/orderpizza', (req, res) => {
    let item = userData.find(item => item.userid == req.body.userid);
    console.log(item)
    item !== undefined 
        ? res.json({ "message": "Order made" }).status(201) 
        : res.json({ "message": "404 Not Found" }).status(404)
})

app.listen(3000, () => console.log(`http://localhost:3000/`))
``` 
This above code takes care of all the boilerplate code we need. This creates a server where there are 2 routes available

1. **/saveinfo - ** which will take ```userid``` & ```deviceid``` as body arguments and save the device id to the server. Currently for the sake of this tutorial we have used a simple array of objects but in real world app, it will be saved on a relational or non-relational database.
2. **/orderpizza - ** which will take ```userid``` as body argument and find if that user exists in the database or not. If yes, find that user's device id and send them a notification that the pizza order has been made. 

Now let's connect Firebase to our backend server. There are several ways to do that but why overcomplicate things when we can do it easily. Go to the project settings of your firebase project and navigate to the "Service Accounts". You can find it using the picture below. 


![Opera Snapshot_2022-05-01_174512_console.firebase.google.com.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651407443546/dpMJM1a1x.png)

Then click on "Generate private key"


![Opera Snapshot_2022-05-01_175019_console.firebase.google.com.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651407674084/Wc8Q7vXSQ.png)

Then finally click on "Generate Key"


![Opera Snapshot_2022-05-01_175131_console.firebase.google.com.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651407741299/cms1jD0GQ.png)

and a json will will be downloaded. Include that file into the root folder of the working directory of the server and make sure to not change it's name.

For the next step, go back to the firebase and you copy the code as per the language you are using. I am using Node so I will pick that and copy paste the code and update the path to my private key. The ```index.js``` file should look like this now - 


```
const express = require('express')
const cors = require('cors')

// --> Newly added code from here 
var admin = require("firebase-admin");
var serviceAccount = require("path/to/serviceAccountKey.json");
admin.initializeApp({
  credential: admin.credential.cert(serviceAccount)
});// <--Newly added code to here 

const app = express()

app.use(cors())
app.use(express.json())

app.get('/', (req, res) => {
    res.send("Welcome to the REST API")
})

let userData = []

app.post('/saveinfo', (req, res) => {
    userData.push({ "userid": req.body.userid, "deviceid": req.body.deviceid })
    res.json({ "message": "Device Code Saved" }).status(201)
    console.table(userData)
})

app.post('/orderpizza', (req, res) => {
    let item = userData.find(item => item.userid == req.body.userid);
    console.log(item)
    item !== undefined 
        ? res.json({ "message": "Order made" }).status(201) 
        : res.json({ "message": "404 Not Found" }).status(404)
})

app.listen(3000, () => console.log(`http://localhost:3000/`))
``` 

Now all that's left for the backend is to send the actual notification which is superrrrr easy and only requires a few lines of code. Modify the ```/orderpizza``` route to the given below and you are finally done with the backend.


```
app.post('/orderpizza', (req, res) => {
    let item = userData.find(item => item.userid == req.body.userid);
    console.log(item)
    if(item !== undefined) {
        res.json({ "message": "Order made" }).status(201)

        const registrationToken = item['deviceid'];
        const message = {
            "notification": {
                "title": "Pizza Ordered",
                "body": "Your Pizza in on the way. Yippie!"
            },
            token: registrationToken
          };
    
        admin.messaging().send(message)
            .then((response) => {
                console.log('Successfully sent message:', response);
            })
            .catch((error) => {
                console.log('Error sending message:', error);
            });
    }
    else {
        res.json({ "message": "404 Not Found" }).status(404)
    }    
})
``` 

The above modified code when sent a ```POST``` request will check if the user exists or not if yes then order a new pizza and after ordering will send a notification to the user's device id notifying him about the pizza order. Run the server using 

```
node index.js
```

There are a lot of configurations you can do in the message and play with different functions but this is the bare minimum you need to send the notification to a particular device id/token. To know more about it read the official Firebase Cloud Messaging Documentation [here](https://firebase.google.com/docs/cloud-messaging/send-message)

With this we conclude our backend and let's come back to the final part of Flutter to make everything work.

# Let's Flutter

To keep the code for this tutorial simple and clean I removed everything from UI of the main screen of the app and just have a single button which says "ORDER PIZZA". It looks like this.


![Screenshot (16).png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651422622656/I5mj6e9Yl.png align="left")

We need to find a way to register device id to the backend server. Our backend part is ready to go but what will we do from flutter end? Ideally we will have an instance of shared preferences checking if the user is logged in first time or if he is already logged in and just opening the app, with that check we can send a post request to our ```/saveinfo``` with user id and device id/token.

For this tutorial we will not be checking with the shared preferences and send the POST request in the main function itself and use the user id as ```544```. To do that inside main function, add this following piece of code below the print statement of token id.


```
  var res = await http.post(Uri.parse('http://10.0.2.2:3000/saveinfo'),
      headers: <String, String>{
        'Content-Type': 'application/json; charset=UTF-8',
      },
      body: jsonEncode(<String, String>{"userid": "544", "deviceid": "$token"}));
``` 
This will save the device id on the backend server.

All that's left for us is to make that yummy pizza order. Inside the ```onPressed``` method of the text button, make POST request to route ```/orderpizza``` and pass the same user id.

```
 onPressed: () async {
            var res =
                await http.post(Uri.parse('http://10.0.2.2:3000/orderpizza'),
                    headers: <String, String>{
                      'Content-Type': 'application/json; charset=UTF-8',
                    },
                    body: jsonEncode(<String, String>{"userid": "544"}));

            print(res.statusCode);
            print(res.body);
          },
```
Save all the code and run the app. If everything is correct then you should see a notification few seconds after you press on "ORDER PIZZA". Let's try it.


![ezgif-5-76e19f1f47.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1651425004148/9rh8h2AS4.gif align="left")

Woohoo!! Everything works and we have successfully implemented Firebase Cloud Messaging in our Flutter App and hooked it up with backend using NodeJS.

If you think I left out something or if you have any queries let me know and I will I will definitely clear that up as well. 

Thank You for reading and don't forget to share it with your friends. 