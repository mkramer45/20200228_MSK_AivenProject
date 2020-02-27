# SurfSender
Application that provides the current wave heights of 6 popular beaches in New England. This app utilizes Python to gather the data from a list of URLs & then the data is sent as messages via Apache Kafka. Ultimately, the data is stored using an Aiven PostgreSQL database. This data can be used for building a web application or alerting mechanism to make surfers in the New England area aware of when there are big waves at their favorite beaches.


# Requirements
In order to get SurfSender up & running on your local environment, you will need the following:

--Python 3.7
--Apache Kafka
--Apache Zookeeper

These items can be downloaded & installed from their respective websites. Just enter the above values in a search engine & you'll find a handful of links to choose from. It's always best to download directly from the distributor's website.

Additionally, once you have Python 3.7 running, you will need to install the following 3rd party libraries.

--BeautifulSoup
--Kafka
--lxml

To install these, open a terminal & enter the following commands:

```
pip3 install bs4
pip3 install Kafka
pip3 install lxml
```

Lastly, you will need the SurfSender package itself. Simply download it from this github repository & save it to your disk.


# Getting started

Before running any of the files in the SurfSender package, we need to first get the Zookeeper & Kafka servers running. For additional information on these items, their documentation serves as a great resource (https://kafka.apache.org/). To get both of these running, enter the following commands in a new instance of a terminal. We will first focus on Zookeeper.

With a fresh terminal window, navigate to the directory where your Apache Kafka package is stored. For example, in my case, I have it saved in my downloads directory (/Users/mike/Downloads). Once you've landed in the package's directory, run the following command to start Zookeeper:

```
>>bin/zookeeper-server-start.sh config/zookeeper.properties
```

You should now see Zookeeper running.

Next, in a similar fashion, we need to get the Kafka server running. In order to do this, we need to open a new tab in the terminal, you can do this by pressing command + T. You'll notice that upon opening the new tab, we remain in the Kafka package directory - so no need to do any navigation like we saw in the previous step.

In your new tab, run the following command to start Kafka server:

```
>>bin/kafka-server-start.sh config/server.properties
```

You should now see Kafka server running.

It's vital to keep these two tabs active while running the files from the SurfSender package, as the Kafka components are dependent on these servers running.

# Running SurfSender

Now it's time to run the files from the SurfSender package that you have downloaded. Open a new terminal and navigate to the package's directory location.

Once inside the directory, the first file that needs to be run is our Producer file named 'producer_surf_info.py'. This is a python file, so your command should look like this:

```
>>python3 producer_surf_info.py
```
In regards to Kafka protocol, this file servers as our producer file. It retrieves the surf data we want & sends the data as messages to the Kafka topic, in this case the topic name is 'surf_data_topic'.

To see our the topic populated with the messages that were sent, you can run the following command from the Kafka directory:

```
>>bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic surf_data_topic --from-beginning
```

Now that the producer has done it's job of sending messages to the topic, we will now run our consumer file in order to print the messages & store them in the Aiven PostgreSQL database.

While remaining in the SurfSender package directory in your terminal, execute the consumer file:

```
>>python3 consumer_surf_info.py
```

Upon executing this file, you should see the messages from the topic displayed in your terminal. Additionally, you are now able to query these messages in the SurfSender database table named 'surf_data' in PostgreSQL.

# Testing SurfSender

---TestCase #1: Invalid URLs---

Given the nature of the program, one of the important areas we should test is our data source. In the event that a webpage in our URL list goes down or the formatting of the contents we are scraping changes, we need to be able that our script is still able to execute in it's entirety, despite such a mishap with our data source.

In the test directory named 'surfsender_testcase1', I've updated the 'producer_surf_info.py' file to include an invalid URL ( changed 'https://magicseaweed.com/Scituate-Surf-Report/372/' to ''https://magicseaweed.com/Scituate-Su1234rf-Report/372/'. If you were to load ''https://magicseaweed.com/Scituate-Su1234rf-Report/372/' in a web browser, you will receive a 404: page not found error.

Due to the function containing our web scraping logic being wrapped with try/except clauses, we see that the script is still able to run until completion, but the data for this message is displayed as:

```
>>Scituate Wave Height: No Data Available
```
Although we aren't able to get any data, the script is still able to be executed without any fatal errors that result in the script crashing, which is what we want.


---TestCase #2: Alter message when current wave height is 'flat'---

At times, the URLs in our list will contain a value of 'flat' instead of an integer for the wave height value. We want to ensure that when such occurances happen, our data doesn't read as 'flat ft', as part of our scraping logic appends the 'ft' substring to the wave height integer value (ie '4 ft'). Having a message contain 'flat ft' looks ugly, we want it to just read 'flat', as that's how we'd describe the conditions in real life.

In the test directory named 'surfsender_testcase2', I've updated the 'producer_surf_info.py' file to NOT scrape actual wave height values from the URLs. Instead, I've hard coded all of the wave height values to 'flat', in order to simulate the instances when the URLs would contain 'flat' as the current wave height.

Upon executing the updated producer file & then the consumer file, we see that the values read as follows:

```
Nahant Wave Height: flat 
Nantasket Wave Height: flat 
Scituate Wave Height: flat 
Seabrook Wave Height: flat 
Rye Wave Height: flat 
Salisbury Wave Height: flat 
```

This is the outcome we expected for such a scenario.

# Author
This software package was created by Mike Kramer. For any questions/feedback/collaboration inquiries, please send emails mkramer789@gmail.com
