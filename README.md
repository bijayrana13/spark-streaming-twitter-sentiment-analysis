<H1>Sentiment analysis on streaming twitter data in real-time using Spark Structured Streaming & Python </H1>

This project is a good start for those want to start learning Spark Structure Streaming in Python. <br>

Spark Structured Streaming/ Spark Streaming (why we chose the one and not the other?) 


<b> Input data:</b> Live tweets with a preselected keyword <br>
<b>Main model:</b> Data preprocessing and apply sentiment analysis on the tweets <br>
<b>Output:</b> A parquet file with all the tweets and their sentiment analysis scores (polarity and subjectivity) <br>

<img align="center"  width="90%" src="https://github.com/stamatelou/twitter_sentiment_analysis/blob/master/architecture.png">

We use Python version 3.7.6 and Spark version 2.4.7. We should be cautious on the versions that we use because different versions of Spark require a different version of Python. 

## Main Libraries
<b> tweepy:</b> interact with the Twitter Streaming API and create a live data streaming pipeline with Twitter <br>
<b> pyspark: </b>preprocess the twitter data (Python's Spark library) <br>
<b> textblob:</b> apply sentiment analysis on the twitter text data <br>


This project consists of 3 parts: <br>
## Part 1: Stream tweets from the Twitter Streaming API using tweepy (twitter_connection.py) <br>

<b> Step 1: </b> Setup necessary packages <br>

```
import tweepy
from tweepy import OAuthHandler
from tweepy import Stream
from tweepy.streaming import StreamListener
import socket
import json
```
Tweepy library is necessary for connecting to the Twitter API and building the data streaming pipeline. We import its classes; StreamListener and Stream for building the stream and OAuthHandler for authenticating on Twitter. We import the socket module to create a communication channel between our local machine and the Twitter API and the json module to handle data of JSON objects.

<b> Step 2: </b> Insert your credentials  <br>
```
consumer_key='hidden'
consumer_secret='hidden'
access_token ='hidden'
access_secret='hidden'
```

Use your developer credentials. If you do not have them yet, you will need to register on [Twitter for a developer account](https://developer.twitter.com/en/apply-for-access) and the request your credentials. 

<b> Step 3: </b> Create a StreamListener instance <br>
```
class TweetsListener(StreamListener):
  # tweet object listens for the tweets
  def __init__(self, csocket):
      self.client_socket = csocket
  def on_data(self, data):
    try:  
        msg = json.loads( data )
        print("new message")
        # if tweet is longer than 140 characters
        if "extended_tweet" in msg:
          # add at the end "end_of_tweet" to facilitate preprocessing
          self.client_socket.send(str(msg['extended_tweet']['full_text'] +"end_of_tweet").encode('utf-8'))         
          print(msg['extended_tweet']['full_text'])
        else:
          # add at the end "end_of_tweet" to facilitate preprocessing
          self.client_socket.send(str(msg['text']+"end_of_tweet").encode('utf-8'))
          print(msg['text'])
        return True
    except BaseException as e:
        print("Error on_data: %s" % str(e))
    return True
  def on_error(self, status):
    print(status)
    return True
```
The class TweetsListener represents a StreamListener instance, which contains one tweet per time. When we will activate the Stream, it creates the instance. The class consists of 3 methods; the on_data, the on_error, and the init. <br>
The <b>on_data</b> method of the TweetsListener receives all messages and defines which data to extract for each tweet from the Twitter Streaming API. Some examples could be the main message,comments,and hashtags. In our case we want to extract the text of the tweet. If we request the ['text'] field from each tweet, we will only receive the messages that are shorter than 140 characters. To always receive the full message, we need first to check if the tweet is longer than 140 charachetrs. If it is, we extract the ['extended_tweet']['full_text'] and if it is not we extract as before the ['text'] field. <br>
The <b>__init__</b> method initializes the socket of the Twitter Streaming API and the <b>on_error</b> method make sure that the stream works.

<b> Step 4: </b> Sent data from Twitter <br>
```
def sendData(c_socket, keyword):
  print('start sending data from client - Twitter to server - local machine')
  # authentication based on the credentials
  auth = OAuthHandler(consumer_key, consumer_secret)
  auth.set_access_token(access_token, access_secret)
  # start sending data from the Streaming API 
  twitter_stream = Stream(auth, TweetsListener(c_socket))
  twitter_stream.filter(track = keyword, languages=["en"])
```
To start getting data from the Twitter API, we first authenticate the connection with the API based on the personal credentials.
Step 4: Start streaming tweet data objects with a user-defined keyword and language.
Step 5: Retrieve the text of each tweet 




<b> Step 5: </b> ---- <br>
Create a listening socket in the local machine (server) with a predefined local IP address and a port.
Step 2: Listen for a connection client in a IP address and port on the client side of the connection.

The user selects locally a keyword and gets back live streaming tweets that include this keyword
```
if __name__ == "__main__":
    # server (local machine) creates listening socket
    s = socket.socket()
    host = "0.0.0.0"    
    port = 5555
    s.bind((host, port))
    print('socket is ready')
    # server (local machine) listens for connections
    s.listen(4)
    print('socket is listening')
    # return the socket and the address on the other side of the connection (client side)
    c_socket, addr = s.accept()
    print("Received request from: " + str(addr))
    # select here the keyword for the tweet data
    sendData(c_socket, keyword = ['piano'])
```
## Part 2: Preprocess the tweets using pyspark (Spark Structure Streaming)<br>

## Part 3: Apply sentiment analysis using textblob <br>



