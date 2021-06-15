# turn-based-game-aws
Building a turn-based game with Amazon DynamoDB and Amazon SNS

# Background and setup
Imagine you are building an online application where users can play Nim, a turn-based strategy game. In Nim, there are three heaps of objects. Two players alternate turns removing any number of objects from a single pile. The goal of the game is to force the other player to remove the last object.

As part of your application, you need to save the state of an existing game. You also need to notify users at various points in a game. You notify them when a user invites them to a new game, when it is their turn to play, and when a winner has been decided.
