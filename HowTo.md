## Implementation:
step1: Django setup:
First, since Django is based on Python, make sure that Python is installed in your server.
Next install virtual environment. The idea behind that is to isolate your project, and all installed package from the rest of the system. This insures no conflict with installed package in your system and ease the task of changing some packages within your project. 
Here are the steps:

```py
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
(myproject) ~$ pip install --upgrade pip
(myproject) ~$ pip install django~=1.10.3  // or the latest
(myproject)~$ django-admin.py startproject myproject
