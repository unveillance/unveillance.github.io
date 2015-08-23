---
layout: post
title: "From 0 to Unveillance (Annex edition)"
date: 2015-07-07 11:00:13
tags: annex-backend how-to code-notes deepdream #bbhmm rihanna
---

Unveillance can be used for a variety of things.  It has been designed to accommodate just about any type of project that involves the automated batch processing of documents.  [Here's an example](https://github.com/DeepLab/deeplab_x_deepdream_annex) I wrote using the Annex component to automatically create [deep dream images](http://googleresearch.blogspot.com/2015/07/deepdream-code-example-for-visualizing.html) submitted via Slack!

<iframe width="640" height="360" src="https://www.youtube.com/embed/o3SExuCl8_k" frameborder="0" allowfullscreen></iframe>

On my own machine at home, I've set up the Deepdream toolkit, and am running Unveillance.  With these two software packages, my machine can respond to a command (sent via XMPP) instructing it to do an iteration or make a gif on the specified image.  Once the command has been executed, Unveillance publishes the result to our Slack channel (using the Slack API.)

## Unveillance Annex

The core Unveillance Annex package is included as a submodule in the project's `lib` folder.  Since the Annex does all the heavy lifting, there are only three things I need to do to make my deep dream machine.

### 1. Create a `vars.json` manifest

The `vars.json` manifest instructs Unveillance how to handle each type of document it'll encounter.  Since we're only dealing with images, I've set the `MIME_TYPES` directive to an object describing the mapping between the mime type name, and its corresponding file extension.  The `MIME_TYPE_MAP` is the inverse of `MIME_TYPES`.  (In the future, this could be programmed automatically, but for now, it's required.)  

The `MIME_TYPE_TASKS` directive is an object enumerating the tasks each document of a specified mime type must go through when it enters the system.  You'll notice, the tasks correspond to python modules outlined in our `Tasks` folder.

The `ASSET_TAGS` directive describes the type of assets Unveillance will generate each time a task is performed on a document.  These are handy, because the Unveillance API allows for developers to pull out types of assets by their code names.  Each tag is a mapping between the tag's namespace and a human-readable description.

### 2. Create your Models

This should be a familiar concept for any programmer.  The `Models` folder contains objects, with each of their associated methods and properties.  Most likely, your models will extend the base `UnveillanceDocument` class-- that way, your objects can inherit Unveillance-specific behavior, like saving to the database, grabbing generated assets, or creating and storing new ones.  When you subclass the `UnveillanceDocument` object, all you have to concern yourself with is writing the methods unique to your object.

```
from lib.Worker.Models.uv_document import UnveillanceDocument

class DeepDream(UnveillanceDocument):
	def __init__(self, _id=None, inflate=None):
		super(DeepDream, self).__init__(_id=_id, inflate=inflate)
```

In this demo, we have dedicated methods for creating deep dreams, making gifs, and sending results to our Slack channel.  Here's an example of how we deep dreamed on a given image:

```
def iterate(self):
	THIS_DIR = os.getcwd()
	os.chdir(os.path.join(ANNEX_DIR, self.base_path))

	try:
		iter_num = len(self.getAssetsByTagName(ASSET_TAGS['DLXDD_DD']))

		bc = BatCountry(os.path.join(getConfig('caffe_root'), "models", "bvlc_googlenet"))
		img = bc.dream(np.float32(self.get_image(file_name="dream_%d.jpg" % iter_num)))
		bc.cleanup()

		os.chdir(THIS_DIR)

		iter_num += 1
		dream = Image.fromarray(np.uint8(img))
		asset_path = self.addAsset(None, "dream_%d.jpg" % iter_num, \
			tags=[ASSET_TAGS['DLXDD_DD']], description="deep dream iteration")

		if asset_path is not None:
			dream.save(os.path.join(ANNEX_DIR, asset_path))
			return True
	
	except Exception as e:
		print "ERROR ON ITERATION:"
		print e, type(e)

	return False
```

