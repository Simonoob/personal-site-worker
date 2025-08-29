---
title: 'I built my first machine learning model'
description: "I've started to dip my toes into deep learning with Python. This article is about how I built an image classification model with fastai."
pubDate: '2023-09-14'
---
I've started studying deep learning using the _[Practical Deep learning for Coders](https://course.fast.ai/)_ course over at Fastai. I have no background in ML or math, but their example-first approach allows you to get demos up extremely quickly.
The course teaches you DL using their fastai library.
Thanks to these resources, I was able to build an impressive demo app in only 2 days.

## What I built
I created an image classification model that can recognize 150 different pokemons (the whole first generation).
I also made a UI to interact with the model and uploaded everything to a free codespace on Hugging Face.

<script
	type="module"
	src="https://gradio.s3-us-west-2.amazonaws.com/3.41.2/gradio.js"
></script>

<gradio-app src="https://simonoob-pokemonclassifier.hf.space"></gradio-app>
[_you can also play around with it on Hugging Face_](https://huggingface.co/spaces/simonoob/pokemonClassifier).


## The tools
The backbone of the project is the fastai library. It provides a high level API to interact with [PyTorch](https://pytorch.org/).
The other major tool I used is the course mentioned above, which teaches you how to use the fastai library.
Additionally, I've used the [Hugging Face Dataset collection](https://huggingface.co/datasets) to get my data and their [Spaces tool](https://huggingface.co/spaces) to host the model and its UI.
Talking about UI, It's made with [Gradio](https://www.gradio.app/).

## The building process
Building the project has 3 main components to it:
1. Collecting and validating the training data
2. Creating and training the deep learning model
3. Creating the UI and publishing everything for other people to use

Thanks to the tools above, solving these problems is simple and accessible.

### 1. Sourcing and validating the training data
Let's define our training data: we need a sizable number of images for each pokemon, and we need to know the correct name for each image.
Additionally, the data should reflect the type of images that we'll feed to the model when using it after training.

A great starting point is this [pokemon classification dataset](https://huggingface.co/datasets/keremberke/pokemon-classification) on Hugging Face. It features more than 4000 images, already validated and correctly labeled.

However, while testing I've noticed that the images didn't reflect enough what I was using in production.
To fix this, in the following notebook the default dataset is augmented with new images from the web.

<script src="https://gist.github.com/Simonoob/fccab35b27c858bcc2668c432680d92d.js"></script>


### 2. Training the model
Ironically, this is the easiest part thanks to the fastai library. Instead of starting from scratch, we can get a pre-trained neural network, and fine tune it by adding additional layers to it.

The pre-trained model used is a general-purpose convolutional neural network, pte-trained on a variety of shapes and objects. The model name is rasnet-`n`, with `n` being the number of layers in the neural network - the more the layers the better the accuracy, but training takes more time. 

The training principle is simple: we ask the model to predict the label, and we check the prediction against the provided label. After doing so for all the images in the training set, we check the model accuracy using our validation set.
We repeat this process in cycles until we get an acceptable accuracy rate. Each cycle adds a layer to our pre-trained model.

Once the training is done, we export the model as a `.pkl` file (a way to save the obj state).

Note: this step might take some time, so we are not showing running it here, however, you can [run it in colab](https://colab.research.google.com/drive/1eElIEUX12jzVSewF013VLMGmnpdAeBCn?usp=sharing) (make sure to include also the previous notebook, otherwise you'll get errors from missing references).

<script src="https://gist.github.com/Simonoob/2de8d0485272a10ed5e18f5b48c17aa6.js"></script>


### 3. Create the UI and host it
Creating the UI is extremely easy. We can use widgets provided by Gradio to interact with the model, and we can host the final product on Hugging Face.

The basic process is simple: you create a repo in Hugging Face spaces, you put all your assets there (the model file, demo images etc.) along with a script to run your app.

The following notebook creates a Python script that functions as a build process to piece together the app.

<script src="https://gist.github.com/Simonoob/383cf08140556ed320938d08c1dad4bb.js"></script>


And that's it. The model is trained and integrated into an app for everyone to use.
It's amazing how far machine learning and deep learning resources have come.
