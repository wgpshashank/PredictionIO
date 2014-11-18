---
layout: docs
title: Recommendation Quick Start
---

# Quick Start - Recommendation Engine Template

An engine template is a basic skeleton of an engine. PredictionIO's Recommendation Engine Template (/templates/scala-parallel-recommendation) has integrated **Apache Spark MLlib**'s Collaborative Filtering algorithm by default.
 You can customize it easily to fit your specific needs.

We are going to show you how to create your own classification engine for production use based on this template.

## Install PredictionIO

First you need to [install PredictionIO {{site.pio_version}}]({{site.baseurl}}/install/)

Let's say you have installed PredictionIO at */home/yourname/predictionio/*.
For convenience, add PredictionIO's binary command path to your PATH, i.e. /home/yourname/predictionio/bin:

```
$ PATH=$PATH:/home/yourname/predictionio/bin; export PATH
```

and please make sure that PredictionIO EventServer, which collects data, is running:

```
$ pio eventserver
```


## Create a Sample App

Your engine will process data of an 'app'. Let's create a sample app called "MyApp" now:

```
$ pio app new MyApp
```

You should find the following in the console output:

```
...
2014-11-12 18:24:22,904 INFO  tools.Console$ - Initialized Event Store for this app ID: 2.
2014-11-12 18:24:22,920 INFO  tools.Console$ - Created new app:
2014-11-12 18:24:22,921 INFO  tools.Console$ -         Name: MyApp
2014-11-12 18:24:22,921 INFO  tools.Console$ -           ID: 1
2014-11-12 18:24:22,921 INFO  tools.Console$ - Access Key: dZN8zAX2SwEmxHN27RGR7va3XFJ3bB7qHTECf3GVL4T5ECnOErRQp5mt8rcdhmzU
2014-11-12 18:24:22,922 INFO  client.HConnectionManager$HConnectionImplementation - Closing master protocol: MasterService
2014-11-12 18:24:22,922 INFO  client.HConnectionManager$HConnectionImplementation - Closing zookeeper sessionid=0x149a2c3cf910011
2014-11-12 18:24:22,923 INFO  zookeeper.ZooKeeper - Session: 0x149a2c3cf910011 closed
2014-11-12 18:24:22,923 INFO  zookeeper.ClientCnxn - EventThread shut down
```

Take note of the `Access Key` and `App ID`, which will be used soon.

## Create a new Engine from an Engine Template

Now let's create a new engine called *MyEngine* by cloning the MLlib Collaborative Filtering engine template:

```
$ cp -r /home/yourname/predictionio/templates/scala-parallel-recommendation MyEngine
$ cd MyEngine
```

*Assuming /home/yourname/predictionio is the installation directory of PredictionIO.*

## Collecting Data

Next, let's collect some training data for the app of this Engine.
By default, the Recommendation Engine Template supports 2 types of events: "rate" and "buy".  A user can give a rating score to an item or he can buy an item.

You can send these data to PredictionIO EventServer in real-time easily through the EventAPI with a SDK or HTTP call:

<div class="codetabs">
<div data-lang="Python SDK">

{% highlight python %}
import predictionio

client = predictionio.EventClient(
    access_key=<ACCESS KEY>,
    url=<URL OF EVENTSERVER>,
    threads=5,
    qsize=500
)

# A user rates an item
client.create_event(
    event="rate",
    entity_type="user",
    entity_id=<USER ID>,
    target_entity_type="item",
    target_entity_id=<ITEM ID>,
    properties= { "rating" : float(<RATING>) }
)

# A user buys an item
client.create_event(
        event="buy",
        entity_type="user",
        entity_id=<USER ID>,
        target_entity_type="item",
        target_entity_id=<ITEM ID>
)
{% endhighlight %}

</div>

<div data-lang="PHP SDK">

{% highlight php %}
(coming soon)
{% endhighlight %}
</div>


<div data-lang="Ruby SDK">

{% highlight ruby %}
(coming soon)
{% endhighlight %}

</div>

<div data-lang="Java SDK">

{% highlight java %}
(coming soon)
{% endhighlight %}

</div>

<div data-lang="REST API">

{% highlight rest %}
(coming soon)
{% endhighlight %}

</div>
</div>


You may use the sample movie data from MLlib repo for demonstration purpose. Execute the following to get the data set:

```
$ curl https://raw.githubusercontent.com/apache/spark/master/data/mllib/sample_movielens_data.txt --create-dirs -o data/sample_movielens_data.txt
```

A python import script `import_eventserver.py` is provided to import the data to Event Server using Python SDK. Replace the value of access_key parameter by your `Access Key`.

```
$ python data/import_eventserver.py --access_key obbiTuSOiMzyFKsvjjkDnWk1vcaHjcjrv9oT3mtN3y6fOlpJoVH459O1bPmDzCdv
```

You should see the following output:

```
Importing data...
1501 events are imported.
```

Now the movie ratings data is stored as events inside the Event Store.








## Deploy the Engine as a Service

Now you can deploy the engine.  Make sure the appId defined in the file `engine.json` match your `App ID`:

```
...
"datasource": {
  "appId": 1
},
...
```

To build *MyEngine* and deploy it as a service:

```
$ pio build
$ pio train
$ pio deploy
```

This will deploy an engine that binds to http://localhost:8000. You can visit that page in your web browser to check its status.


![Engine Status]({{ site.baseurl }}/images/engine-server.png)

|

Now, You can try to retrieve predicted results.
To recommend 4 movies to user whose id is 1, you send this JSON { "user": 1, "num": 4 } to the deployed engine and it will return a JSON of the recommended movies.

```
$ curl -H "Content-Type: application/json" -d '{ "user": 1, "num": 4 }' http://localhost:8000/queries.json

{"productScores":[{"product":22,"score":4.072304374729956},{"product":62,"score":4.058482414005789},{"product":75,"score":4.046063009943821},{"product":68,"score":3.8153661512945325}]}
```

Your MyEngine is now running. Next, we are going to take a look at the engine architecture and explain how you can customize it completely.

#### [Next: DASE Components Explained](dase.html)