# goal:

This project is broken up into 3 sections.

First we will generate a neural network image gallery. The gallery allows you to test out different neural styles and deep dream algorithms.

Secondly we will build a slack bot 


Third, we will try and modify deep dream so that we can create our own effects.

we are going to try and modify a neural network to generate different effects.  then we make a bot that takes requests from a service we choose (slack, reddit,twitter) and generate photo callergy.  

 images will be generated using neural style and deep dream.

be able to test out different visualizations of deep dream and 


Training on these algorithms usually takes a day to several weeks depending on the training data and the parameters chosen for the models.

So we will provide pretrained models for you to use.

Register a user account at: http://www.somatic.io/user/sign_up?code=hackday 


Now you can  view any of pretrained deep dream and style transfers we have:



We have provided some sample code that runs to generate images given a folder. to run it you should have bash and curl installed.


generate_gallery.sh /path/to/folder

This should be enough for you to

# Part I: Generate a neural network image gallery

# Part II: Building a slackbot

###### Based largely on Full Stack Python's tutorial ["How to Build Your First Slack Bot with Python"](https://www.fullstackpython.com/blog/build-first-slack-bot-python.html) with adjustments made to integrate Somatic's deep learning models and for simplicity.


## What you need

* [Python 2 or 3](https://www.fullstackpython.com/python-2-or-3.html)
* [pip](https://pip.pypa.io/en/stable/) and [virtualenv](https://virtualenv.pypa.io/en/stable/) to handle Python [application dependencies](https://www.fullstackpython.com/application-dependencies.html)
* [Free Slack account](https://slack.com/) with a team on which you have API access or sign up for the [Slack Developer Hangout team](http://dev4slack.xoxco.com/)
* Official Python [slackclient](https://github.com/slackhq/python-slackclient) code library built by the Slack team
* [Slack API testing token](https://api.slack.com/tokens)

## Establishing Our Environment

Go to the terminal (or Command Prompt on Windows) and change into the directory where you want to store this project. 

Within that directory, create a new virtualenv to isolate our application dependencies from other Python projects.

```
virtualenv starterbot
```

Activate the virtualenv:

```
source starterbot/bin/activate
```

Your prompt should now look like the one in this screenshot.

![](https://www.fullstackpython.com/source/static/img/160604-simple-python-slack-bot/virtualenv-activate.png)

The official slackclient API helper library built by Slack can send and receive messages from a Slack channel. Install the slackclient library with the `pip` command:

```
pip install slackclient
```

When `pip` is finished you should see output like this and you'll be back at the prompt.

![](https://www.fullstackpython.com/source/static/img/160604-simple-python-slack-bot/pip-install-slackclient.png)

We also need to obtain an access token for our Slack team so our bot can use it to connect to the Slack API.

## Slack Real Time Messaging (RTM) API

Go to the [Slack web API page](https://api.slack.com/) and sign up to create your own Slack team. You can also sign into an existing account where you have administrative privileges.


![](https://www.fullstackpython.com/source/static/img/160604-simple-python-slack-bot/sign-in-slack.png)

After you have signed in go to the [Bot Users page](https://api.slack.com/bot-users).

![](https://www.fullstackpython.com/source/static/img/160604-simple-python-slack-bot/custom-bot-users.png)

Name your bot "starterbot" then click the “Add bot integration” button.

![](https://www.fullstackpython.com/source/static/img/160604-simple-python-slack-bot/starterbot.jpg)

The page will reload and you will see a newly-generated access token. You can also change the logo to a custom design. For example, I gave this bot the Full Stack Python logo.

![](https://www.fullstackpython.com/source/static/img/160604-simple-python-slack-bot/slack-token.png)

Click the "Save Integration" button at the bottom of the page. Your bot is now ready to connect to Slack's API.

A common practice for Python developers is to export secret tokens as environment variables. Export the Slack token with the name `SLACK_BOT_TOKEN`:

```
export SLACK_BOT_TOKEN='your slack token pasted here'
```

Nice, now we are authorized to use the Slack API as a bot.

There is one more piece of information we need to build our bot: our bot's ID. Next we will write a short script to obtain that ID from the Slack API.

## Obtaining Our Bot’s ID

It is finally time to write some Python code! We'll get warmed up by coding a short Python script to obtain StarterBot's ID. The ID varies based on the Slack team.

We need the ID because it allows our application to determine if messages parsed from the Slack RTM are directed at StarterBot. Our script also tests that our `SLACK_BOT_TOKEN` environment variable is properly set.

Create a new file named `print_bot_id.py` and fill it with the following code.

```
import os
from slackclient import SlackClient


BOT_NAME = 'starterbot'

slack_client = SlackClient(os.environ.get('SLACK_BOT_TOKEN'))


if __name__ == "__main__":
    api_call = slack_client.api_call("users.list")
    if api_call.get('ok'):
        # retrieve all users so we can find our bot
        users = api_call.get('members')
        for user in users:
            if 'name' in user and user.get('name') == BOT_NAME:
                print("Bot ID for '" + user['name'] + "' is " + user.get('id'))
    else:
        print("could not find bot user with the name " + BOT_NAME)
```

Our code imports the SlackClient and instantiates it with our `SLACK_BOT_TOKEN`, which we set as an environment variable. When the script is executed by the `python` command we call the Slack API to list all Slack users and get the ID for the one that matches the name "starterbot".

We only need to run this script once to obtain our bot’s ID.

```
python print_bot_id.py
```

The script prints a single line of output when it is run that provides us with our bot's ID.

![](https://www.fullstackpython.com/source/static/img/160604-simple-python-slack-bot/printed-bot-id.png)

Copy the unique ID that your script prints out. Export the ID as an environment variable named BOT_ID.

```
(starterbot)$ export BOT_ID='bot id returned by script'
```

The script only needs to be run once to get the bot ID. We can now use that ID in our Python application that will run StarterBot.

## Coding Our StarterBot

We've got everything we need to write the StarterBot code. Create a new file named `starterbot.py` and include the following code in it.

```
import os
import time
from slackclient import SlackClient
```

The `os` and `SlackClient` imports will look familiar because we used them in the `print_bot_id.py` program.

With our dependencies imported we can use them to obtain the environment variable values and then instantiate the Slack client.

```
# starterbot's ID as an environment variable
BOT_ID = os.environ.get("BOT_ID")

# constants
AT_BOT = "<@" + BOT_ID + ">"
EXAMPLE_COMMAND = "do"

# instantiate Slack & Twilio clients
slack_client = SlackClient(os.environ.get('SLACK_BOT_TOKEN'))
```

The code instantiates the `SlackClient` client with our `SLACK_BOT_TOKEN` exported as an environment variable.

```
if __name__ == "__main__":
    READ_WEBSOCKET_DELAY = 1 # 1 second delay between reading from firehose
    if slack_client.rtm_connect():
        print("StarterBot connected and running!")
        while True:
            command, channel = parse_slack_output(slack_client.rtm_read())
            if command and channel:
                handle_command(command, channel)
            time.sleep(READ_WEBSOCKET_DELAY)
    else:
        print("Connection failed. Invalid Slack token or bot ID?")
```

The Slack client connects to the Slack RTM API WebSocket then constantly loops while parsing messages from the firehose. If any of those messages are directed at StarterBot, a function named `handle_command` determines what to do.

Next add two new functions to parse Slack output and handle commands.

```
def handle_command(command, channel):
    """
        Receives commands directed at the bot and determines if they
        are valid commands. If so, then acts on the commands. If not,
        returns back what it needs for clarification.
    """
    response = "Not sure what you mean. Use the *" + EXAMPLE_COMMAND + \
               "* command with numbers, delimited by spaces."
    if command.startswith(EXAMPLE_COMMAND):
        response = "Sure...write some more code then I can do that!"
    slack_client.api_call("chat.postMessage", channel=channel,
                          text=response, as_user=True)


def parse_slack_output(slack_rtm_output):
    """
        The Slack Real Time Messaging API is an events firehose.
        this parsing function returns None unless a message is
        directed at the Bot, based on its ID.
    """
    output_list = slack_rtm_output
    if output_list and len(output_list) > 0:
        for output in output_list:
            if output and 'text' in output and AT_BOT in output['text']:
                # return text after the @ mention, whitespace removed
                return output['text'].split(AT_BOT)[1].strip().lower(), \
                       output['channel']
    return None, None
```

The `parse_slack_output` function takes messages from Slack and determines if they are directed at our StarterBot. Messages that start with a direct command to our bot ID are then handled by our code - which is currently just posts a message back in the Slack channel telling the user to write some more Python code!

Here is how the entire program should look when it's all put together:

```
import os
import time
from slackclient import SlackClient


# starterbot's ID as an environment variable
BOT_ID = os.environ.get("BOT_ID")

# constants
AT_BOT = "<@" + BOT_ID + ">"
EXAMPLE_COMMAND = "do"

# instantiate Slack & Twilio clients
slack_client = SlackClient(os.environ.get('SLACK_BOT_TOKEN'))


def handle_command(command, channel):
    """
        Receives commands directed at the bot and determines if they
        are valid commands. If so, then acts on the commands. If not,
        returns back what it needs for clarification.
    """
    response = "Not sure what you mean. Use the *" + EXAMPLE_COMMAND + \
               "* command with numbers, delimited by spaces."
    if command.startswith(EXAMPLE_COMMAND):
        response = "Sure...write some more code then I can do that!"
    slack_client.api_call("chat.postMessage", channel=channel,
                          text=response, as_user=True)


def parse_slack_output(slack_rtm_output):
    """
        The Slack Real Time Messaging API is an events firehose.
        this parsing function returns None unless a message is
        directed at the Bot, based on its ID.
    """
    output_list = slack_rtm_output
    if output_list and len(output_list) > 0:
        for output in output_list:
            if output and 'text' in output and AT_BOT in output['text']:
                # return text after the @ mention, whitespace removed
                return output['text'].split(AT_BOT)[1].strip().lower(), \
                       output['channel']
    return None, None


if __name__ == "__main__":
    READ_WEBSOCKET_DELAY = 1 # 1 second delay between reading from firehose
    if slack_client.rtm_connect():
        print("StarterBot connected and running!")
        while True:
            command, channel = parse_slack_output(slack_client.rtm_read())
            if command and channel:
                handle_command(command, channel)
            time.sleep(READ_WEBSOCKET_DELAY)
    else:
        print("Connection failed. Invalid Slack token or bot ID?")
```

Now that all of our code is in place we can run our StarterBot on the command line with the `python starterbot.py` command.

![](https://www.fullstackpython.com/source/static/img/160604-simple-python-slack-bot/starterbot-running.png)

In Slack, create a new channel and invite StarterBot or invite it to an existing channel.

![](https://www.fullstackpython.com/source/static/img/160604-simple-python-slack-bot/create-channel.png)

Now start giving StarterBot commands in your channel.

![](https://www.fullstackpython.com/source/static/img/160604-simple-python-slack-bot/working-starterbot.png)

As it is currently written above in this tutorial, the line `AT_BOT = "<@" + BOT_ID + ">"` does not require a colon after the "@starter" (or whatever you named your particular bot) mention. Previous versions of this tutorial did have a colon because Slack clients would auto-insert the ":" but that is no longer the case.







# Part III: Modify deep dream










# Why Deep Learning Image Transformation Matters For Advertisers, Marketers, and Designers
### Machine learning will become a standard part of the creator toolkit. 
###It's not equivalent to a human expert, but it's fast and scalable.

## What deep learning will enable you to do.

##### Build Better Products

*People love personalization of products and featuring themselves or friends.*

One of a kind products give customers a way to stand out and express themselves. We see these features in products like NIKEiD. But, even in this case the option set is limited. With deep learning to personalization is endless. Perhaps you really, really, really, love your cat.
![](https://d17oy1vhnax1f7.cloudfront.net/items/3L0G0o3c3V3R08211S3f/Image%202016-11-15%20at%206.00.59%20PM.png?v=4db6f583)

Content that features the customer is a big part of consumer facing custom printing. Printing photos of your friends and family, buttons for your high school election, or posters for your band are just a few cases where you might want to apply some sort of deep learning filter. Somatic is currently being utilized on one site that provides easy custom printing and framing.
![](https://d17oy1vhnax1f7.cloudfront.net/items/0G3M1D1u3w0i1k1R3w0X/Image%202016-11-15%20at%204.32.56%20PM.png?v=2d9bdc72)



##### Create Engaging Ads 

*It's really hard to stand out in the digital space. And, even if you catch someone's eye, how can you get them to think about your brand and shere their interest with friends. *

Snapchat has created a variety of fun and engaging ads. Similar branded and personalized video content is being created cross platform by companies like Vivoom. Machine learning models are used in many of these applications today, and going forward will likely expand what is possible. 

![](https://d17oy1vhnax1f7.cloudfront.net/items/3X1M390k0L0Q1S1h1M1e/Image%202016-11-15%20at%204.27.42%20PM.png?v=f85a7e80)

As another example, we have made a simple filter that could be branded for any coffee shop on Somatic. It's early, you are watching the sunrise but you have not had your coffee yet and this is all you can think about. A styled image is a much more fun way to share that condition than text.

![](https://d17oy1vhnax1f7.cloudfront.net/items/1Y112d2J1e203W1r3r0i/Image%202016-11-15%20at%204.14.03%20PM.png?v=23bae0c3)


##### Accelerate Content Creation

*Custom models can accelerate the creation process by allowing new images styled with little work, and making it easy to control the style shown via web languages.*

For example, espnfc.us often uses artistic images to tell stories. While no fast neural style model can match human created content, for prototyping or applications where a desiger isn't justified, neural style can provide a new possibility. Additionally, the API will make it easy to switch styles on the fly.

![](https://d17oy1vhnax1f7.cloudfront.net/items/0E0k3b2S3U0J2t0v2V1q/Image%202016-11-15%20at%203.58.56%20PM.png?v=bc63df79)


##### Nurture Your Cutting Edge Brand

Being on the cutting edge is important for an agency's brand as well as the brand of individual creatives. Given the growing influence of machine learning, demonstrated knowledge can help build an influencer image. Whether it's chat bots, neural style filters, image search, AlphaGo, or self driver cars, machine learning is driving significant change in the economy. 

* [MIT Nightmare Machine](http://nightmare.mit.edu/)

* [Deep Dream](https://github.com/google/deepdream)


# How Bots Change The Game For Image Transformations

Bots can make deep learning creations accessible from anywhere, which will opens up monetization pathways.

Unlike many of the popular apps today, that exist in a closed system, open APIs with social buts will lower the level of friction to create a marketing campaign and drive conversions. 

> For example, a printer could create a twitter bot that restyles images. It could present those images in a frame and link directly to the purchase page from the reply tweet.

Additionally, image filters can be used to add a more personal tone to interactions. While it is automated, good filters feel like a human way to interact. Thus, the presentation feels less like a scheduled campaign and more like a 1 on 1 conversation.


# How Somatic Can Help

##### We Makes Training & Deployment Easy
Although training remains an art, at Somatic we have built a system to make training new models very easy. You can get started training in just a few steps.

In this exercise, we will demonstrate how easy it is to get a new model up and running.

Additionally, we have built out an appended URL api, that takes almost all the work off of the developer for simple applications and customer development and testing.
