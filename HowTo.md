##Goal: 
create simple wireless sensor network

##Keywords: 
GET, POST, AJAX, JS, XML, JSON, Django channels, Websocket, Node.js

##Questions:
Can I use the light weight server that comes with django and access it remotely ?Start by breaking down the problem into smaller goals

##System components:
###(a) Server side.

1. Django
        - offers a complete web framework. Takes care of user authentication, content administration, site maps, RSS feeds ..etc.
        - provides both, admin and public interface.
        - built-in sqlite DB, and can easily connect to other DB’s.
        - built-in a light web server, and can easily connect to other production servers such as: Apache
        - based on simple concept of requests and responses: the browser makes a request, Django calls a view, which returns a response that’s sent back to the browser
        - In short, Django saves all the hassles of repeated tasks of common websites, and it is written in well-structured way (i.e. ready-made components to use).

        References: 
        - Good tutorials:  https://tutorial.djangogirls.org/en/
                                    https://www.tutorialspoint.com/django/index.htm
                                    https://docs.djangoproject.com/en/1.10/intro/

2. Django-Channels 
        - unlike standard Django, Channels replaces the request/response cycle with messages that are sent across channels. 
        - Channels allows Django to support WebSockets in a way that’s very similar to traditional HTTP views.
        - Channels also allow for background tasks that run on the same servers as the rest of Django. 
        - HTTP requests continue to behave the same as before, but also get routed over channels.

        References: 
        - https://blog.heroku.com/in_deep_with_django_channels_the_future_of_real_time_apps_in_django
        - http://channels.readthedocs.io/en/latest/
        - https://medium.com/@johngrant/raspberry-pi-and-django-channels-8d5cddb36226#.dgg0nosw2
        

3. Bootstrap
        - the most popular HTML, CSS, and JS framework for developing responsive, mobile first projects on the web.
        - makes your website look nice.
        http://getbootstrap.com/


4. HighCharts
        - excellent for charts: http://www.highcharts.com/
        - Built based on JS and JQuery. 


###(b) Client side.
###(c) connections.

##Big Picture:
This article tries to explain one way to build a simple wireless sensor network. At the end of this tutorial, you should be able to send data from multiple sensors to a server, then show these acquired data in a nice dashboard displayed in a public web page.

The sensor (client) sends the data to the server using Websocket protocol. The server uses Django channels; an extension to Django framework which adds a new layer to support Websocket handling. After receiving the data at the server side, Django and some Javascript packages will take care of web development part.

## Implementation:
### (1): Django
#### Basic Django setup:
First, since Django is based on Python, make sure that Python is installed in your server.
Next install virtual environment. The idea behind that is to isolate your project, and all installed package from the rest of the system. This insures no conflict with installed package in your system and ease the task of changing some packages within your project. 
Here are the steps:

```
$ pip install virtualenv    //install virtualenv package
$ mkdir myproject           // make new directory 
$ cd my project             
$ virtualenv . -p python2.7 // start new virtual environment that uses Python 2.7
$ source bin/activate       // to activate this environment 
```
Now, you have new environment ready. Install latest pip, Django, and start new Django project.
```
(myproject) ~$ pip install --upgrade pip
(myproject) ~$ pip install django~=1.10.3  // or the latest
(myproject) ~$ django-admin.py startproject myproject
```
After starting the new project: myproject, the directory structure will look like:
```
myproject/
├───manage.py
└───myproject/
        settings.py
        urls.py
        wsgi.py
        __init__.py
```
* The first `myproject/` is the root directory, a container for your project.
* `manage.py:` A command-line utility that lets you interact with this Django project in various ways. For example, you can run a server by typing `manage.py runserver`.
* The inner *myproject/*: contains the basic files of the project, here you can configure the global settings, define URLs, etc.
* At the *settings.py*: here we add a reference to any new app, middleware, a reference to the used DB and other things.
* `wsgi.py`: An entry-point for WSGI-compatible web servers to serve your project. For example, Django is shipped with light weight server, if instead, you need to use an alternative, then you need to specify the configuration in this file.

Django should work now !, change the directory to the outer `myproject`, then run: 
```
$ python manage.py runserver
```
You just run the server locally at http://127.0.0.1:8000/, check if it works by typing this IP address in your browser. If it didn't work, then you may need to add '127.0.0.1' to your allowed hosts at the setting file. i.e. at myproject/settings.py add:
```
ALLOWED_HOSTS = ['127.0.0.1']
```
#### Creating a new App:
```
Projects vs. apps [https://docs.djangoproject.com/en/1.10/intro/tutorial01/]

What’s the difference between a project and an app? An app is a Web application that does something – e.g., a Weblog system, a database of public records or a simple poll app. A project is a collection of configuration and apps for a particular website. A project can contain multiple apps. An app can be in multiple projects.
```
Now, we had the Django ready, we'll start a new App that is specific to our purpose: responsible of handling the task of reading data from a sensor at the client side. This App will take care of the communication between the client and server as well as recieving and parsing data.
To start a new App, at the same directory as manage.py and type:
```
$ python manage.py startapp sensorReading
```
This will create a new directory called sensorReading, the structure of the project will look like this now:
```
myproject/
├───manage.py
├───sensorReading/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    views.py
└───myproject/
        settings.py
        urls.py
        wsgi.py
        __init__.py
```
before we start working on this App, register the App in `myproject/settings.py` under INSTALLED_APPS. 
Notice the files inside the `sensorReading` App directory, we will focus on the main three files, which represents main blocks on Django: Views, Models, Admin.

#### Django View:
In `view.py`, we define the logic of interaction with the user. It specifies how we are handling new request, and returning a response by rendering web page. views are Python functions. Starting by simple example, in `sensorReading/view.py` 
```py
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world. You're at the sensorReading index.")
```

#### Django URLs:
To make it work, we need to map it to a URL.
To have the project well-organized, we will seperate our configurations of URLs in two files.
* the first one: the main `url.py` file which is located already at `/myproject/myproject/urls.py`
* the second file: will have same file name, but isolated inside our App: `sensorReading`. This file does not exist,  so create new file `/myproject/sensorReading/urls.py`.
The purpose of having two files is to keep our project as an independant blocks, if we delete our new App, nothing will be affected. Or if we add new App, we just need to reference its urls in the main `/myproject/myproject/urls.py` file. This will be clear when we edit the two files now.
First edit the main */myproject/myproject/urls.py* as follows: 

