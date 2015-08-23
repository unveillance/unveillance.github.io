---
layout: post
title:  "From 0 to Unveillance (Frontend Edition)"
date:   2015-07-06 10:58:31
categories: frontend-interface how-to code-notes
permalink: /:year/:month/:day/:title.html
---

These documents should inform our development team, and anyone else keeping up with the project, on how to encorporate the Unveillance engine into their own projects.  Taking some questions from our lead frontend dev, Jonny, as our starting point...

## Package Structure

Any application using Unveillance will incorporate the core libraries (Interface and Core).  Core contains functions universal to the either the Interface or the Annex.  Your custom application will incorporate the Interface, and your initial project structure should show them as nested within one another like a set of Russian dolls.

{% highlight text %}
|_ CusomUnveillanceApplication
	|_ lib/Frontend (a.k.a the Interface)
		|_ lib/Core
{% endhighlight %}

Conventionally, developers should not even need to modify anything in the `lib/Frontend` or `lib/Frontend/lib/Core` directories.  Customizations should be restricted to the top level of the project.  However, if the adventurous developer (like Jonny!) wants to make changes to either the Interface or Core, that should be done via standard Open Source practice of submitting pull requests.

### The `web` Directory

The `lib/Frontend` directory contains a directory for web assets.  For our purposes, I'm omitting a few subdirectories that might change, or are not too important to focus on right now, but this is the basic structure:

{% highlight text %}

|_ MyCusomUnveillanceInterface
	|_ lib/Frontend (a.k.a the Interface)
		|_ web
			|_ css
			|_ images
			|_ js
				|_ lib
				|_ models
				|_ modules
			|_ layout
				|_ tmpl
				|_ views
		|_ lib/Core
{% endhighlight %}

In order to add your own custom models, views, controllers, styles, and javascript libraries, your project must contain a similarly-structured directory in the project root, like so:

{% highlight text %}

|_ MyCusomUnveillanceInterface
	|_ web
		|_ css
		|_ images
		|_ js
			|_ lib
			|_ models
			|_ modules
		|_ layout
			|_ tmpl
			|_ views
	|_ lib/Frontend (a.k.a the Interface)
		|_ web
			|_ css
			|_ images
			|_ js
				|_ lib
				|_ models
				|_ modules
			|_ layout
				|_ tmpl
				|_ views
		|_ lib/Core
{% endhighlight %}

Anything in your custom `web` directory will be symbolically linked to the `web` directory in `lib/Frontend`, allowing you to reference whatever file you need without having to think too hard about the file path, and without having to modify any of the files in the Interface library.

### What are Modules?  What are Models?

You'll notice that the `web/js` folder contains three main folders: `lib`, `models`, and `modules`.  The `lib` directory is for third-party frameworks like jQuery, D3, Backbone, Underscore, and a few others that I've found helpful along the way.  The concept of models should be familiar to you, and feel free to create them as you see fit in the `models` directory.  

Modules, on the other hand, probably bear more explaination and fleshing out.  They're supposed to represent controllers for each page of the app.  Why is the folder not called `controllers` rather than `modules`?  I don't have an answer to that; maybe we can change the taxonomy to be more standard.