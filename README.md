# whatsbot
A Whatsapp bot built with Python Flask + Twilio API 

## Requirements

  `Python 3.6 or newer:` If your operating system does not provide a Python interpreter, go to python.org and download from the link [here!](https://www.python.org/downloads/)<br/>
  `Flask:` We will create a web application that responds to incoming WhatsApp messages with it. We will install Flask through Python package installer<br/>
  `ngrok:` We will use this handy utility to connect the Flask application running on your system to a public URL that Twilio can connect to. This is necessary for the development version of the chatbot because your computer is likely behind a router or firewall, so it isn’t directly reachable on the Internet. If you don’t have ngrok it installed, you can [download a copy for Windows, MacOS or Linux](https://ngrok.com/download).<br/>
  `Twilio:` If you are new to Twilio create a free account now. You can review the features and limitations of a free Twilio account.<br/>
  `Smartphone` with an active phone number and WhatsApp installed.<br/>

### Twilio Configuration

Signup with a free account in [twilio.com](https://twilio.com) and add your phone number which has whatsapp installed. 

Twilio provides a WhatsApp sandbox where you can easily develop and test your application. Once your application is complete you can request production access for your Twilio phone number, which requires approval by WhatsApp. 

Let’s connect your smartphone to the sandbox. From your Twilio Console, select Programmable Messaging, then click on "Try it Out" and finally click on Try WhatsApp. The WhatsApp sandbox page will show you the sandbox number assigned to your account, and a join code.

Or go to this [link](https://www.twilio.com/console/sms/whatsapp/learn) directly.

### Create a Python virtual environment

Use a virtual environment to manage the dependencies for your project, both in development and in production.

What problem does a virtual environment solve? The more Python projects you have, the more likely it is that you need to work with different versions of Python libraries, or even Python itself. Newer versions of libraries for one project can break compatibility in another project.

For those of you following the tutorial on Windows, enter the following commands in a command prompt window:

```
$ mkdir whatsbot
$ cd whatsbot
$ python3 -m venv whatsbot-venv
$ whatsbot-venv\Scripts\activate
(whatsbot-venv) $ pip install twilio flask requests
```

The last command uses `pip, the Python package installer`, to install the three packages that we are going to use in this project, which are:

1. The `Flask framework`, to create the web application<br/>
2. The `Twilio Python Helper library`, to work with the Twilio APIs<br/>
3. The `Requests` package, to access third party APIs<br/>

## Create a Flask bot Service

Lets start with `Hello Madurai` code. Create a file called `app.py` inside the `whatsbot` folder.

```
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, Madurai!'
```    

For this tutorial we are going to build an extremely simple chatbot that recognizes two keywords in messages sent by the user and reacts to them. If the user writes anything that contains the word “quote”, then the chatbot will return a random famous quote. If instead the message has the word “cat”, then a random cat picture will be returned. If both “quote” and “cat” are present in the message, then the bot will respond with a quote and a cat picture together.

### Webhook

A Webhook is basically a way to be notified when an event has occurred, usually not due to a direct action from your application. For example, say I had created an application for my restaurant that used the Zomato API to track when people checked in.

The Twilio API for WhatsApp uses a webhook to notify an application when there is an incoming message. Our chatbot application needs to define an endpoint that is going to be configured as this webhook so that Twilio can communicate with it.

With the Flask framework it is extremely easy to define a webhook. Below is a skeleton application with a webhook definition. Don’t worry about copying this code, I will first show you all the different parts of the implementation and then once you understand them I’ll show you how they are all combined into a working application.

with this, lets start writing our bot

```
from flask import Flask
app = Flask(__name__)

@app.route('/bot’, methods=['POST'])
def bot():
    # add webhook logic here and return a response

if __name__ == '__main__':
    app.run()
```    

Open your terminal. Go to whatsbot folder. And run

`python3 app.py`