```py
from django.conf.urls import include, url
from django.contrib import admin

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'', include('sensorReading.urls')),
]
```
inside the urlpattern, the two line codes are written in *regular expression* syntax, please read about for more details. The main idea here is `r` means this a regular expression `^` means start with, i.e. when you encounter a link that has *http://127.0.0.1:8000/admin/*, then redirect it to admin.site.urls. Likewise, in the second line, we have `r''`, redirect everything that comes into *http://127.0.0.1:8000/* to sensorReading.urls and look for further instructions there, notice we have an empty argument after 8000/, unlike with admin because we are having empty string: `r''`.

Now, edit `/myproject/sensorReading/urls.py` as follows:
```py
from django.conf.urls import url
from . import views

urlpatterns = [
    url(r'^$', views.index, name='index'),
]
```
couple of things to notice here: from . means from the same directory. `r'^$'` look for empty string. `views.index`: redirect any request to index function inside the `view.py` file. 
Now, if you browser to: `http://127.0.0.1:8000` , you should see: "Hello, world. You're at the sensorReading index." 

#### Django Models:
#### Django Admin:

#### Working with HTML and templates:
Now to make things more practical and reliable, instead of rendering things directly in `views.py`, we will edit this file such that we redirect things to templates directory where all our html files should reside.

For that, create new folder called template, inside it create another folder with the App name, and throw all the html files there (for now create a new file `index.html`). Having this order will make things convient and reliable in your project as you'll notice later.
The project directory will look like this:
```
myproject/
├───manage.py
├───sensorReading/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    views.py
└───myproject/
        settings.py
        urls.py
        wsgi.py
        __init__.py
├───templates/
         sensorReading/
                index.html
```
move what you write in `view.py` to `index.html`:
```html
<html>
    <p>Hello, world. You're at the sensorReading index</p>
</html>
```
we also need to change the code in `views.py` a bit:
```py
from django.shortcuts import render

def index(request):
    return render(request, 'sensorReading/index.html', {})
```
if you now browse to `http://127.0.0.1:8000`, you should get same message as before: `"Hello, world. You're at the sensorReading index."`.


### (2): Django-Channels
#### Why Django-Channels?
Django is based on the classic concept of Http request/reponse. Our application here requires real-time server that can handle data streamed from various sensors. To meet this requirement, one can use AJAX or other technologies. Channels is based on Websocket; which is a communicaion protocol, providing full-duplex communication channels over a single TCP connection. It suits well for real-time applications allowing for messages to be passed back and forth while keeping the connection open.

Without channels, django handles request/response as illustrated below:
![alt text](/path/img.jpg)

When introducing channels, the architecture looks as:
![alt text](/path/img.jpg)

proFor more information, please refer to the references.

#### Channel setup
1. In your same environment, first install channels:
```
$ pip install channels
```
then add 'channels' to the INSTALLED_APPS in the project setting file.

2. Next, we need to setup a channel layer
Previously, we were using in-memory channel layer implementation as our default channel layer. This just stores all the channel data in a dict in memory. Django runs everthing in one process along with WSGI server. Channels, on the other hand makes Django able to run on multi-processes. In other words, now, we are running one or more interface servers, and one or more worker servers, connected by a channel layer. For example, we can have different “interface servers”, and each one will service a different type of request - one might do both WebSocket and HTTP requests, while another might act as an SMS message gateway. To achieve that, we need to use a channel layer that supports cross-process. The recommended option by Channels is to use: Redis backend. So first, install it:
```
$ pip install asgi_redis
```
then in the setting file, tell Django to use it:
```py
CHANNEL_LAYERS = {
    "default": {
        "BACKEND": "asgi_redis.RedisChannelLayer",
        "CONFIG": {
            "hosts": [os.environ.get('REDIS_URL', 'redis://localhost:6379')],
        },
        "ROUTING": "sensorReading.routing.channel_routing",
    },
}
```

