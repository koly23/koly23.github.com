---
layout: post
title: "Javascript Pattern - Singleton"
description: "This a small demo on javascript singleton"
category: reading notes
tags: [singleton, pattern]
---
{% include JB/setup %}

**The reading material is: <<Learning Javascript design patterns>>**

##What's Singleton?
Briefly, a class only has one instance in the application. It restricts instantiation of a class to a single object.

####Singleton in Java
Let's first see a singleton in Java:

The first version:

	public class Singleton {
	
	  private Singleton(){}
	  
	  private static Singleton instance = new Singleton();
	  
	  public static Singleton getInstance() {
	    return instance;
	  }
	}

The second version:

	public class Singleton {
	  private Singleton(){}
	
	  private static Singleton instance = null;
	  
	  public static synchronized Singleton getInstance() {
	    if (instance == null){
	      instance = new Singleton();
	    }
	    return instance;
	  }
	} 
Every time, I can get the singleton instance by calling `getInstance()` method.


##Simplest Singleton in Javascript
In javascript, there is no concept 'class'. If we want to create a unique object, the easiest way is to do:

	var mySingleton = {
	
	  property1: "something",
	  property2: "something else",
	  
	  method1: function(){
	    console.log('hello world');
	  }
	};
In javascript, the object `mySingleton` is unique. No other identical object.

If you want to extend it, you can add private members and methods by encapsulating variable and function declarations inside a closure, like:

	var mySingleton = function() {
	  //private variables and methods
	  var privateVariable = 'something private';
	  
	  function showPrivate(){
	    console.log(privateVariable);
	  }
	  
	  //public variables and methods
	  return {
	    publicMethod: function() {
	      showPrivate();
	    },
	    publicVar: 'the public can see this!'
	  };
	};
	 
The `mySingleton` function returns a singleton instance.

	var mySingleton = function() {
	  var privateVariable = 'something private';
      
      function showPrivate() {
        console.log(privateVariable);
      }
    
      return {
        publicMethod: function(){
          showPrivate();
          },
      publicVar: 'the public can see this!'
      };
	};
	      
	var one = new mySingleton();
	console.log(one);
	var two = new mySingleton();
	console.log(two);

I still can instantiate two instances of `mySingleton`, so `mySingleton` is not singleton, but the `mySingleton()` is singleton, which means its return value is singleton.

##Maybe better singleton 
A maybe better one is:

	var Singleton = (function(){
	  var instantiated;
	  
	  function init() {
	    //singleton here
	    return {
	      publicMethod: function() {
	        console.log('hello world');
	      },
	      publicProperty: 'test'
	    };
	  }
	  return {
	    getInstance: function() {
	      if (!instantiated) {
	        instantiated = init();
	      }
	      return instantiated;
	    }
	  };
	})();

To get the singleton instance:

	Singleton.getInstance();
	console.log(Singleton.getInstance() === Singleton.getInstance()); //true

You cannot use:

	var one = new Singleton();
Because the Singleton is acturally an object.
This one is most like the `Singleton` in Java.

##One use case
The singleton is useful when exactly one object is needed to coordinate patterns across the system.

Example:

	var SingletonTester = (function(){
	  //options: an object containing configuration options for the singleton
	  //e.g var options = { name: 'test', pointX: 5};
	  function Singleton(options) {
	    options = options || {};
	    this.name = 'SingletonTester';
	    this.pointX = options.pointX || 6;
	    this.pointY = options.pointY || 10;  
	  }
	  
	  //this is our instance holder
	  var instance;
	  
	  //this is an emulation of static variables and methods
	  var _static = {
	    name: 'SingletonTester',
	    //returns a singleton instance of a singleton object
	    getInstance: function (options) {
	      if (instance === undefined){
	        instance = new Singleton(options);
	      }
	      return instance;
	    }
	  };
	  return _static;
	})();

To use it:

	var singletonTest = SingletonTester.getInstance({
	  pointX: 5
	});
	
	console.log(singletonTest.pointX); //output 5
	vat two = SingletonTester.getInstance({
	  pointX: 6
	});
	console.log(singletonTest.pointX); //still output 5
	singletonTest === two  //true

If you're confused about the 'this', check another blog.

##Discussion
Key of singleton:

* can only have one instance
* need a private variable to hold the unique instance  
* provide a public acesser  