At some point soon, we will publish a document with the full set of `UnveillanceDocument` methods and global variables (like `ANNEX_DIR`, which refers to... you guessed it-- where the Annex dumps its stuff).

### 3. Create your Tasks

It's kind of self-explanatory, but tasks are functions that process documents.  They can be chained together to execute one after the other.  Every Unveillance task is passed a `uv_task` object as an initial argument.  The `uv_task` has its own set of methods for signaling its status and augmenting the chain of tasks it belongs to.  It also contains a pointer to the document in question, so you can load it in right away.  For example, here we invoke the DeepDream image in question by quering our database for its id:

```
from lib.Worker.Models.deepdream import DeepDream

dd = DeepDream(_id=uv_task.doc_id)
dd.iterate()
```

## And so, we Deep Dreamed...

Here are some of my favorite deep dreams from this little experiment.  I was especially interested in deep dreaming Rihanna's #BBHMM, because is that video not a big old deep dream already?  (Slightly nsfw...)

![rihrih gif!](/media{{ page.id }}/bbhmm_deepdream.gif)

![original img 1](/media{{ page.id }}/dlxdd_bbhmm_1_orig.png)

![deepdream img 1](/media{{ page.id }}/dlxdd_bbhmm_1.png)

![original img 2](/media{{ page.id }}/dlxdd_bbhmm_2_orig.png)

![deepdream img 2](/media{{ page.id }}/dlxdd_bbhmm_2.png)

![original img 3](/media{{ page.id }}/dlxdd_bbhmm_3_orig.png)

![deepdream img 3](/media{{ page.id }}/dlxdd_bbhmm_3.png)

![original img 4](/media{{ page.id }}/dlxdd_bbhmm_4_orig.png)

![deepdream img 4](/media{{ page.id }}/dlxdd_bbhmm_4.png)

![original img 5](/media{{ page.id }}/dlxdd_bbhmm_5_orig.png)

![deepdream img 5](/media{{ page.id }}/dlxdd_bbhmm_5.png)

Also, shout out to the developers of [Bat Country](https://github.com/jrosebr1/bat-country)!  What a great python package!

## Addendum

This post is supposed to be about leveraging Unveillance's Annex component to spin up a project, but surely some of you will have questions about how to set up an environment for deep dreaming, or how to create a Slackbot.  Here are my notes:

### DeepDream Environment

1.	"New" Macbook Pro (actually, just wiped and repurposed for this very thing!)
1.	Install homebrew
1.	Install anaconda python (a much more capable version of python for doing the types of calculations deep dream does.  Fun fact: Unveillance Annex installs anaconda automatically, so I didn't need to do this.  But, I did, just for good measure, and when I set up the Annex, Unveillance skipped over its normal anaconda-install step.)
1.	Install OpenBLAS
1.	Install [CUDA](http://docs.nvidia.com/cuda/cuda-getting-started-guide-for-mac-os-x/index.html)
1.	Install [Caffe](http://caffe.berkeleyvision.org/install_osx.html)
1.	Install [Protobuf library](https://developers.google.com/protocol-buffers/)
1.	Setup git
1.	Clone [GoogleNet DNN](https://github.com/BVLC/caffe/tree/master/models/bvlc_googlenet)
1.	Get some other DNNs from [ModelZoo](https://github.com/BVLC/caffe/wiki/Model-Zoo)
1.	Install FFMpeg

### Slackbot

Code for my bot component can be found [here](https://github.com/DeepLab/deeplab_x_deepdream_slackbot).

On AWS, I'm hosting a node.js bot that serves as a data broker between slack and my personal computer.  Using Slack's Outgong Webhooks, the bot can either:

1.	take in image, and upload it to a specific dropbox folder
1.	respond to `moar [id of image]` to perform an iteration on the specified image
1.	respond to `gif [id of image]` to create a gif of an image's transformation into a deep dream.

The instance communicates with the Annex using XMPP.
