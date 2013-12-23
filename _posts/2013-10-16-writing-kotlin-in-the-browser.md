---
layout: post
title: Writing Kotlin in the Browser
categories:
- JetBrains
tags:
- JavaScript
- Kotlin
status: publish
type: post
published: true
meta:
  _publicize_pending: '1'
  _elasticsearch_indexed_on: '2013-10-16 05:24:11'
  twitter_cards_summary_img_size: a:6:{i:0;i:610;i:1;i:499;i:2;i:3;i:3;s:24:"width="610"
    height="499"";s:4:"bits";i:8;s:4:"mime";s:9:"image/png";}
---
Did you know that Kotlin can target JavaScript as well as the JVM? Don’t be too surprised if you didn’t know, as we’ve not been giving it too much coverage, despite already shipping a <a href="http://blog.jetbrains.com/webide/2012/08/liveedit-plugin-features-in-detail/">successful product</a> that uses this capability. But here’s hoping to change things.
<h1>The Basics – A Simple Project</h1>
First step is to set up a new project. When using Kotlin in a project, we have the ability to target either the JVM or JavaScript. If we’re adding a new Kotlin file to an existing project, <a href="http://blog.jetbrains.com/kotlin/2013/10/how-to-configure-kotlin-in-your-project/">Kotlin will prompt us for this</a>. If we’re starting a new project, we can choose the target during the setup wizard, assuming we’re using IntelliJ IDEA.

<img style="padding-top:0;padding-left:0;padding-right:0;border-width:0;" alt="image" src="{{ site.images }}/image.png" width="610" height="499" border="0" />

We also need to add the Kotlin standard library to the project. Clicking on the Create button we’ll be prompted with the location where we want these files copied. By default it copies is to a folder named <em>script</em>. And we’ll get to this later on as it’s important.

<img style="padding-top:0;padding-left:0;padding-right:0;border-width:0;" alt="image" src="{{ site.images }}/image1.png" width="603" height="157" border="0" />
<h2>Project organization</h2>
The resulting structure of the project should be

<img style="padding-top:0;padding-left:0;padding-right:0;border-width:0;" alt="image" src="{{ site.images }}/image2.png" width="350" height="196" border="0" />

The project has two files that are added:
<ul>
	<li>The <em>lib</em> folder contains the header files, as a jar file, which are used during development.</li>
	<li>The <em>script </em>folder contains a single <em>kotlin.js </em>which is the Kotlin runtime library. This is used in production.</li>
</ul>
All our Kotlin source code should be placed in the <em>src </em>folder. Once this is compiled, it will generate some JavaScript files that need to then be shipped along with the <em>kotlin.js</em> runtime file.

All other source code, be this external JavaScript libraries or files, CSS and HTML files can be placed anywhere, preferably in the same project to make development easier but not necessarily. For our example we’re going to place this in a folder called <em>web </em>and create a series of subfolders to structure our code.

<img style="padding-top:0;padding-left:0;padding-right:0;border-width:0;" alt="image" src="{{ site.images }}/image3.png" width="344" height="259" border="0" />
<h2>Setting up a workflow</h2>
When we write Kotlin code,the compiler will generate some JavaScript which needs to be shipped as part of our application. By default, IntelliJ IDEA will output this file and its sourcemap to a folder named <em>out/production/{project_name}</em>

<img style="padding-top:0;padding-left:0;margin:0;padding-right:0;border-width:0;" alt="image" src="{{ site.images }}/image4.png" width="275" height="154" border="0" />

During development we need to have an up to date version of these files so ideally we’d like to have these located in our <em>web/js/app </em>folder. We can do this in many ways, either using <a href="http://www.jetbrains.com/idea/webhelp/artifact.html">IntelliJ IDEA artifacts</a> or Maven/Gradle. In our case we’re just going to use an artifact. We can set one up to copy the corresponding files to the desired output location and additional also copy the <em>kotlin.js</em> file that was originally copied to the <em>script </em>folder to the same location*.

<strong><em><span style="font-size:x-small;">*This is a one-time operation so a better alternative is to define the output location of this file directly to our required output folder when setting up the project. We have done it this way to explain things step by step. </span></em></strong>
<h2>Interacting with DOM Elements</h2>
Now that we have the project layout ready, let’s start writing some code. The most basic thing we can do is manipulate some DOM elements. We can create a simple HTML page (named <em>index.html</em>) and place it under the <em>web </em>folder

{% highlight html %}
<!DOCTYPE html>
<html>
<head>
    <title>This page is manipulated with Kotlin</title>
</head>
<body>
    <input type="text" id="email">
</body>
</html>
{% endhighlight %}

The idea is to now update the value of the input field using Kotlin.  For that we can create a new file called <em>main</em>.<em>kt </em>and place it under our <em>src </em>folder.
<h2>Web-targeted Libraries</h2>
Kotlin provides a series of libraries targeted specifically at the web. In our case, since we want to manipulate the DOM, we can import the <em>js.dom.html </em>to access the <em>document</em>variable. The resulting code would be

{% highlight kotlin %}
package org.hadihariri.js.basics

import js.dom.html.document

fun main(args: Array<String>) {

    document.getElementById("email").setAttribute("value", "hello@kotlinlang.org")

}
{% endhighlight %}

which is very straightforward. We’re using the <em>document.getElementById </em>to retrieve the DOM element and then setting its value using <em>setAttribute</em>. Exactly the same way we’d do it using JavaScript, except here we’re using Kotlin and have the benefit of static typing, among other things.

The standard library already provides support for DOM manipulation, HTML 5 features such as Canvas and Local Storage, as well as wrappers for common libraries such as jQuery. We will be adding more as we go along, and we’ll cover some of them in future posts.
<h2>Running the code</h2>
Now that we have the code compiled, we need to actually run it. For this we have to reference both the <em>kotlin.js </em>as well as the generated (<em>basic.js</em>) file from our <em>index.html</em> page.

{% highlight kotlin %}
package org.hadihariri.js.basics

import js.dom.html.document

fun main(args: Array<String>) {

    document.getElementById("email").setAttribute("value", "hello@kotlinlang.org")

}
{% endhighlight %}

The code corresponding to the <em>main</em> function will automatically be called when the page is loaded.

Once we load our <em>index.html </em>page, we should see the result.

<img alt="image" src="{{ site.images }}/image5.png" width="638" height="283" />
<h2>Calling Kotlin from JavaScript</h2>
What if we have for instance some code in Kotlin we want to use from JavaScript? For instance, think of scenarios where you need to do some sort of business logic that needs to be repeated both on the client and the server. Well all we need to do is write it and then call it. Here is a simple function written in Kotlin

{% highlight kotlin %}
fun calculateTax(amount: Double): Double {
    return amount + amount * 0.21
}
{% endhighlight %}

This is placed inside the same module as the application and we can call it referencing it by the Kotlin module name*

{% highlight html %}
 <input type="button" value="Calculate" onclick="alert(Kotlin.modules.basic.org.sample.calculateTax(200.0))">
{% endhighlight %}
<strong></strong>

<strong><span style="color:#000000;"><em>*This API is not final and will most likely change in the future, and probably will be much more compact.</em></span></strong>
<h1>Next Steps</h1>
That’s not all that is possible with Kotlin. One thing we haven’t mentioned is the ability to call JavaScript code from within Kotlin and that is something we’ll be covering in a another post, as this one has already become too long!

If you want to play with Kotlin to JavaScript without having to install anything, you can also try it directly in the <a href="http://kotlin-demo.jetbrains.com">browser</a>. And as always, give us your feedback!
