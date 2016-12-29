#Goal:# create simple wireless sensor network

* Get the Django webserver started. Just enough to see some kind of Django sample page when you access the server from your web browser.
* Learn how to edit the models.py so that you can "render" (show in the browser) whatever view you want.
* Make the VIEW mentioned above have a link to another url (still on your web server). This will trigger a request to your server for a different url. This url will be handled by a different method in your models.py, which in turn will render some other view. For now, imagine this other view displaying "Yes, youve lit the LED" or "LED lighting completed". The new method needed to render this view will be where you hook your code into for running your script.
* Instead of running the script, just print something to the console to ensure that your "hook" is working. print "***** SOON ILL REPLACE THIS WITH WHATEVER IS NEEDED TO RUN MY SCRIPT ****" and look for that in the console
* Then learn how to execute python from Django, and drop it in there.

#Tools: Raspberry Pi, django, AJAX, JSON

#Keywords: GET, POST, AJAX, JS, XML, JSON, Django channels, Websocket, Node.js

#Questions:
Can I use the light weight server that comes with django and access it remotely ?Start by breaking down the problem into smaller goals

##System components:
#(a) Server side.
1. Django: 
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
        

3. Bootstrap
        - the most popular HTML, CSS, and JS framework for developing responsive, mobile first projects on the web.
        - makes your website look nice.
        http://getbootstrap.com/


4. HighCharts
        - excellent for charts: http://www.highcharts.com/
        - Built based on JS and JQuery. 


#(b) Client side.
#(c) connections.

##Big Picture:
This article tries to explain one way to build a simple wireless sensor network. At the end of this tutorial, you should be able to send data from multiple sensors to a server, then show these acquired data in a nice dashboard displayed in a public web page.

The sensor (client) sends the data to the server using Websocket protocol. The server uses Django channels; an extension to Django framework which adds a new layer to support Websocket handling. After receiving the data at the server side, Django and some Javascript packages will take care of web development part.

## Implementation:
step1: Django setup:
First, since Django is based on Python, make sure that Python is installed in your server.
Next install virtual environment. The idea behind that is to isolate your project, and all installed package from the rest of the system. This insures no conflict with installed package in your system and ease the task of changing some packages within your project. 
Here are the steps:

```js
var WebSocket = require('ws');
var ws = new WebSocket('ws://www.host.com/path');

ws.on('open', function open() {
  var array = new Float32Array(5);

  for (var i = 0; i < array.length; ++i) {
    array[i] = i / 2;
  }

  ws.send(array, { binary: true, mask: true });
});
```
Now, you have new environment ready. Install latest pip, then Django.
```
(myproject) ~$ pip install --upgrade pip
(myproject) ~$ pip install django~=1.10.3  // or the latest
(myproject)~$ django-admin.py startproject myproject
```
