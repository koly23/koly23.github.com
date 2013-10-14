---
layout: post
title: "AngularJS Directive   Simple Version"
description: "An introduction on AngularJs Directive Simple Version"
category: 
tags: [angularjs]
---
{% include JB/setup %}

This is an article about AngularJS directive, and it's based on the [official document](http://docs.angularjs.org/guide/directive). This article is my own way of learning directive, it's better to read the official document first.

The content contains:

* What is AngularJS directive?
* What is the simple version of directive?
* How to write and understand the simple version?

-----
##What is AngularJS directive?
Directive is a way to extend standard HTML vocabulary.

For example, we have a div:

{% highlight html %}

<div>Click and See</div>

{% endhighlight %}

we want to add click event to this element and after user click it, the background of the element will change to green.

*What we may do without Directive is using jQuery:*

html:

{% highlight html %}
<div id="click">Click and See</div>
{% endhighlight %}

javascript:

{% highlight javascript %}
$("#click").click(function(){
	$(this).addClass('green-background');
});
{% endhighlight %}

css:

{% highlight css %}
.green-background{
	background-color: green;
}
{% endhighlight %}

In this way, we implemented the function. However we need add an id attribute on the element. We may change it to class, since next time maybe we want to make another element behave the same way.

*What if we do it in the AngualarJS Directive way*

html:

{% highlight html %}
<div ng-app="App">
<div click-change-background>Click and See</div>
</div>
{% endhighlight %}

javascript: (skip this if you don't understand it)

{% highlight javascript %}

var app = angular.module('App', []);

app.directive('clickChangeBackground', function(){
	return function(scope, elem){
		var element = angular.element(elem);
		element.bind('click', function(){
			element.addClass('green-background');
		});
	};
});

{% endhighlight %}

css:

{% highlight css %}
.green-background{
	background-color: green;
}
{% endhighlight %}

Compare to the jQuery implementation, AngularJS directive gives us a more understanding way like we created a new html word.

##What is the simple version of directive?

From the official document, there are two ways to write a directive. One is to return an object, another is to return a function. The latter is simpler. 

The way to write a simple directive is:

{% highlight javascript %}

var app = angular.module('App', []);

app.directive('camelCase', function(){
	return function(scope, iElement, iAttrs, controller){
		...
	};
});

{% endhighlight %}

So inside the directive function, we only need to return another function, which is called 'link function'. This function will be executed by AngularJS when compiling dom after the dom is parsed by the browser.

*What can we do in the returning function?*

* get reference of the element on which the directive is put
* get the normalized attributes of the element
* communicate with the scope the directive is in
* communicate with other directives who has a controller on the same element

*get reference of the element on which the directive is put*

The parameter iElement is a reference of the element. It can be used as a selector.

Example:

{% highlight javascript %}

var app = angular.module('App', []);

app.directive('camelCase', function(){
	return function(scope, iElement, iAttrs, controller){
		angular.element(iElement).addClass('green-background');
	};
});

{% endhighlight %}

In the example, first we use the element method to select the element and add a css class on it.

*get the normalized attributes of the element*

First, we have the html:

{% highlight html %}
<input camel-case name="family_name" id="family_name">
{% endhighlight %}

then, javascript:

{% highlight javascript %}

var app = angular.module('App', []);

app.directive('camelCase', function(){
	return function(scope, iElement, iAttrs, controller){
		console.log(iAttrs, 'attributes');
	};
});

{% endhighlight %}

Here, what "normalized" means is, for example:

we have a input:

{% highlight html %}
<input name="family_name" id="family_name" dl-hover app-version>
{% endhighlight %}

the "normalized attributes" are: *dlHover*, *appVersion*, *id*, *name*

"normalized" means all the attribute names will be changed to *camel-case* style. 

Another thing is we can use iAttrs to pass things to the link function, like:

html:

{% highlight html %}
<div camelCase="hello" > </div>
{% endhighlight %}

javascript:

{% highlight javascript %}

app.directive('camelCase', function(){
	return function(scope, iElement, iAttrs, controller){
		console.log(iAttrs, 'attributes');
		//this will output hello
		console.log(iAttrs.camelCase, 'camelCase');
	};
});
{% endhighlight %}

By using iAttrs.camelCase, you can get the value passed to the link function.


*communicate with other directives who has a controller on the same element*

To understand this, first we need to create a directive with controller, which will be explained in the next article. However, this is not commonly used.


On creating directive, we need to always think about several questions:

* do I really need to use directive? Can I use some other way to achieve the same objective?
* what is the advantage and disadvantage of using directive? Does it worth?

Directive is just a tool, it is the people to use it. People can use it properly, and can also use it badly. Always think before using it.

