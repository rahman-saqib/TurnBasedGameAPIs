# Turn Based Game
Building a turn-based game with Amazon DynamoDB and Amazon SNS

# Prerequisite
An AWS Account with root priviliges

# Background and setup
Imagine you are building an online application where users can play Nim, a turn-based strategy game. In Nim, there are three heaps of objects. Two players alternate turns removing any number of objects from a single pile. The goal of the game is to force the other player to remove the last object.

As part of your application, you need to save the state of an existing game. You also need to notify users at various points in a game. You notify them when a user invites them to a new game, when it is their turn to play, and when a winner has been decided.

# Step 1: Create an AWS account
Use a personal AWS account or create a new AWS account for this lab. Do not use an organizational account so that you have full access to the necessary services and do not leave behind any resources. If you do not delete the resources when you are finished, **you may incur AWS charges**.

# Step 2: Set up your AWS Cloud9 IDE
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

# Step 3: Download the supporting code
In this, you use JavaScript to interact with your Amazon DynamoDB database and Amazon SNS. Run the following commands in your AWS Cloud9 terminal to download and unpack the module code.

``cd ~/environment ``<br>``
curl -sL http://d118jxrmrxsq90.cloudfront.net/turn-based.tar | tar -xv``

Run the following command in your AWS Cloud9 terminal to view your directories.

`ls`



You should see two directories in the AWS Cloud9 file explorer:

**Application**: The application directory contains example code for the turn-based game application. This code is similar to the code you would have in your real turn-based game application backend.

**Scripts**: The scripts directory contains administrator-level scripts, such as for creating AWS resources or loading data into your database.

Run the following command in your AWS Cloud9 terminal to install the dependencies for both directories.

`npm install --prefix scripts/ && npm install --prefix application`

Run the following command in your AWS Cloud9 terminal to set your AWS Region in an environment file. This example uses us-east-1, but enter your AWS Region of choice to use for the lab. 

`echo "export AWS_REGION=us-east-1" >> env.sh && source env.sh`

You use the **env.sh** file to store environment variables of resources and other parameters you need in this lab. If you take a break during this lab and then start a new session in your AWS Cloud9 environment, be sure to reload your environment variables by executing the following command in your terminal:

`source env.sh`



















