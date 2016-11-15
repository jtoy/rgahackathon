storyteller:

given an image, AI system will generate a story around that image.

the original research paper trains a romance novel generator. so you give it an image and romance novels appear, test out the model here:

keep in mind that due to the size of this model,it takes ~5 minutes to generate a story.

somatic and RGA have trained a model on the cannes lions submission data.
 So when given an image, we generate a fake cannes lion submission. We trained many different models to find the best hyper parameters.


Here you can see the original research paper:
https://arxiv.org/abs/1506.06726

higher level blog post describing how it works:

http://www.somatic.io/blog/how-neural-storyteller-works

video describing neural storyteller:
https://www.youtube.com/watch?v=HethZTW-39s

can we make a compelling tool to engage users to play with this.
how can we make a compelling storyline around this to engage users.


# Goals:
  * learn how to create a problem statement and an execution plan
  * familiarize yourself with the basic concepts of neural storyteller and think about what is possible
  * implement cannes lions storyteller into a app, website, or bot to create an engaging user experience.



# Notes on Training
Training your own custom storyteller is out of scope for today's project, it typically takes 3-4 days to train a model.  If you have access to a large dataset(at least 20 MB) and want to train your own model, you can signup at htpt://somatic.io and contact jason@somatic.io to train.


# Getting Started:

To use the pretrained models, register a user account at: http://www.somatic.io/user/sign_up?code=hackday

you can test out the model here:
http://www.somatic.io/models/gZDgvqnx
you can view the API docs here:
http://www.somatic.io/models/advanced/gZDgvqnx

 Note that the model is async, You can get notified of a finished request using webhooks. To use webhooks in development mode, it is highly recommended to use ngrok which creates a a public proxy that routes request outside to your local development machine.
The api uses http. Upload an image through the api and then you will get results several minutes later.
