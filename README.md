# Intoduction

In this session, you learn how to build a multiplayer, turn-based game using Amazon DynamoDB and Amazon Simple Notification Service (Amazon SNS).

Amazon DynamoDB is a fully-managed, NoSQL database that provides lightning-fast performance at any scale.

Amazon SNS is a high-volume messaging service that allows for pub/sub functionality as well as messaging directly to SMS, email, or mobile applications.

## Why use both Amazon DynamoDB and Amazon SNS for a game application?

Amazon DynamoDB and Amazon SNS are common choices for use in game applications. Both services provide high-scalability with ease of use and straight-forward pricing.

Some of the key reasons to use Amazon DynamoDB and Amazon SNS for your game application are:

• Performance: Amazon DynamoDB provides single-digit millisecond latency at any scale.

• Pay-per-use: Both Amazon DynamoDB and Amazon SNS offer pay-per-use pricing models. This eases development and ensures your bill only grows as your user base does.

• Fully-managed: Both Amazon DynamoDB and Amazon SNS do not require you to provision servers, perform upgrades, or handle failovers. All operations are managed by AWS, allowing you to focus on building your application.


## Prerequisite
An AWS Account with root priviliges

## Technologies used:

• Active AWS Account**

• Browser: AWS recommends Chrome

• Amazon DynamoDB

• Amazon SNS

• Amazon Cloud9

• Amazon Cognito

• AWS Lambda

• Amazon API Gateway

• AWS SDK for Node.js

## Table of contents

