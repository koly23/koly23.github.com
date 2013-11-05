---
layout: post
title: "AngularJS Directive full version"
description: "A breif introduction with examples"
category: 
tags: [angularjs directive]
---
{% include JB/setup %}

This is a following article after [AngularJS directive simple version](http://koly23.github.io/angularjs-directive---simple-version/2013/10/07/). Before reading this article, it's better to read the previous article.

If you want to try the example code, try use [AngularJS seed](https://github.com/angular/angular-seed).

The code may not be correct, so be careful. When you're reading the example, it is assumed that you know basics of AngularJS.

* [A simple one to start](#a_simple_one_to_start)
* [restrict](#restrict)
* [replace](#replace)
* [template](#template)
* [templateUrl](#templateurl)
* [controller](#controller)
* [transclude](#transclude)
* [priority](#priority)
* [terminal](#terminal)
* [scope](#scope)
* [require](#require)

In the full version, the directive function returns an object:

{% highlight javascript %}
var app = angular.module('App', []);

app.directive('sample', function(){
	var obj = {
		...
	};
	return obj;
});
{% endhighlight %}

The returned object has some attributes, which we will explain one by one.

###A simple one to start

{% highlight javascript %}
app.directive('sample', function(){
	var obj = {
		restrict: "E",
		replace: true,
		template: "<input type='text'>",

	};
	return obj;
});
{% endhighlight %}

usage:

{% highlight html %}
<sample>
</sample>
{% endhighlight %}

after the directive is parsed by AngularJS, the html result will be:

{% highlight html %}
<input type='text'>
{% endhighlight %}

###restrict

The restrict attribute defines the way how we use the directive in html. There are four choices: "A", "E", "C", "M". For example, we can change the "restrict" in previous example to:

"A", then in the html, we should use:

{% highlight html %}
<div sample></div>
{% endhighlight %}

"C", then in the html:

{% highlight html %}
<div class="sample"></div>
{% endhighlight %}

"M", then  in the html:

{% highlight html %}
<!-- directive: sample -->
{% endhighlight %}

###replace

The replace attribute means the element on which the directive is put will be replaced by the template. The value of this can be 'true' or 'false'. The default value is 'false', which will replace the *contents* of the current element:


{% highlight javascript %}
app.directive('sample', function(){
	var obj = {
		restrict: "E",
		template: "<div>Content in the sample directive</div>",

	};
	return obj;
});
{% endhighlight %}

usage:

{% highlight html %}
<sample>
	<a href="#">This is an link.</a>
</sample>
{% endhighlight %}

after the directive is parsed by AngularJS, the html result will be:

{% highlight html %}
<sample>
	<div>Content in the sample directive</div>
</sample>
{% endhighlight %}

The original link element will be replaced by the template, but the element(directive `sample`) will be retained. If we set the replace to 'true', there would be no `<sample>` element.

###template

Template is just a piece of html which can have some angularJS expression or directive, etc.

If you see 'template' in the official document, don't get confused, it's just a piece of html.

###templateUrl

We can use templateUrl instead of template, the former one is an url. AngularJS can get a piece of html through the url, which means you need a *server* to serve the template file.

or:

in the html:

{% highlight html %}
  <script type="text/ng-template" id="test.html">
    <input type="checkbox" ng-model="check">
    <span ng-show="!check">click checkbox to hide me.</span>
  </script>
  <div sample>
    <a href="#">This is an link.</a>
  </div>

{% endhighlight %}

the directive:

{% highlight javascript %}
app.directive('sample', function(){
	var obj = {
		restrict: "A",
		templateUrl: 'test.html'
	};
	return obj;
});
{% endhighlight %}

One thing is *the script tag must be before the usage of the directive*, or AngularJS will not find the template when compiling the directive.

###controller

This defines a controller constructor on the directive. We focus on the parameters:

html:

{% highlight html %}

  <div ng-controller="MyCtrl">
    <div sample>
      <a href="#">This is an link.</a>
    </div>
  </div>

{% endhighlight %}

javascript:

{% highlight javascript %}
app.directive('sample', function(){
	var obj = {
		restrict: "A",
		controller: function($scope, $element, $attrs){
			console.log($scope, 'scope in directive controller');
			console.log($element, 'element in directive controller');
			console.log($attrs, 'attrs in directive controller');
		},
		templateUrl: 'test.html'
	};
	return obj;
});
{% endhighlight %}

The controller attribute is a function. It has four parameters: `$scope`, this is the scope the element is in. Here it is the scope create by `ng-controller="MyCtrl"`; `$element`, the element, same as that in the link function; `attrs`, attributes of the element.

The forth parameter is `$transclude`, which we will cover in the next section.

###transclude

Let's see an example first:

html:

{% highlight html %}

  <script type="text/ng-template" id="test.html">
  	<input type="checkbox" ng-model="check">
  	<span ng-show="!check">click checkbox to hide me.</span>
  	<div ng-transclude></div>
  </script>

  <div ng-controller="MyCtrl">
    <div sample>
      <a href="#">This is an \{\{test}} link.</a> // remove the backslash
    </div>
  </div>

{% endhighlight %}

javascript:

{% highlight javascript %}
app.directive('sample', function(){
	var obj = {
		restrict: "A",
		templateUrl: 'test.html',
		transclude: true
	};
	return obj;
});

app.controller('MyCtrl', ["$scope", function($scope){
	$scope.test = "test";
}]);
{% endhighlight %}

result:

{% highlight html %}
<div ng-controller="MyCtrl" >
	<div sample>
		<input type="checkbox" ng-model="check">
		<span ng-show="!check">click checkbox to hide me.</span>
		<div ng-transclude>
			<a href="#">This is an test link.</a>
		</div>
	</div>
</div>
{% endhighlight %}

In the example, first the template is loaded and compiled. Then, the content of the element which the directive is on (`<a href="#">This is an {{test}} link.</a>`) is compiled and stick to the content of the element which `ng-transclude` is on. Which means the `<div ng-transclude></div>` would become:

{% highlight html %}
<div ng-transclude>
	<a href="#">This is an test link.</a> //the original content is replaced.
</div>
{% endhighlight %}

And, we add the `$transclude` parameter in the controller attribute function:

{% highlight javascript %}
app.directive('sample', function(){
	var obj = {
		restrict: "A",
		templateUrl: 'test.html',
		transclude: true,
		controller: function($scope, $element, $attrs, $transclude) {
        console.log($transclude, 'transclude in directive controller');
        $transclude(function(clone){
          console.log(clone, 'clone in directive controller');
        });
      }
	};
	return obj;
});
{% endhighlight %}

The parameter `$transclude` is a function, we call the funtion and pass a function in. The `clone` parameter is an array, which contains element inside the element which the `ng-transclude` is on. Here the element is a link element.

A more detailed article on transclude is [here](http://blog.omkarpatil.com/2012/11/transclude-in-angularjs.html).

###priority

The priority attribute defines the executing sequence of directives on the same element.

html:

{% highlight html %}
<div sample-one sample-two>Here is the priority test.</div>
{% endhighlight %}

javascript:

{% highlight javascript %}
app.directive('sampleOne', function(){
	var obj = {
		priority: "1",
		link: function($scope, iElm, iAttrs, controller) {
           console.log('sample one');
        }
	};
	return obj;
});

app.directive('sampleTwo', function(){
	var obj = {
		priority: "1",
		link: function($scope, iElm, iAttrs, controller) {
           console.log('sample two');
        }
	};
	return obj;
});
{% endhighlight %}

and the output is:

{% highlight html %}
sample one
sample two
{% endhighlight %}

If we change the html to:

{% highlight html %}
<div sample-two sample-one>Here is the priority test.</div>
{% endhighlight %}

the result will be:

{% highlight html %}
sample two
sample one
{% endhighlight %}

Now we change the priority of sample one to `2`:

html:

{% highlight html %}
<div sample-two sample-one>Here is the priority test.</div>
{% endhighlight %}

javascript:
{% highlight javascript %}
app.directive('sampleOne', function(){
	var obj = {
		priority: "2",
		link: function($scope, iElm, iAttrs, controller) {
           console.log('sample one');
        }
	};
	return obj;
});
{% endhighlight %}

the result would be:

{% highlight html %}
sample one
sample two
{% endhighlight %}

###terminal

In the previous example, we add a `terminal` attribute to the directive:

html:

{% highlight html %}
<div sample-zero sample-one sample-two>Here is the priority test.</div>
{% endhighlight %}

javascript:

{% highlight javascript %}
app.directive('sampleOne', function(){
	var obj = {
		priority: "1",
		link: function($scope, iElm, iAttrs, controller) {
           console.log('sample one');
        }
	};
	return obj;
});

app.directive('sampleTwo', function(){
	var obj = {
		priority: "1",
		link: function($scope, iElm, iAttrs, controller) {
           console.log('sample two');
        }
	};
	return obj;
});

app.directive('sampleZero', function(){
	var obj = {
		link: function($scope, iElm, iAttrs, controller) {
           console.log('sample zero');
        }
	};
	return obj;
});
{% endhighlight %}

and the output is:

{% highlight html %}
sample one
sample two
sample zero
{% endhighlight %}

Now, we add the terminal attribute to `sampleOne`, which would be:

{% highlight javascript %}
app.directive('sampleOne', function(){
	var obj = {
		priority: "1",
		terminal: true;
		link: function($scope, iElm, iAttrs, controller) {
           console.log('sample one');
        }
	};
	return obj;
});
{% endhighlight %}

and the result would be:

{% highlight html %}
sample one
sample two
{% endhighlight %}

Notice that there is no `sample zero`, because the `terminal` property tells Angular to skip all directives on that element that comes after it.

###scope

Take the previous codes in 'controller' section:

{% highlight html %}

  <div ng-controller="MyCtrl">
    <div sample>
      <a href="#">This is an link.</a>
    </div>
  </div>

{% endhighlight %}

javascript:

{% highlight javascript %}
app.directive('sample', function(){
	var obj = {
		restrict: "A",
		controller: function($scope, $element, $attrs){
			console.log($scope, 'scope in directive controller');
			console.log($element, 'element in directive controller');
			console.log($attrs, 'attrs in directive controller');
		},
		templateUrl: 'test.html'
	};
	return obj;
});
{% endhighlight %}

Here, we didn't define the `scope`, which means the value of `scope` is the default one: false.

The scope we log out is the same with the controller `MyCtrl`. For example, the `$id` of the scope parameter maybe `003`, which is the same with the `MyCtrl`.

What if we set the `scope` to `true`?

html:

{% highlight html %}

  <div ng-controller="MyCtrl">
    <div sample>
      <a href="#">This is an link.</a>
    </div>
  </div>

{% endhighlight %}

javascript:

{% highlight javascript %}
app.directive('sample', function(){
	var obj = {
		restrict: "A",
		scope: true,
		controller: function($scope, $element, $attrs){
			console.log($scope, 'scope in directive controller');
			console.log($element, 'element in directive controller');
			console.log($attrs, 'attrs in directive controller');
		},
		templateUrl: 'test.html'
	};
	return obj;
});
{% endhighlight %}

Check out the console output, we could see that the scope changed. The `$id` of the scope inside the controller construct function is, for example, `004`. And the `MyCtrl` controller's `$id` is `003`. And if we check the `$parent` attribute of the `004` scope, we will find it's the `003` scope.

So, if the `scope` is set to `true`, a new child scope will be created. And if no attribute were found on the current scope, the parent scope will be searched.

Besides `true`, we also can set the `scope` to an object `{}`.

Example:

html:

{% highlight html %}

  <div ng-controller="MyCtrl">
    <div sample>
      <a href="#">This is an link.</a>
    </div>
  </div>

{% endhighlight %}

javascript:

{% highlight javascript %}
app.directive('sample', function(){
	var obj = {
		restrict: "A",
		scope: {},
		controller: function($scope, $element, $attrs){
			console.log($scope, 'scope in directive controller');
			console.log($element, 'element in directive controller');
			console.log($attrs, 'attrs in directive controller');
		},
		templateUrl: 'test.html'
	};
	return obj;
});
{% endhighlight %}

Then, we check the output, the `$parent` parameter is still `003`. However, if we add some change:

`MyCtrl`:

{% highlight javascript %}

angular.module('myApp.controllers', [])
  .controller('MyCtrl', ['$scope', function($scope){
  	$scope.test = "test";
  	console.log($scope, 'scope in controller');
  	console.log($scope.test);
  }]);

{% endhighlight %}

directive:

{% highlight javascript %}
app.directive('sample', function(){
	var obj = {
		restrict: "A",
		scope: {},
		controller: function($scope, $element, $attrs){
			console.log($scope, 'scope in directive controller');
			console.log($element, 'element in directive controller');
			console.log($attrs, 'attrs in directive controller');
			// try to log the value in `MyCtrl` controller scope
			console.log($scope.test, 'test inside directive');
		},
		templateUrl: 'test.html'
	};
	return obj;
});
{% endhighlight %}

With the `{}`, the output of `$scope.test` inside directive is 'undefined'. With `true`, the value is `test`, which is defined in the controller scope.

By using `{}`, we created an 'isolated' scope.

We can use `@`, `=`, `&` in the `{}`.

For `@`:

directive:

{% highlight javascript %}
app.directive('sample', function(){
	var obj = {
		restrict: "A",
		scope: {
			localName: '@myAttr'
		},
		controller: function($scope, $element, $attrs){
			console.log($scope, 'scope in directive controller');
			console.log($element, 'element in directive controller');
			console.log($attrs, 'attrs in directive controller');
			// try to log the value in `MyCtrl` controller scope
			console.log($scope.localName, 'test inside directive');
		},
		templateUrl: 'test.html'
	};
	return obj;
});
{% endhighlight %}

usage in html:

{% highlight html %}

<div sample my-attr="hello">
   <a href="#">This is an link.</a>
</div>

{% endhighlight %}

The output of `$scope.localName` is `undefined`. And check the output `$scope`, we will find the `localName` attribute and the value is `hello`. However, if you debug and check the attributes of `$scope`, you would not find `localName` attribute.

Based on the previous codes, let's add a link function:

{% highlight javascript %}
app.directive('sample', function(){
	var obj = {
		restrict: "A",
		scope: {
			localName: '@myAttr'
		},
		controller: function($scope, $element, $attrs){
			console.log($scope, 'scope in directive controller');
			console.log($element, 'element in directive controller');
			console.log($attrs, 'attrs in directive controller');
			// try to log the value in `MyCtrl` controller scope
			console.log($scope.localName, 'test inside directive');
		},
		templateUrl: 'test.html',
		link: function($scope, iElm, iAttrs, controller) {
        	console.log($scope, 'scope in link function');
        	console.log($scope.localName, 'local name in link function.');
      }
	};
	return obj;
});
{% endhighlight %}

Unfortunately, the value of `$scope.localName` is still `undefined`, which means the `$scope` inside the link function still cannot get access to `localName`. So where can we use the `localName`?

Let's change our `test.html`:

{% highlight html %}
  <script type="text/ng-template" id="test.html">
    <input type="checkbox" ng-model="check" />
    <span ng-show="!check">{{localName}}, click checkbox to hide me.</span>
  </script>
{% endhighlight %}

the result:

{% highlight html %}
<input type="checkbox" ng-model="check" class="ng-pristine ng-valid">
<span ng-show="!check" class="ng-binding">hello, click checkbox to hide me.</span>
{% endhighlight %}

Here we see the value `hello` is outputed to the html. So the value can only be used in the template. 

What can we use the `$scope` parameter? Well, the purpose is more to add attributes to `$scope` rather than reading something from it. Like writing rather than reading. So that in the template you can use the attributes of the `$scope`. So you can treat both controller and link function as a prepration of the template.

Another trick is:

{% highlight javascript %}
app.directive('sample', function(){
	var obj = {
		restrict: "A",
		scope: {
			localName: '@'
		},
		controller: function($scope, $element, $attrs){
			console.log($scope, 'scope in directive controller');
			console.log($element, 'element in directive controller');
			console.log($attrs, 'attrs in directive controller');
			// try to log the value in `MyCtrl` controller scope
			console.log($scope.localName, 'test inside directive');
		},
		templateUrl: 'test.html',
		link: function($scope, iElm, iAttrs, controller) {
        	console.log($scope, 'scope in link function');
        	console.log($scope.localName, 'local name in link function.');
      }
	};
	return obj;
});
{% endhighlight %}

Inside the `scope` attribute, we remove the 'myAttr', and the usage is changed:

{% highlight html %}

<div sample local-name="hello">
   <a href="#">This is an link.</a>
</div>

{% endhighlight %}

so inside the `scope` attribute, `@` is the same with `@localName`. The attribute name(localName) is used by default.

In general, we use `@` to bind a local variable(`localName`) to the DOM attribute(`local-name`); 

*And the DOM attribute's value must be a string*. If you want to set an attribute to the attribute, it won't work. Like: `local-name="{name:'koly'}"`.

That's about `@`, now we talk about `=`: set up bi-directional binding between a local scope property and the parent scope property. Since it's a bi-directional binding, so if we change one, the other will be changed too.

directive:

{% highlight javascript %}
app.directive('sample', function(){
	var obj = {
		restrict: "A",
		scope: {
			localName: '=parentVar'
		},
		controller: function($scope, $element, $attrs){
			console.log($scope, 'scope in directive controller');
			console.log($element, 'element in directive controller');
			console.log($attrs, 'attrs in directive controller');
			console.log($scope.localName, 'test inside directive');
		},
		templateUrl: 'test.html',
		link: function($scope, iElm, iAttrs, controller) {
        	console.log($scope, 'scope in link function');
        	console.log($scope.localName, 'local name in link function.');
      }
	};
	return obj;
});
{% endhighlight %}

usage:

{% highlight html %}
  <div ng-controller="MyCtrl">
    <div sample parent-var="hello">
      <a href="#">This is an {{test}} link.</a>
    </div>
  </div>
{% endhighlight %}

Here the `hello` is not a string, but a object in the controller(the parent scope).

controller:

{% highlight javascript %}
angular.module('myApp.controllers', [])
  .controller('MyCtrl', ['$scope', function($scope){
    $scope.hello = "hello world!";
  }]);
{% endhighlight %}

result:

{% highlight html %}
<span ng-show="!check" class="ng-binding">hello world!, click checkbox to hide me.</span>
{% endhighlight %}

the value of the `$scope.hello` is outputed in the template. If you omit the `parentVar`, it will use `localName` by default, which is the same rule with `@`.

The last one is `&`: provides a way to execute an expression in the context of the parent scope.

directive:

{% highlight javascript %}
app.directive('sample', function(){
	var obj = {
		restrict: "A",
		scope: {
			localFn: '&'
		},
		controller: function($scope, $element, $attrs){
			console.log($scope, 'scope in directive controller');
			console.log($element, 'element in directive controller');
			console.log($attrs, 'attrs in directive controller');
			console.log($scope.localFn, 'test inside directive');
		},
		templateUrl: 'test.html',
		link: function($scope, iElm, iAttrs, controller) {
        	console.log($scope, 'scope in link function');
        	console.log($scope.localFn, 'local name in link function.');
      }
	};
	return obj;
});
{% endhighlight %}

html:

{% highlight html %}
  <div ng-controller="MyCtrl">
    <div sample localFn="add()">
      <a href="#">This is an {{test}} link.</a>
    </div>
  </div>
{% endhighlight %}

controller:

{% highlight javascript %}
angular.module('myApp.controllers', [])
  .controller('MyCtrl', ['$scope', function($scope){
    $scope.add = function(){
    	alert("add in controller");
    }
  }]);
{% endhighlight %}

template:

{% highlight html %}
  <script type="text/ng-template" id="test.html">
    <p ng-click="localFn()">click me</p>
  </script>
{% endhighlight %}

result:

{% highlight html %}
<p ng-click="localFn()">click me.</p>
{% endhighlight %}

If we click the "I am from link function.", an alert message will be displayed. This means the `add` function in the controller is called.

We can also pass data to the `add` function:

html:

{% highlight html %}
  <div ng-controller="MyCtrl">
    <div sample localFn="add(a, b)">
      <a href="#">This is an {{test}} link.</a>
    </div>
  </div>
{% endhighlight %}

controller:

{% highlight javascript %}
angular.module('myApp.controllers', [])
  .controller('MyCtrl', ['$scope', function($scope){
    $scope.add = function(a, b){
		alert(a+b);
    }
  }]);
{% endhighlight %}

template:

{% highlight html %}
  <script type="text/ng-template" id="test.html">
    <p ng-click="localFn({a:1, b:2})">click me</p>
  </script>
{% endhighlight %}

result:

{% highlight html %}
<p ng-click="localFn({a:1, b:2})">click me.</p>
{% endhighlight %}

When click the 'click me.', an alert message with content '3', will be displayed. In this way, we can call the method of parent scope and pass values to it.

###require

Require another controller be passed into current directive *linking* function.

Let's see an example:

html:

{% highlight html %}
  <div ng-controller="MyCtrl">
    <input my-require my-tool type="text" ng-model="hello">
  </div>  
{% endhighlight %}

directive:

{% highlight javascript %}
angular.module('myApp.controllers', [])
  .directive('myRequire', function(){
    return {
      controller: function($scope, $element, $attrs){
          this.hello = "hello";
      }
    };
  })
  .directive('myTool', function(){
    return {
      require: 'myRequire',
      link: function($scope, iElm, iAttrs, controller){
        console.log(controller, 'controller in myTool directive');
        console.log(controller.hello, 'hello from contructor');
      }
    };
  });
{% endhighlight %}

Here, we can see the log out controller is an object, this object is generated by the controller constructor function in directive `myRequire`. We will cover the constructor function later.

Another example is about `ngModel`:


html:

{% highlight html %}
  <div ng-controller="MyCtrl">
    <input my-require my-tool type="text" ng-model="hello" placeholder="{{hello}}">
    <div>{{hello}}</div>
  </div>  
{% endhighlight %}

directive:

{% highlight javascript %}
angular.module('myApp.controllers', [])
  .directive('myTool', function(){
    return {
      require: 'ngModel',
      link: function($scope, iElm, iAttrs, controller){
	    ngModel.$setViewValue("value in directive");
        console.log(controller, 'controller in myTool directive');
        console.log(ngModel.$viewValue, 'hello from contructor');
      }
    };
  });
{% endhighlight %}

result:

{% highlight html %}
<input my-require="" my-tool="" type="text" ng-model="hello" placeholder="value in directive" class="ng-valid ng-dirty">
<div class="ng-binding">value in directive</div>
{% endhighlight %}

By using `$viewValue`, we can get the current value of the model `hello`, and we can set the value by calling `$setViewValue()`.

However, why there are attributes and functions in `ngModel` controller constrcutor function?
Let's see the previous example again:

html:

{% highlight html %}
  <div ng-controller="MyCtrl">
    <input my-require my-tool type="text" ng-model="hello">
  </div>  
{% endhighlight %}

directive:

{% highlight javascript %}
angular.module('myApp.controllers', [])
  .directive('myRequire', function(){
    return {
      controller: function($scope, $element, $attrs){
          this.hello = "hello";
          this.helloFn = function(){
          	return "function";
          };
      }
    };
  })
  .directive('myTool', function(){
    return {
      require: 'myRequire',
      link: function($scope, iElm, iAttrs, controller){
        var a = controller.hello + controller.helloFn();
        $scope.hello = a;
        console.log(controller, 'controller in myTool directive');
        console.log(a, 'hello from contructor');
      }
    };
  });
{% endhighlight %}

result:

{% highlight html %}
<input my-require="" my-tool="" type="text" placeholder="hellofunction">
<div class="ng-binding">hellofunction</div>
{% endhighlight %}

So, what is constructor function? First, let's see how could we build an object in javascript?

{% highlight javascript %}
var person = {
	id : 1,
	name : 'koly'
};
console.log(person);
{% endhighlight %}

What if we have another person? 

{% highlight javascript %}
var personTwo = {
	id : 2,
	name : 'yun'
};
console.log(personTwo);
{% endhighlight %}

What if there are a lot of people? Acturally, we can build a contructor to build these people:

{% highlight javascript %}
var People = function(id, name){
	this.id = id;
	this.name = name;
};
var person = new People(1, 'koly');
var personTwo = new People(2, 'yun');
console.log(person);
console.log(personTwo);
{% endhighlight %}

The People function is a contructor function, which is used to construct people.

The two attributes left is `compile` and `name`, if you do need to use them, please search them in google.


