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
3. [Create table **genre** with GraphQL](#3-create-table-genre-with-graphql)
4. [Insert data in **genre**  with GraphQL](#4-insert-data-in-the-table-with-graphql)
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






















