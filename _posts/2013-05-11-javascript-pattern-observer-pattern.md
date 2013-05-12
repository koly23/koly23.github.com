---
layout: post
title: "Javascript pattern - Observer"
description: "The observer pattern in Javascript"
category: javascript pattern
tags: [Javascript pattern]
---
{% include JB/setup %}

**A reading note of <<Learning Javascript Design Patterns>>.**

##Why observer?
Kinda like event trigger mechanism.

##What is observer pattern?
The pattern is also known as the Publish/Subscribe pattern.

A design pattern which allows an object(known as a subscriber) to watch another object(the publisher), where we provide a means for the subscriber and publisher from a listen and broadcast relationship.

Three key words: **watch**, **listen**, **broadcast**

The subscribers want to receive topic notifications from the publisher. So they are able to register(subscribe). When the publisher wants to notify, it broadcasts(publishes) a notification of these to each observer.

The subscribers can unregister themselves.

####Advantage
Loose coupling. Rather than single objects calling on the methods of other objects directly, they instead subscribe to a specific task or activity of another object and are notified when it occurs.
####Disadvantages
Because of the decoupling, sometimes, it's difficult to obtain guarantees that particular parts of our applications are functioning as we may expect.
For example, a publisher makes an assumption that one or more subscribers are listening to them. It wants to log or ouput errors regarding some application process. If the subscriber performing the logging crashes, the publisher won't have a chance to know it because of the decoupling.
Another one is observers are quit ignorant to the existence of each other and are blind to the cost of switching in subject. So it's hard to track the update dependency.


##Demo
{% highlight javascript %}
var pubsub = {};

(function(q) {
  var topics = {},
      subUid = -1;
      
  //add the token and function
  //use the token as a key to find the object later
  q.subscribe = function(topic, func){
    if (!topics[topic]){
      topics[topic] = [];
    }
    var token = (++subUid).toString();
    topics[topic].push(){
      token: token,
      func: func
    };
    return token;
  };
  
  //find and delete
  q.unsubscribe = function(token) {
    for (var m in topics){
      if (topics[m]){
        for (var i = 0; i < topics[m].length; i++){
          if (topics[m][i].token === token){
            topics[m].splice(i, 1);
            return token;
          }
        }
      }
    }
  }
  
  q.publish = function(topic, args){
    if (!topics[topic]){
      return false;
    }
    var subscribers = topics[topic],
        len = subscribers ? subscribers.length : 0;
    while (len--){      //can use in to traverse the array
      subscribers[len].func(topic, args);
    }
    return this;
  };
})(pubsub);

{% endhighlight %}

usage:

{% highlight javascript %}

//this is the subscriber
var testHandler = function(topics, data){
  console.log(topics + ": " + data);
};

var testSubscription = pubsub.subscribe('example', testHandler);

pubsub.publish('example', 'hello world!');

pubsub.unsubscribe(testSubscription);

pubsub.publish('example', 'hello again! (this will fail)');

{% endhighlight %}

##Observer pattern in Java
The publisher:

{% highlight java %}

public class Publisher {

  private ArrayList<Subscriber> subscribers = new ArrayList();
  
  public void register(Subscriber subscriber){
    subscribers.add(subscriber);
  }
  
  public void unregister(subscriber){
    subscribers.remove(subscriber);
  }
  
  public void broadcast(){
    for (Subscriber sub : subscribers){
      sub.update();
    }
  }
  
}

{% endhighlight %}

The subscriber:

{% highlight java %}

public class Subscriber{
  public void update(){
    System.out.println("the subscriber is notified!");
  }
}

{% endhighlight %}

Usage:

{% highlight java %}

Publisher publisher = new Publisher();
Subscriber subscriber = new Subscriber();
publisher.register(subscriber);
publisher.broadcast();
publisher.unregister(subscriber);
publisher.broadcast();

{% endhighlight %}

Now, the classes depends on the specific details. The further step is to use interface to make it depend on abstract, not the details.

However, it's still a very simple example of Observer.

##Discussion
* Three functions: register, unregister, broadcast
* Need a data structure to store the subscribers
* Find and delete, traverse