1. [Background and Setup](#1-Background-and-setup)
2. [Provision a database](#2-Provision-a-database)
3. [Set up notifications with Amazon SNS](#3-Set-up-notifications-with-Amazon-SNS)
4. [Add authentication to your application](#4-Add-authentication-to-your-application)
5. [Retrieve values of **genre** table](#5-retrieving-list-of-values)
6. [Create **movie** table](#6-creating-a-movies-table)



# 1. Background and Setup
Imagine you are building an online application where users can play Nim, a turn-based strategy game. In Nim, there are three heaps of objects. Two players alternate turns removing any number of objects from a single pile. The goal of the game is to force the other player to remove the last object.

As part of your application, you need to save the state of an existing game. You also need to notify users at various points in a game. You notify them when a user invites them to a new game, when it is their turn to play, and when a winner has been decided.

## Step 1a: Create an AWS account
Use a personal AWS account or create a new AWS account for this lab. Do not use an organizational account so that you have full access to the necessary services and do not leave behind any resources. If you do not delete the resources when you are finished, **you may incur AWS charges**.

## Step 1b: Set up your AWS Cloud9 IDE
To set up your AWS Cloud9 development environment:

Navigate to the AWS Management Console, choose Services at the top of the page, and then choose Cloud9 under Developer Tools.
- Choose Create environment.
- Type Turn-based game in the Name box. Leave the Description box empty.
- Choose Next step.
- Leave the Environment settings at their defaults to create a new t2.micro EC2 instance, which will be hibernated after 30 minutes of inactivity.
- Choose Next step.
- Review the environment name and settings, and choose Create environment. Your environment will be provisioned and prepared after several minutes.
- When the environment is ready, your IDE should open with a welcome note.


You should now see your AWS Cloud9 environment. You need to be familiar with the three areas of the AWS Cloud9 console shown in the following screenshot:

- File explorer: On the left side of the IDE, the file explorer shows a list of the files in your directory.
- File editor: On the upper right area of the IDE, the file editor is where you view and edit files that you’ve selected in the file explorer.
- Terminal: On the lower right area of the IDE, this is where you run commands to execute code samples.
![alt text](https://d1.awsstatic.com/Getting%20Started/AWS-Labs-Turn-Based-Game/turn-based-game-cloud9.a8b55e4797f1a0dc1a7f55da0cfb94065fb2bbad.png)

## Step 1c: Download the supporting code
In this, you use JavaScript to interact with your Amazon DynamoDB database and Amazon SNS. Run the following commands in your AWS Cloud9 terminal to download and unpack the module code.

```
cd ~/environment 
curl -sL http://d118jxrmrxsq90.cloudfront.net/turn-based.tar | tar -xv
```

Run the following command in your AWS Cloud9 terminal to view your directories.

```
ls
```



You should see two directories in the AWS Cloud9 file explorer:

**Application**: The application directory contains example code for the turn-based game application. This code is similar to the code you would have in your real turn-based game application backend.

**Scripts**: The scripts directory contains administrator-level scripts, such as for creating AWS resources or loading data into your database.

Run the following command in your AWS Cloud9 terminal to install the dependencies for both directories.

```
npm install --prefix scripts/ && npm install --prefix application
```
Run the following command in your AWS Cloud9 terminal to set your AWS Region in an environment file. This example uses us-east-1, but enter your AWS Region of choice to use for the lab. 

```
echo "export AWS_REGION=us-east-1" >> env.sh && source env.sh
```

You use the **env.sh** file to store environment variables of resources and other parameters you need in this lab. If you take a break during this lab and then start a new session in your AWS Cloud9 environment, be sure to reload your environment variables by executing the following command in your terminal:

```
source env.sh
```

[🏠 Back to Table of Contents](#table-of-contents)



# 2. Provision a database

In this module, you provision an Amazon DynamoDB database and learn how to use DynamoDB to store information about your turn-based game. 

Amazon DynamoDB is a fully-managed NoSQL database provided by AWS. It provides single-digit millisecond response times and near-infinite scalability. DynamoDB is used by a wide variety of applications and industries, from the Amazon.com shopping cart to Lyft’s geolocation service to a wide variety of online games.

All interaction with DynamoDB is done over HTTPS using AWS Identity and Access Management (IAM) for authentication and authorization. Usually, you use the AWS SDK for your language of choice to interact with DynamoDB. If you are using AWS compute options for your application, such as Amazon Elastic Compute Cloud (Amazon EC2) or AWS Lambda, your application can use the AWS credentials in your compute environment to make requests to DynamoDB.

## Step 2a: Provision an Amazon DynamoDB database

First, let’s provision a DynamoDB database. A database in DynamoDB is also called a table.

When creating a DynamoDB table, you need to specify the attribute(s) that will make up the primary key of your table. Each record you write to your DynamoDB table is called an item, and each item must include the primary key of your table.

DynamoDB data modeling and primary key design is an important topic. However, your game has a simple access pattern that does not require advanced data modeling.

In your application, you use DynamoDB as a simple key-value store. Each game has a *gameId* attribute that uniquely identifies each game. The *gameId* attribute is used as the primary key for your table.

In the **scripts/** directory, there is a file called **create-table.sh** that creates your DynamoDB table using the AWS Command Line Interface (AWS CLI). The contents of the file are as follows:

```
aws dynamodb create-table \
  --table-name turn-based-game \
  --attribute-definitions '[
    {
      "AttributeName": "gameId",
      "AttributeType": "S"
    }
  ]' \
  --key-schema '[
    {
      "AttributeName": "gameId",
      "KeyType": "HASH"
    }
  ]' \
  --provisioned-throughput '{
    "ReadCapacityUnits": 5,
    "WriteCapacityUnits": 5
  }'
  ```

First, it gives the table a name *turn-based-game*. Then, it declares the attributes that are used in the table’s primary key. You are using a simple primary key in this example, so you only declare a single attribute, gameId, which is of a string data type. Next, you specify your primary key schema by stating that the *gameId* attribute be used as your table’s hash key.

Finally, you specify the number of read and write capacity units you want for your table. DynamoDB does have an on-demand pricing mode where you pay for each read and write request against your table. However, the provisioned throughput mode fits within the AWS Free Tier, so you can use that here.

Create your table by running the following command in your terminal:

```
bash scripts/create-table.sh
```

You should see the following output in your terminal:

```
{
    "TableDescription": {
        "AttributeDefinitions": [
            {
                "AttributeName": "gameId",
                "AttributeType": "S"
            }
        ],
        "TableName": "turn-based-game",
        "KeySchema": [
            {
                "AttributeName": "gameId",
                "KeyType": "HASH"
            }
        ],
        "TableStatus": "CREATING",
        "CreationDateTime": 1574086642.07,
        "ProvisionedThroughput": {
            "NumberOfDecreasesToday": 0,
            "ReadCapacityUnits": 5,
            "WriteCapacityUnits": 5
        },
        "TableSizeBytes": 0,
        "ItemCount": 0,
        "TableArn": "arn:aws:dynamodb:us-east-1:955617200811:table/turn-based-game",
        "TableId": "c62cb86a-211e-4a50-a160-4a616c8f3445"
    }
}
```

## Step 2b:  Save an example game item in your table

Now that you have created your table, you can add items to your table. Each game is represented by a single item in the table.

The schema of each game item is as follows:


![alt text](https://d1.awsstatic.com/Getting%20Started/AWS-Labs-Turn-Based-Game/turn-based-game-schema.e4cd3826657521e17bd481d2063d8ecd5cf7fe48.png)

Each game includes a **GameId**, which is a unique identifier for the game. The attributes *User1* and *User2* store the usernames of the two users that are playing the game. The attributes *Heap1*, *Heap2*, and *Heap3* store the number of objects in each of the three heaps. Finally, the *LastMoveBy* attribute indicates the player who made the most recent move.

In the scripts/ directory, there is a createGame.js file that adds an example game to your table. The contents of the file are as follows:

```
const AWS = require('aws-sdk')
const documentClient = new AWS.DynamoDB.DocumentClient()

const params = {
  TableName: 'turn-based-game',
  Item: {
    gameId: '5b5ee7d8',
    user1: 'myfirstuser',
    user2: 'theseconduser',
    heap1: 5,
    heap2: 4,
    heap3: 5,
    lastMoveBy: 'myfirstuser'
  }
}

documentClient.put(params).promise()
  .then(() => console.log('Game added successfully!'))
  .catch((error) => console.log('Error adding game', error))
```

You import the AWS SDK and then create an instance of the DynamoDB Document Client. The Document Client is a higher-level abstraction over the low-level DynamoDB API and makes it easier to work with DynamoDB items. After creating the client, the script assembles the parameters for a **PutItem** API call, including the table name and the attributes on the item. Then it calls the **put()** method on the Document Client and logs out information on success or failure.

You can run the script to insert a game into your table by running the following command in your terminal:

```
node scripts/createGame.js
```

You should see the following output in your terminal:

``
Game added successfully!
``

**Note**: If you run the command too quickly, you may get an error that the table is not yet available. Wait one minute, then try the command again.

Great! You have now added a single game to your table. In the next step, you learn how to update that game item to simulate a user making a move.

## Step 2c: Update a game item in your table

Now that you have a game item in your table, you can learn how to simulate a player making a move for a game that’s in progress.

There are two ways you could handle this operation. In the first way, you retrieve the item using the **GetItem** API. Then, you update the game item in your application according to the move made by the player. Finally, you replace the existing item in DynamoDB using the **PutItem** API. Although this option works, it requires multiple requests to your DynamoDB table and risks overwriting any changes that happened between fetching and re-writing your game item.

The second way to handle this operation is to use the **UpdateItem** API call in DynamoDB. With the **UpdateItem** API, you can update your DynamoDB item in place via a single request. You specify the item you want changed, the attributes to change, and any conditions to want to assert on the item. This is the preferred way to make a change to an item because it doesn't require multiple calls to your database.

In the **scripts/** directory, there is a file called **performMove.js** which has the following contents:

```
const AWS = require('aws-sdk')
const documentClient = new AWS.DynamoDB.DocumentClient()

const performMove = async ({ gameId, user, changedHeap, changedHeapValue }) => {
  if (changedHeapValue < 0 ) {
    throw new Error('Cannot set heap value below 0')
  }
  const params = {
    TableName: 'turn-based-game',
    Key: { 
      gameId: gameId
    },
    UpdateExpression: `SET lastMoveBy = :user, ${changedHeap} = :changedHeapValue`,
    ConditionExpression: `(user1 = :user OR user2 = :user) AND lastMoveBy <> :user AND ${changedHeap} > :changedHeapValue`,
    ExpressionAttributeValues: {
      ':user': user,
      ':changedHeapValue': changedHeapValue,
    },
    ReturnValues: 'ALL_NEW'
  }
  try {
    const resp = await documentClient.update(params).promise()
    console.log('Updated game: ', resp.Attributes)
  } catch (error) {
    console.log('Error updating item: ', error.message)
  }
}

performMove({ gameId: '5b5ee7d8', user: 'theseconduser', changedHeap: 'heap1', changedHeapValue: 3 })
```

This script is a little complicated, so let’s take it step-by-step.

Like the previous script, you import the AWS SDK and create a DynamoDB Document Client instance.

Then, you define a method called **performMove**. This method is similar to an internal method that would be used in your application when a user requests to make a move. The script assembles the parameters for an **UpdateItem** API call. First, it changes two attributes on the game -- the last user to make a move, and the number of elements in the changed heap.

The UpdateItem API parameters then make some assertions about the current state of the game. The ConditionExpression is evaluated before the item is updated to confirm that the item is in the state you want. You are making the following three assertions in your condition expression:

  1. That the user requesting to perform a move is one of the two users in the game;
  2. That the current turn belongs to the user requesting to perform the move;
  3. That the heap being changed has a current value higher than the value to which is it being changed.

Finally, the parameters state a **ReturnValue** of **ALL_NEW**, which means that DynamoDB returns the entire item after its values have been updated. Once you have this, your application can evaluate the game to see if there’s a winner.

At the bottom of the file is an example of how this method is called in your application. You can execute the script with the following command:

```
node scripts/performMove.js
```

You should see the following output in your terminal:

```
Updated game:  { heap2: 4,
  heap1: 3,
  heap3: 5,
  gameId: '5b5ee7d8',
  user2: 'theseconduser',
  user1: 'myfirstuser',
  lastMoveBy: 'theseconduser' }
```

You can see that the write was successful and the game has been updated.

Try running the script again in your terminal. You should see the following output:

``
Error updating item:  The conditional request failed
``

Your conditional requests failed this time. The requesting user -- *theseconduser* -- was the most recent player to move. Further, the changed heap -- *heap1* -- already had a value of 3, meaning the user didn’t change anything. Because of this, the request was rejected.

[🏠 Back to Table of Contents](#table-of-contents)

# 3. Set up notifications with Amazon SNS

When building a turn-based game, you need some way to contact your users to notify them of important events. This includes alerting them that a new game is started, notifying them that it is their turn to play, or informing them that a game has ended.

In this module, you use Amazon Simple Notification Service (Amazon SNS) to notify your users. Amazon SNS is a fully-managed messaging system that allows for pub/sub functionality as well as direct messaging to SMS and email.

Amazon SNS has pay-per-use pricing, so you are only billed for the messages you send. You don’t need to provision throughput ahead of time.

## Step 3a: Send SMS messages via Amazon SNS

There are two different ways to use Amazon SNS. The first way is to create a topic and publish messages to it. You can then configure multiple subscribers to the topic, and each subscriber receives a copy of a message when it is published. This approach is using Amazon SNS in a pub/sub manner and is commonly used for broadcasting messages between services in a microservices architecture.

The second way to use Amazon SNS is to publish messages directly to a consumer. This method is the approach used in this module. Amazon SNS allows you to publish directly to an SMS number for simple text messaging.

In the **scripts/** directory, there is a **sendMessage.js** file. The contents of that file are as follows:


```
const AWS = require('aws-sdk')
const sns = new AWS.SNS();

const sendMessage = async ({ phoneNumber, message }) => {
  const params = {
    Message: message,
    PhoneNumber: phoneNumber
  }

  return sns.publish(params).promise()
}

sendMessage({ phoneNumber: process.env.PHONE_NUMBER, message: 'Sending a message from SNS!'})
  .then(() => console.log('Sent message successfully'))
  .catch((error) => console.log('Error sending SNS: ', error.message))
```

Like in the DynamoDB module, you are importing the AWS SDK for JavaScript in Node.js and creating a client for the service you are using.

There is a **sendMessage** function that is similar to a function you have in your application. It takes a phone number and message, and it publishes it as an SMS message.

At the bottom of the file is an example invocation of that message. You can test this out in your console.

First, export your phone number as an environment variable. You need to include the country code in the number. If you are in the United States, that country code is **1**.

Use the following command to save and export your phone number, substituting your number for the default listed:

```
echo "export PHONE_NUMBER=+15555555555" >> env.sh && source env.sh
```

Next, execute the **sendMessage.js** script by entering the following command in your terminal:

```
node scripts/sendMessage.js
```

You should see the following output in your terminal:

``
Sent message successfully
``

You should also receive a message on your phone.

![alt text](https://d1.awsstatic.com/Getting%20Started/AWS-Labs-Turn-Based-Game/turn-based-game-sms.e68686bcc577495147e4d76ea904b509078d70b5.png)

Success! You were able to send a message via Amazon SNS. You use Amazon SNS to notify your users of a number of events in your turn-based game. 

[🏠 Back to Table of Contents](#table-of-contents)

# 4 Add authentication to your application

In this module, you configure Amazon Cognito for use as the authentication provider in your application. Amazon Cognito is a fully managed authentication provider that allows for user sign-up, verification, login, and more.

Amazon Cognito has two different components: *user pools* and *identity pools*. User pools are standard user directories where users sign in through Amazon Cognito or through a third-party identity, such as Google or GitHub. After successful authentication, users receive tokens, such as access tokens or ID tokens, that can be used to access resources in the backend.

In contrast, identity pools provide a way for users to receive temporary AWS credentials for accessing AWS resources. This could be used to provide limited, direct access to an AWS Lambda function, an Amazon DynamoDB table, or other resources.

In this tutorial, you use Amazon Cognito user pools. You allow users to register via your application. After they’ve registered, they can login via a client to receive an ID token. This ID token can be passed as a header to your application to authenticate the user.

In the following steps below, you create an Amazon Cognito user pool. Then you create a client to access the user pool. Finally, you look at some example code to interact with the user pool.

## Step 4a: Create an Amazon Cognito user pool

A *user pool* is a user directory -- all user and group management happens in your pool. When creating a user pool, you need to specify rules for your user pool, including your password policy and the required attributes.

By default, Amazon Cognito includes a number of standard attributes that are not required by your user. In your application, you want the *phone_number* attribute to be a required attribute because users are notified via SMS message. Thus, when creating your user pool, mark phone_number as required.

In the **scripts/** directory, there is a file called **create-user-pool.sh.** The contents are as follows:

```
USER_POOL_ID=$(aws cognito-idp create-user-pool \
  --pool-name turn-based-users \
  --policies '
      {
        "PasswordPolicy": {
          "MinimumLength": 8,
          "RequireUppercase": true,
          "RequireLowercase": true,
          "RequireNumbers": true,
          "RequireSymbols": false
        }
      }' \
  --schema '[
      {
        "Name": "phone_number",
        "StringAttributeConstraints": {
            "MinLength": "0",
            "MaxLength": "2048"
        },
        "DeveloperOnlyAttribute": false,
        "Required": true,
        "AttributeDataType": "String",
        "Mutable": true
      }
  ]' \
  --query 'UserPool.Id' \
  --output text)

echo "User Pool created with id ${USER_POOL_ID}"
echo "export USER_POOL_ID=${USER_POOL_ID}" >> env.sh
```

This script uses the AWS Command Line Interface (AWS CLI) to create a user pool. You give your user pool a name -- *turn-based-users* -- and specify your password policies. For the password, this script requires a minimum length of eight characters, and the password must include an uppercase letter, a lowercase letter, and a number.

Additionally, you specify in your schema that the *phone_number* attribute is a required attribute for all users, as this is needed for notifying your users of important game events. 

You can create your user pool by executing the script with the following command:

```
bash scripts/create-user-pool.sh
```

You should see the following output:

```
User Pool created with id <user-pool-id>
```

In the next step, you create a client to access the user pool.

## Step 4b: Create a user pool client

Now that you have a user pool configured, you need to make a *user pool client*. A user pool client is used to call unauthenticated methods on the user pool, such as register and sign in.

The user pool client is used by your backend Node.js application. To register or sign in, a user makes an HTTP POST request to your application containing the relevant properties. Your application uses the user pool client and forwards these properties to your Amazon Cognito user pool. Then, your application returns the proper data or error message.

There is a file in the **scripts/** directory called **create-user-pool-client.sh** for creating a user pool client. The contents of the file are as follows:

```
source env.sh

CLIENT_ID=$(aws cognito-idp create-user-pool-client \
  --user-pool-id ${USER_POOL_ID} \
  --client-name turn-based-backend \
  --no-generate-secret \
  --explicit-auth-flows ADMIN_NO_SRP_AUTH \
  --query 'UserPoolClient.ClientId' \
  --output text)

echo "User Pool Client created with id ${CLIENT_ID}"
echo "export COGNITO_CLIENT_ID=${CLIENT_ID}" >> env.sh
```

The script creates a user pool client for your newly-created user pool. It uses the **ADMIN_NO_SRP_AUTH** flow, which is a flow that can be used in a server-side application. Because you’re doing a server-side flow, you don’t need a client secret used in a client-side flow, such as on a mobile device or in a single-page application.

Run the script to create the user pool client with the following command:

```
bash scripts/create-user-pool-client.sh
```

You should see the following output:

```
User Pool Client created with id <client-id>
```

In the last step of this module, you learn about the authentication code that is used in your application.

## Step 4c: Review authentication code

Now that you have created a user pool and a client to access the pool, let’s see how you use Amazon Cognito in your application.

In the **application/** directory of your project, there is a file called **auth.js**. This file contains a few authentication helpers for your application. There are four core functions to that file. Let’s review them in turn.

The first function is **createCognitoUser** and is used when registering a new user in Amazon Cognito. The function looks as follows:

```
const createCognitoUser = async (username, password, email, phoneNumber) => {
  const signUpParams = {
    ClientId: process.env.COGNITO_CLIENT_ID,
    Username: username,
    Password: password,
    UserAttributes: [
      {
        Name: 'email',
        Value: email
      },
      {
        Name: 'phone_number',
        Value: phoneNumber
      }
    ]
  }
  await cognitoidentityserviceprovider.signUp(signUpParams).promise()
  const confirmParams = {
    UserPoolId: process.env.USER_POOL_ID,
    Username: username
  }
  await cognitoidentityserviceprovider.adminConfirmSignUp(confirmParams).promise()
  return {
    username,
    email,
    phoneNumber
  }
}
```

It uses the Amazon Cognito client ID that you set in the previous step, as well as the username, password, email, and phone number given by the user, to create a new user. Additionally, you immediately confirm the user so that the user can sign in. Usually, you opt for a confirmation process that verified the user’s email and/or mobile phone number so that you have a contact method for them. That’s out of scope for this tutorial, so you can just automatically confirm new users.

The second core method is the login function that is used when registered users are authenticating. The code is as shown below:

```
const login = async (username, password) => {
  const params = {
    ClientId: process.env.COGNITO_CLIENT_ID,
    UserPoolId: process.env.USER_POOL_ID,
    AuthFlow: 'ADMIN_NO_SRP_AUTH',
    AuthParameters: {
      USERNAME: username,
      PASSWORD: password
    }
  }
  const { AuthenticationResult: { IdToken: idToken } }= await cognitoidentityserviceprovider.adminInitiateAuth(params).promise()
  return idToken
}
```

Like in the **createCognitoUser** function, you use the Client Id and the given parameters to make a call to Amazon Cognito. The **adminInitiateAuth** method for Amazon Cognito authenticates the user and return tokens if the authentication method passes. You use ID tokens for authentication, so that is what you return for successful user logins.

The third function is **fetchUserByUsername**. You use this function to fetch a user object by a given username to find the phone number for a user you need to notify. The code is as follows:

```
const fetchUserByUsername = async username => {
  const params = {
    UserPoolId: process.env.USER_POOL_ID,
    Username: username
  };
  const user = await cognitoidentityserviceprovider.adminGetUser(params).promise();
  const phoneNumber = user.UserAttributes.filter(attribute => attribute.Name === "phone_number")[0].Value;
  return {
    username,
    phoneNumber
  };
};
```

The function takes a single parameter, username. It then calls the **adminGetUser** function from the Amazon Cognito client. Then, it filters through the attributes to find the user’s phone number and returns an object with the user’s username and phone number.

Finally, there is a **verifyToken** function. The contents of this function are as follows:

```
const verifyToken = async (idToken) => {
  function getKey(header, callback){
    client.getSigningKey(header.kid, function(err, key) {
      var signingKey = key.publicKey || key.rsaPublicKey;
      callback(null, signingKey);
    });
  }

  return new Promise((res, rej) => {
    jwt.verify(idToken, getKey, {}, function(err, decoded) {
      if (err) { rej(err) }
      res(decoded)
    })
  })
}
```

This function verifies an ID token that has been passed up with a request. The ID token given by Amazon Cognito is a JSON Web Token, and the **verifyToken** function confirms that the token was signed by your trusted source and to identify the user. This function is used in endpoints that require authentication to ensure that the requesting user has access.

In subsequent modules, you use these four authentication functions in your backend application.

[🏠 Back to Table of Contents](#table-of-contents)



















