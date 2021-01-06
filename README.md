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
(whatsbot-venv) $ python3 -m pip install twilio flask requests
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

If you are on Windows, the environment variable syntax depends on command line interpreter. On Command Prompt:

```
C:\path\to\app>set FLASK_APP=hello.py
```
And on PowerShell:
```
PS C:\path\to\app> $env:FLASK_APP = "hello.py"
```
Alternatively you can use python -m flask:
```
$ export FLASK_APP=hello.py
$ python3 -m flask run
 * Running on http://127.0.0.1:5000/
```
This launches a very simple builtin server, which is good enough for testing but probably not what you want to use in production. For deployment options see Deployment Options.

Now head over to http://127.0.0.1:5000/, and you should see your hello world greeting.

In the command prompt, you will see

```
(whatsbot-venv) C:\src\python\whatsbot>set FLASK_APP=app.py

(whatsbot-venv) C:\src\python\whatsbot>python3 -m flask run
 * Serving Flask app "app.py"
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
127.0.0.1 - - [05/Jan/2021 22:16:16] "[37mGET / HTTP/1.1[0m" 200 -
```

If you are not familiar with the Flask framework, its documentation has a [quick start](https://flask.palletsprojects.com/en/1.1.x/quickstart/) section that should bring you up to speed quickly. If you want a more in-depth learning resource then check [Flask Mega-Tutorial](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world).

For this tutorial we are going to build an extremely simple chatbot that recognizes two keywords in messages sent by the user and reacts to them. If the user writes anything that contains the word “quote”, then the chatbot will return a random famous quote. If instead the message has the word “cat”, then a random cat picture will be returned. If both “quote” and “cat” are present in the message, then the bot will respond with a quote and a cat picture together.

### Webhook

A Webhook is basically a way to be notified when an event has occurred, usually not due to a direct action from your application. For example, say I had created an application for my restaurant that used the Zomato API to track when people checked in.

The Twilio API for WhatsApp uses a webhook to notify an application when there is an incoming message. Our chatbot application needs to define an endpoint that is going to be configured as this webhook so that Twilio can communicate with it.

With the Flask framework it is extremely easy to define a webhook. Below is a skeleton application with a webhook definition. Don’t worry about copying this code, I will first show you all the different parts of the implementation and then once you understand them I’ll show you how they are all combined into a working application.

with this, lets start writing our bot

```
from flask import Flask
app = Flask(__name__)

@app.route('/bot', methods=['POST'])
def bot():
    # add webhook logic here and return a response

if __name__ == '__main__':
    app.run()
```    

Open your terminal. Go to whatsbot folder. And run

`python3 -m flask run`

The important thing to keep in mind about the code above is that the application defines a `/bot` endpoint that listens to POST requests. Each time an incoming message from a user is received by Twilio, they will in turn invoke this endpoint. The body of the function bot() is going to analyze the message sent by the user and provide the appropriate response.

### Messages and Responses

The first thing we need to do in our chatbot is obtain the message entered by the user. This message comes in the payload of the POST request with a key of ’Body’. We can access it through Flask’s request object:

```
from flask import request
incoming_msg = request.values.get('Body', '').lower()
```
Since we are going to perform some basic language analysis on this text, I have also converted the text to lowercase, so that we don’t have to worry about all the different ways a word can appear when you introduce case variations.

The response that Twilio expects from the webhook needs to be given in TwiML or Twilio Markup Language, which is an XML-based language. The Twilio helper library for Python comes with classes that make it easy to create this response without having to create XML directly. Below you can see how to create a response that includes text and media components:

```
from twilio.twiml.messaging_response import MessagingResponse

resp = MessagingResponse()
msg = resp.message()
msg.body('this is the response text')
msg.media('https://example.com/path/image.jpg')
```
Note how to return an image Twilio expects a URL that points to it instead of the actual image data.

### Chatbot logic

For the actual chatbot logic we are going to use a very simple, yet surprisingly effective approach. What we are going to do is search the incoming messages for the keywords ’quote’ and ’cat’. Here is the basic structure of the chatbot:
```
    responded = False
    if 'quote' in incoming_msg:
        # add a quote to the response here
        responded = True
    if 'cat' in incoming_msg:
        # add a cat picture to the response here
        responded = True
    if not responded:
        # return a generic response here
```

With this simple structure we can detect references to quotes and/or cats and configure the Twilio response object accordingly. The responded boolean is useful to track the case where the message does not include any of the keywords we are looking for, and in that case offer a generic response.

### Third-Party APIs

To supply the chatbot with original quotes and cat pictures I’m going to use two publicly available APIs. For famous quotes, I’ve chosen the Quotable API from Luke Peavey. A GET request to https://api.quotable.io/random returns a random quote out of a pool of 1500 of them in JSON format.

For cat pictures I’m going to use the Cat as a Service API from Kevin Balicot. This is an extremely simple API, the https://cataas.com/cat URL returns a different cat image every time (you can test it out by pasting this URL in the browser’s address bar and then hitting refresh to get a new cat picture). This is actually very handy because as I mentioned above, Twilio wants the image given as a URL when preparing the TwiML response.

## Putting Everything Together

Now you have seen all the aspects of the chatbot implementation, so we are ready to integrate all the pieces into the complete chatbot service. You can copy the code below into a app.py file:

```
from flask import Flask, request
import requests
from twilio.twiml.messaging_response import MessagingResponse

app = Flask(__name__)


@app.route('/bot', methods=['POST'])
def bot():
    incoming_msg = request.values.get('Body', '').lower()
    resp = MessagingResponse()
    msg = resp.message()
    responded = False
    if 'quote' in incoming_msg:
        # return a quote
        r = requests.get('https://api.quotable.io/random')
        if r.status_code == 200:
            data = r.json()
            quote = f'{data["content"]} ({data["author"]})'
        else:
            quote = 'I could not retrieve a quote at this time, sorry.'
        msg.body(quote)
        responded = True
    if 'cat' in incoming_msg:
        # return a cat pic
        msg.media('https://cataas.com/cat')
        responded = True
    if not responded:
        msg.body('I only know about famous quotes and cats, sorry!')
    return str(resp)


if __name__ == '__main__':
    app.run()
```

### Let's test it!

Are you ready to test the chatbot? After you copy the above code into the app.py file, start the chatbot by running python bot.py, making sure you do this while the Python virtual environment is activated. The output should be something like this:

```
(whatsbot-venv) C:\src\python\whatsbot>python3 app.py
 * Serving Flask app "app" (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

The service is now running as a private service on port 5000 inside your computer and will sit there waiting for incoming connections. To make this service reachable from the Internet we need to use ngrok.

### Installing ngrok

For windows users:

1. UNZIP TO INSTALL
On Windows, just double click ngrok.zip.

2. CONNECT YOUR ACCOUNT
Running this command will add your authtoken to your ngrok.yml file. Connecting an account will list your open tunnels in the dashboard, give you longer tunnel timeouts, and more. Visit the dashboard to get your auth token. Once you signup, you will get your auth token. 
```
./ngrok authtoken <your_auth_token>
```
3. FIRE IT UP
Try running it from the command line:
```
./ngrok help
```

Open a second terminal window and run `ngrok http 5000` to allocate a temporary public domain that redirects HTTP requests to our local port 5000. The output of ngrok should be something like this:

```
Session Status                online  
                                                                                  
Account                       Karthikeyan NG (Plan: Free)                                                               
Version                       2.3.35                                                                                    
Region                        United States (us)                                                                        
Web Interface                 http://127.0.0.1:4040                                                                     
Forwarding                    http://ce5341ebfa4e.ngrok.io -> http://localhost:5000                                     
Forwarding                    https://ce5341ebfa4e.ngrok.io -> http://localhost:5000 

Connections                   ttl     opn     rt1     rt5     p50     p90                                                                             
				                      0       0       0.00    0.00    0.00    0.00 
```

Note the lines beginning with “Forwarding”. These show the public URL that ngrok uses to redirect requests into our service. What we need to do now is tell Twilio to use this URL to send incoming message notifications.

Go back to the Twilio Console, click on Programmable Messaging, then on Settings, and finally on WhatsApp Sandbox Settings. Copy the https:// URL from the ngrok output and then paste it on the “When a message comes in” field. Since our chatbot is exposed under the /bot URL, append that at the end of the root ngrok URL. Make sure the request method is set to HTTP Post. Don’t forget to click the red Save button at the bottom of the page to record these changes.

Now you can start sending messages to the chatbot from the smartphone that you connected to the sandbox. You can type any sentences that you like, and each time the words “quote” and “cat” appear in messages the chatbot will invoke the third party APIs and return some fresh content to you. 

Keep in mind that when using ngrok for free there are some limitations. In particular, you cannot hold on to an ngrok URL for more than 8 hours, and the domain name that is assigned to you will be different every time you start the ngrok command. You will need to update the URL in the Twilio Console every time you restart ngrok.

Enjoy!

For any queries, ping: 

-- 
Karthikeyan NG

Amazon: https://www.amazon.com/s?k=karthikeyan+ng <br/>
Github: http://github.com/intrepidkarthi<br/>
LinkedIn: https://linkedin.com/in/intrepidkarthi<br/>
Twitter: https://twitter.com/intrepidkarthi<br/>
Email: intrepidkarthi@gmail.com
_________________________________________________


