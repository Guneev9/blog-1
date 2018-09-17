+++
# Date this page was created.
date = "2018-05-02"

# Project title.
title = "Twitter Engine"

# Project summary to display on homepage.
summary = "An elixir-based server-side simulation of Twitter, capable of simulating around 5000 users on a Macbook Pro."

# Optional image to display on homepage (relative to `static/img/` folder).
#image_preview = "twitter-simulator.png"

# Tags: can be used for filtering projects.
# Example: `tags = ["machine-learning", "deep-learning"]`
tags = ["distributed-systems", "elixir", "twitter", "phoenix"]

# Optional external URL for project (replaces project detail page).
#external_link = "https://github.com/krohitm/Twitter-Simulator"

# Does the project detail page use math formatting?
math = false

# Optional featured image (relative to `static/img/` folder).
[header]
#image = "twitter-simulator.png"
#caption = "A screenshot of the UI"

+++

## **Overview** 

The application can be seen at https://github.com/krohitm/Twitter-Simulator.

In this project, we implemented a Twitter Clone and a client tester/simulator.

As part I of this project, we built an engine that (in [part II](https://didyousaydata.xyz/project/twitter-simulator/)) will be paired up with WebSockets to provide full functionality. Specific things done in this project are: 

* Implemented a Twitter like engine with the following functionality:

1. Register account.
2. Send tweet. Tweets can have hashtags and mentions.
3. Subscribe to user's tweets.
4. Re-tweets (so that your subscribers get an interesting tweet you got by other means).
5. Allow querying tweets subscribed to, tweets with specific hashtags, tweets in which the user is mentioned (my mentions).
6. If the user is connected, deliver the above types of tweets live (without querying)

* Implemented a tester/simulator to test the above:

8. Simulated more than 5000 users on a 64-bit i5 Macbook Pro with 8GB RAM.
9. Simulated periods of live connection and disconnection for users
10. Simulated a Zipf distribution on the number of subscribers. For accounts with a lot of subscribers, increased the number of tweets. Made some of these messages re-tweets.

## **Client-Server**

A client upon connection is able to write tweets as well as search tweets with hashtags and mentions. Tweets contain randomly generated hashtags. Every tweet contains a random mention of another user, chosen by the simulator. A client can also log itself off and upon login receive its tweets. Also, clients are able to retweet.

The client part (send/receive tweets) and the engine (distribute tweets) are separate processes. We used multiple independent client processes that simulate thousands of clients and a single engine process.

## **Simulator**

* The Simulator is a separate process and dictates what requests clients can send. 
* The simulator initially spawns a given number(n) of clients. These clients register themselves with the server by sending requests.
* Once the clients are registered, the simulator gives each client the number of users that will follow that user. These numbers are decided using Zipf distribution. In our application, the most subscribed user will have n-1 followers, the second most subscribed has (n-1)/2 no. of followers, and so on, as per the Zipf distribution.
* Once the registrations and subscriptions have been done, the clients, independently, start sending tweets. The rate of tweets sent by each user depends upon the no. of subscribers the client has. The more the number of subscribers of a user, the lower the interval in sending tweets.
* Each user, after a certain number of tweets, sends a random request to the server which may be searching for tweets of all users it has subscribed to, searching for certain hashtags, searching for its mentions, or retweeting a tweet. After the random request, the user continues the same cycle.
* Each user, after a certain number of tweets, logs out of the system for x seconds. During this time, the user doesn’t send out any requests. Once reconnected, the user asks server for the tweets of the users it has subscribed to.

## **Running the Project**
* Run epmd -daemon
* Build the application using mix escript.build
* Run the server using the command: ./project server
* On the same machine, run the simulator using the command: ./project simulator <number of users>
  where <number of users> is the number of users you want to simulate. Example: ./project simulator 10000

## **Performance** 
The tests were run on a 64-bit i5 machine with 8GB RAM. 
![image](https://user-images.githubusercontent.com/10449636/34135026-205a432a-e42c-11e7-901b-92f75b20b2bb.png)

## **Database performance** 
Databases are ETS tables. These tables are public to all the server processes (not the client processes, since client runs on a different node).

1. Read concurrency - search requests are handled by the 1000 concurrent read actors. 
	This has been enabled when we create the ETS tables for tweets.
2. Write concurrency - this flag has also been set to true to enable 2 write actors to write at the same time.
These flags allow us to take advantage or Elixir’s concurrency and handle more database reads.

## **Architecture and Notes**
The Server has been designed to distribute the load of incoming tweet requests.  Requests coming to the server can be of the following form: 

1. Write a tweet and send it to followers 
2. Search a particular tweet with certain properties 

The server is an Erlang Node which is connected to client and has the PID of the client processes which are running on a different node.

The server maintains data about the clients in the form of ETS tables.
Actor on Server: 

1. Read Actor:
At present we have 1000 actors that receive a read request from clients. The server distributes read/search requests to one of these actors.

2. Write Actor:
We have kept only 2 write actors at the moment that do the job of writing data to the respective database once a tweet request has been received.

## **Further Updates**

This project was [further updated](https://didyousaydata.xyz/project/twitter-simulator/) to implement a WebSocket interface using Phoenix web framework.