3. Channel routing
Notice that in the settings file, in our channel layer settings, we specified channel routing "ROUTING" to follow a mapping defined in file called routing.py within our App: sensorReading. Thus let's create a new file with that name inside our App (i.e. `routing.py`). In that file add the following:
```py
channel_routing = {
}
```
Now, it is empty, but we will connect it to Websocket later on.

4. Run interface servers
interface servers are the processes that do the work of taking incoming requests and loading them into the channels system. WSGI does not support WebSockets, long-poll HTTP requests and other Channels features, for that we need to run a standard like ASGI interface server. Django channels is shipped with an interface server called Daphne. To run Daphne, it just needs to be supplied with a channel backend.
Define the new handler that overrides the built-in WSGI-based request handler. For that, initiate a new file `asgi.py`:
```py
import os
import channels.asgi

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "myproject.settings")
channel_layer = channels.asgi.get_channel_layer()
```

5. Channel consumers
Next, create a new file in the App folder: `/sensorReading/consumers.py`, here we will write functions to handle WebSockets events. These functions will override the built-in handling.
Channels maps WebSocket connections to three channels:
1. When a client connects for the first time, a message is sent to `websocket.connect`.
2. through the session, all messages sent by the client will go through `websocket.receive` channel.
3. A message is sent to `websocket.disconnect`, when the client disconnect.

Here is how to write that as functions:
```py
from channels import Group
from channels.sessions import channel_session

@channel_session
def ws_connect(message):
        print("Adding new user to sensor group")
        Group(“myproject").add(message.reply_channel)                           # Adds user to group for broadcast
        message.reply_channel.send({                                            # Reply to individual directly
        "text": "You're connected to myproject group :) ",

@channel_session
def ws_message(message):
    print("Received message is:" + message['text'])
   
@channel_session
def ws_disconnect(message):
    Group("sensor").discard(message.reply_channel)

```
Notice in the code above, we use `Group`, Group allows for message broadcasting. so anyone connected to "myproject group", can receive WebSocket data from a client.

One last step we need to configure in order to have things set is to go back to the routhing section in `routing.py`, and map channel events to our “consumer” functions:
```py
from sensorReading.consumers import ws_message, ws_connect, ws_disconnect

channel_routing = {
    'websocket.connect': ws_connect,
    'websocket.receive': ws_message,
    'websocket.disconnect': ws_disconnect,
}
```

#### Handling real-time data
After having the channel layer ready, the next step is to configure some files to handle the real-time data sent by clients, by preparing the following:

1. Add a new directory: `/myproject/static/`, then create `/myproject/static/sensorReading`. Here we should place our static files.

2. In the `settings.py` file add this new line:
```py
STATICFILES_DIRS = ( os.path.join(BASE_DIR, ‘static’),)
```

3. In `/myproject/static/sensorReading`, create new Javascript, call it for example: `sensor.js`. This contains the code that handle WebSockets dynamically in the backend.
```js
$(function() {
    // ? : (Conditional ) means: If Condition is true? Then value X : Otherwise value Y
    var ws_scheme = window.location.protocol == "https:" ? "wss" : "ws";
    var sensorSocket = new ReconnectingWebSocket(ws_scheme + '://' + window.location.host + "/myproject/");

    sensorSocket.onopen = function() {
           console.log("Connected!");             // just for debugging
           $('#myproject').text("Connected!");
           sensorSocket.send("Connected!");
    };

    sensorSocket.onmessage = function(message) {
        console.log("Received Socket message!");  // just for debugging
        console.log(message);                     // just for debugging
        $('#myproject').text(message.data);
    };

});
```
Here, we are initiating a new WebSocket object, with specific address. This offers us an access to number of object's events. We used, onopen: which defines what happened when socket connection is established. and onmessage: This event occurs when client receives data from server. Both function receives data from client "message". To read it check: message.data.
Notice that we are sending and received data with `myproject` as an id.

4. To have the received data displayed, we need to modify our `index.html` file:
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>{% block title %}SensorReading Demo{% endblock %}</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
</head>
<body>
    {% load staticfiles%}
    <div class="container">
          <div id="sensor">SensorReading Demo</div>            
    </div>
  <script type="text/javascript" src='{% static "sensorReading/jquery-1.12.1.min.js" %}'></script> 
  <script type="text/javascript" src='{% static "sensorReading/reconnecting-websocket.min.js" %}'></script>
  <script type="text/javascript" src='{% static "sensorReading/sensor.js" %}'></script>
</body>
</html>
```
The static javascript files are added in the static directory. jquery-1.12.1.min.js is the jQuery library and reconnecting-websocket.min.js is a library that automatically reconnects the WebSocket if connection is dropped. 

Data is received by stating the id: `myproject`, which we set in our sensor.js file. i.e. here:
``` html
  <div id="sensor">SensorReading Demo</div> 
```
Now, if your client is up and it's sending data (which we haven't explained yet), you should see the data stream if you open the browser to http://127.0.0.1:8000/.
