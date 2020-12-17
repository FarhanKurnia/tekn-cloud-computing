# Application Containerization and Microservice Orchestration

## Stage Setup
Let’s get started by first cloning the demo code repository, changing the working directory, and checking the demo branch out.
```bash
[node1] (local) root@192.168.0.28 ~
$ git clone https://github.com/ibnesayeed/linkextractor.git
Cloning into 'linkextractor'...
remote: Enumerating objects: 4, done.
remote: Counting objects: 100% (4/4), done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 144 (delta 0), reused 0 (delta 0), pack-reused 140
Receiving objects: 100% (144/144), 44.55 KiB | 7.42 MiB/s, done.
Resolving deltas: 100% (43/43), done.
[node1] (local) root@192.168.0.28 ~
$ cd linkextractor
[node1] (local) root@192.168.0.28 ~/linkextractor
$ git checkout demo
Branch 'demo' set up to track remote branch 'demo' from 'origin'.
Switched to a new branch 'demo'
```

## Step 0: Basic Link Extractor Script
Checkout the step0 branch and list files in it.
```bash
[node1] (local) root@192.168.0.23 ~/linkextractor
$ git checkout step0
Switched to branch 'step0'
Your branch is up to date with 'origin/step0'.
[node1] (local) root@192.168.0.23 ~/linkextractor
$ tree
.
├── README.md
├── linkextractor.py
└── logs
    └── extraction.log

1 directory, 3 files
```
The linkextractor.py file is the interesting one here, so let’s look at its contents:
```bash
[node1] (local) root@192.168.0.23 ~/linkextractor
$ cat linkextractor.py
```
```python
#!/usr/bin/env python

import sys
import requests
from bs4 import BeautifulSoup

res = requests.get(sys.argv[-1])
soup = BeautifulSoup(res.text, "html.parser")
for link in soup.find_all("a"):
    print(link.get("href"))
```

This is a simple Python script that imports three packages: sys from the standard library and two popular third-party packages requests and bs4. User-supplied command line argument (which is expected to be a URL to an HTML page) is used to fetch the page using the requests package, then parsed using the BeautifulSoup. The parsed object is then iterated over to find all the anchor elements (i.e., <a> tags) and print the value of their href attribute that contains the hyperlink.

However, this seemingly simple script might not be the easiest one to run on a machine that does not meet its requirements. The README.md file suggests how to run it, so let’s give it a try:
```bash
[node1] (local) root@192.168.0.23 ~/linkextractor
$ ./linkextractor.py http://example.com/
bash: ./linkextractor.py: Permission denied
```
When we tried to execute it as a script, we got the `Permission denied` error. Let’s check the current permissions on this file:
```bash
$ ls -l linkextractor.py
-rw-r--r--    1 root     root           220 Dec 17 08:56 linkextractor.py
```

This current permission -rw-r--r-- indicates that the script is not set to be executable. We can either change it by running chmod a+x linkextractor.py or run it as a Python program instead of a self-executing script as illustrated below:
```bash
[node1] (local) root@192.168.0.28 ~/linkextractor
$ python3 linkextractor.py
Traceback (most recent call last):
  File "linkextractor.py", line 5, in <module>
    from bs4 import BeautifulSoup
ModuleNotFoundError: No module named 'bs4'
```
Here we got the first ImportError message because we are missing the third-party package needed by the script. We can install that Python package (and potentially other missing packages) using one of the many techniques to make it work, but it is too much work for such a simple script, which might not be obvious for those who are not familiar with Python’s ecosystem.

Depending on which machine and operating system you are trying to run this script on, what software is already installed, and how much access you have, you might face some of these potential difficulties:

Is the script executable?
Is Python installed on the machine?
Can you install software on the machine?
Is pip installed?
Are requests and beautifulsoup4 Python libraries installed?
This is where application containerization tools like Docker come in handy. In the next step we will try to containerize this script and make it easier to execute.

## Step 1: Containerized Link Extractor Script
Checkout the step1 branch and list files in it.
```bash
[node1] (local) root@192.168.0.23 ~/linkextractor
$ tree
.
├── Dockerfile
├── README.md
├── linkextractor.py
└── logs
    └── extraction.log

1 directory, 4 files
```

We have added one new file (i.e., Dockerfile) in this step. Let’s look into its contents:
```bash
1 directory, 4 files
[node1] (local) root@192.168.0.23 ~/linkextractor
$ cat Dockerfile
FROM       python:3
LABEL      maintainer="Sawood Alam <@ibnesayeed>"

RUN        pip install beautifulsoup4
RUN        pip install requests

WORKDIR    /app
COPY       linkextractor.py /app/
RUN        chmod a+x linkextractor.py

ENTRYPOINT ["./linkextractor.py"]
```
Using this Dockerfile we can prepare a Docker image for this script. We start from the official python Docker image that contains Python’s run-time environment as well as necessary tools to install Python packages and dependencies. We then add some metadata as labels (this step is not essential, but is a good practice nonetheless). Next two instructions run the pip install command to install the two third-party packages needed for the script to function properly. We then create a working directory /app, copy the linkextractor.py file in it, and change its permissions to make it an executable script. Finally, we set the script as the entrypoint for the image.

So far, we have just described how we want our Docker image to be like, but didn’t really build one. So let’s do just that:
```bash
[node1] (local) root@192.168.0.23 ~/linkextractor
$ docker image build -t linkextractor:step1 .
Sending build context to Docker daemon  122.4kB
Step 1/8 : FROM       python:3
 ---> 2770b69c10e1
Step 2/8 : LABEL      maintainer="Sawood Alam <@ibnesayeed>"
 ---> Using cache
 ---> e127c0bb2911
Step 3/8 : RUN        pip install beautifulsoup4
 ---> Using cache
 ---> 8556d765ed11
Step 4/8 : RUN        pip install requests
 ---> Using cache
 ---> cc35dc9182a3
Step 5/8 : WORKDIR    /app
 ---> Using cache
 ---> 103e2e9a8cad
Step 6/8 : COPY       linkextractor.py /app/
 ---> 5cf1a9db6cbe
Step 7/8 : RUN        chmod a+x linkextractor.py
 ---> Running in 328badde0c82
Removing intermediate container 328badde0c82
 ---> 05601f1ead15
Step 8/8 : ENTRYPOINT ["./linkextractor.py"]
 ---> Running in 769c6961e91f
Removing intermediate container 769c6961e91f
 ---> 309179fef864
Successfully built 309179fef864
Successfully tagged linkextractor:step1
```
We have created a Docker image named linkextractor:step1 based on the Dockerfile illustrated above. If the build was successful, we should be able to see it in the list of image:
```bash
[node1] (local) root@192.168.0.23 ~/linkextractor
$ docker image ls
REPOSITORY          TAG            IMAGE IDCREATED             SIZE
linkextractor       step1          309179fef86441 seconds ago      895MB
python              3              2770b69c10e135 hours ago        885MB
```
This image should have all the necessary ingredients packaged in it to run the script anywhere on a machine that supports Docker. Now, let’s run a one-off container with this image and extract links from some live web pages:
$ docker container run -it --rm linkextractor:step1 http://example.com/
https://www.iana.org/domains/example
This outputs a single link that is present in the simple example.com web page:
```http://www.iana.org/domains/example```

Let’s try it on a web page with more links in it:
```bash
[node1] (local) root@192.168.0.23 ~/linkextractor
$ docker container run -it --rm linkextractor:step1 https://training.play-with-docker.com/
/
/about/
#ops
#dev
/ops-stage1
/ops-stage2
/ops-stage3
/dev-stage1
/dev-stage2
/dev-stage3
/alacart
https://twitter.com/intent/tweet?text=Play with Docker Classroom&url=https://training.play-with-docker.com/&via=docker&related=docker
https://facebook.com/sharer.php?u=https://training.play-with-docker.com/
https://plus.google.com/share?url=https://training.play-with-docker.com/
http://www.linkedin.com/shareArticle?mini=true&url=https://training.play-with-docker.com/&title=Play%20with%20Docker%20Classroom&source=https://training.play-with-docker.com
https://www.docker.com/dockercon/
https://www.docker.com/dockercon/
https://dockr.ly/slack
https://www.docker.com/legal/docker-terms-service
https://www.docker.com
https://www.facebook.com/docker.run
https://twitter.com/docker
https://www.github.com/play-with-docker/play-with-docker.github.io
```

This looks good, but we can improve the output. For example, some links are relative, we can convert them into full URLs and also provide the anchor text they are linked to. In the next step we will make these changes and some other improvements to the script.

## Step 2: Link Extractor Module with Full URI and Anchor Text
Checkout the step2 branch and list files in it.
```bash
$ git checkout step2
Switched to branch 'step2'
Your branch is up to date with 'origin/step2'.
[node1] (local) root@192.168.0.23 ~/linkextractor
$ tree
.
├── Dockerfile
├── README.md
├── linkextractor.py
└── logs
    └── extraction.log

1 directory, 4 files
```
In this step the linkextractor.py script is updated with the following functional changes:

Paths are normalized to full URLs
Reporting both links and anchor texts
Usable as a module in other scripts
Let’s have a look at the updated script:
```bash
[node1] (local) root@192.168.0.23 ~/linkextractor
$ cat linkextractor.py
```
```python
#!/usr/bin/env python

import sys
import requests
from bs4 import BeautifulSoup
from urllib.parse import urljoin

def extract_links(url):
    res = requests.get(url)
    soup = BeautifulSoup(res.text, "html.parser")
    base = url
    # TODO: Update base if a <base> element is present with the href attribute
    links = []
    for link in soup.find_all("a"):
        links.append({
            "text": " ".join(link.text.split()) or "[IMG]",
            "href": urljoin(base, link.get("href"))
        })
    return links

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("\nUsage:\n\t{} <URL>\n".format(sys.argv[0]))
        sys.exit(1)
    for link in extract_links(sys.argv[-1]):
        print("[{}]({})".format(link["text"], link["href"]))
```

The link extraction logic is abstracted into a function extract_links that accepts a URL as a parameter and returns a list of objects containing anchor texts and normalized hyperlinks. This functionality can now be imported into other scripts as a module (which we will utilize in the next step).

Now, let’s build a new image and see these changes in effect:
```bash
[node1] (local) root@192.168.0.23 ~/linkextractor
$ docker image build -t linkextractor:step2 .
Sending build context to Docker daemon  122.9kB
Step 1/8 : FROM       python:3
 ---> 2770b69c10e1
Step 2/8 : LABEL      maintainer="Sawood Alam <@ibnesayeed>"
 ---> Using cache
 ---> e127c0bb2911
Step 3/8 : RUN        pip install beautifulsoup4
 ---> Using cache
 ---> 8556d765ed11
Step 4/8 : RUN        pip install requests
 ---> Using cache
 ---> cc35dc9182a3
Step 5/8 : WORKDIR    /app
 ---> Using cache
 ---> 103e2e9a8cad
Step 6/8 : COPY       linkextractor.py /app/
 ---> Using cache
 ---> cc8192a267dd
Step 7/8 : RUN        chmod a+x linkextractor.py
 ---> Using cache
 ---> fe56a5fc3c55
Step 8/8 : ENTRYPOINT ["./linkextractor.py"]
 ---> Using cache
 ---> bf3431cafdc6
Successfully built bf3431cafdc6
Successfully tagged linkextractor:step2
```
We have used a new tag linkextractor:step2 for this image so that we don’t overwrite the image from the step0 to illustrate that they can co-exist and containers can be run using either of these images.
```bash
[node1] (local) root@192.168.0.23 ~/linkextractor
$ docker image ls
REPOSITORY          TAG            IMAGE IDCREATED             SIZE
linkextractor       step2          bf3431cafdc647 minutes ago      895MB
linkextractor       step1          309179fef8646 minutes ago       895MB
python              3              2770b69c10e135 hours ago        885MB
```

Running a one-off container using the linkextractor:step2 image should now yield an improved output:
```bash
[node1] (local) root@192.168.0.23 ~/linkextractor
$ docker container run -it --rm linkextractor:step2 https://training.play-with-docker.com/
[Play with Docker classroom](https://training.play-with-docker.com/)
[About](https://training.play-with-docker.com/about/)
[IT Pros and System Administrators](https://training.play-with-docker.com/#ops)
[Developers](https://training.play-with-docker.com/#dev)
[Stage 1: The Basics](https://training.play-with-docker.com/ops-stage1)
[Stage 2: Digging Deeper](https://training.play-with-docker.com/ops-stage2)
[Stage 3: Moving to Production](https://training.play-with-docker.com/ops-stage3)
[Stage 1: The Basics](https://training.play-with-docker.com/dev-stage1)
[Stage 2: Digging Deeper](https://training.play-with-docker.com/dev-stage2)
[Stage 3: Moving to Staging](https://training.play-with-docker.com/dev-stage3)
[Full list of individual labs](https://training.play-with-docker.com/alacart)
[[IMG]](https://twitter.com/intent/tweet?text=Play with Docker Classroom&url=https://training.play-with-docker.com/&via=docker&related=docker)
[[IMG]](https://facebook.com/sharer.php?u=https://training.play-with-docker.com/)
[[IMG]](https://plus.google.com/share?url=https://training.play-with-docker.com/)
[[IMG]](http://www.linkedin.com/shareArticle?mini=true&url=https://training.play-with-docker.com/&title=Play%20with%20Docker%20Classroom&source=https://training.play-with-docker.com)
[[IMG]](https://www.docker.com/dockercon/)
[Sign up today](https://www.docker.com/dockercon/)
[Register here](https://dockr.ly/slack)
[here](https://www.docker.com/legal/docker-terms-service)
[[IMG]](https://www.docker.com)
[[IMG]](https://www.facebook.com/docker.run)
[[IMG]](https://twitter.com/docker)
[[IMG]](https://www.github.com/play-with-docker/play-with-docker.github.io)
```

Running a container using the previous image linkextractor:step1 should still result in the old output:
```bash
[node1] (local) root@192.168.0.23 ~/linkextractor
$ docker container run -it --rm linkextractor:step1 https://training.play-with-docker.com/
/
/about/
#ops
#dev
/ops-stage1
/ops-stage2
/ops-stage3
/dev-stage1
/dev-stage2
/dev-stage3
/alacart
https://twitter.com/intent/tweet?text=Play with Docker Classroom&url=https://training.play-with-docker.com/&via=docker&related=docker
https://facebook.com/sharer.php?u=https://training.play-with-docker.com/
https://plus.google.com/share?url=https://training.play-with-docker.com/
http://www.linkedin.com/shareArticle?mini=true&url=https://training.play-with-docker.com/&title=Play%20with%20Docker%20Classroom&source=https://training.play-with-docker.com
https://www.docker.com/dockercon/
https://www.docker.com/dockercon/
https://dockr.ly/slack
https://www.docker.com/legal/docker-terms-service
https://www.docker.com
https://www.facebook.com/docker.run
https://twitter.com/docker
https://www.github.com/play-with-docker/play-with-docker.github.io
```

So far, we have learned how to containerize a script with its necessary dependencies to make it more portable. We have also learned how to make changes in the application and build different variants of Docker images that can co-exist. In the next step we will build a web service that will utilize this script and will make the service run inside a Docker container.

## Step 3: Link Extractor API Service
Checkout the step3 branch and list files in it.
```bash
[node1] (local) root@192.168.0.23 ~/linkextractor
$ git checkout step3
Switched to branch 'step3'
Your branch is up to date with 'origin/step3'.
[node1] (local) root@192.168.0.23 ~/linkextractor
$ tree
.
├── Dockerfile
├── README.md
├── linkextractor.py
├── logs
│   └── extraction.log
├── main.py
└── requirements.txt

1 directory, 6 files
```
The following changes have been made in this step:

Added a server script main.py that utilizes the link extraction module written in the last step
The Dockerfile is updated to refer to the main.py file instead
Server is accessible as a WEB API at http://<hostname>[:<prt>]/api/<url>
Dependencies are moved to the requirements.txt file
Needs port mapping to make the service accessible outside of the container (the Flask server used here listens on port 5000 by default)
Let’s first look at the Dockerfile for changes:
```bash
[node1] (local) root@192.168.0.23 ~/linkextractor
$ cat Dockerfile
FROM       python:3
LABEL      maintainer="Sawood Alam <@ibnesayeed>"

WORKDIR    /app
COPY       requirements.txt /app/
RUN        pip install -r requirements.txt

COPY       *.py /app/
RUN        chmod a+x *.py

CMD        ["./main.py"]
```

Since we have started using requirements.txt for dependencies, we no longer need to run pip install command for individual packages. The ENTRYPOINT directive is replaced with the CMD and it is referring to the main.py script that has the server code it because we do not want to use this image for one-off commands now.

The linkextractor.py module remains unchanged in this step, so let’s look into the newly added main.py file:
```bash
[node1] (local) root@192.168.0.23 ~/linkextractor
$ cat main.py
```
```python
#!/usr/bin/env python

from flask import Flask
from flask import request
from flask import jsonify
from linkextractor import extract_links

app = Flask(__name__)

@app.route("/")
def index():
    return "Usage: http://<hostname>[:<prt>]/api/<url>"

@app.route("/api/<path:url>")
def api(url):
    qs = request.query_string.decode("utf-8")
    if qs != "":
        url += "?" + qs
    links = extract_links(url)
    return jsonify(links)

app.run(host="0.0.0.0")
```

Here, we are importing extract_links function from the linkextractor module and converting the returned list of objects into a JSON response.

It’s time to build a new image with these changes in place:
```bash
[node1] (local) root@192.168.0.23 ~/linkextractor
$ docker image build -t linkextractor:step3 .
Sending build context to Docker daemon  125.4kB
Step 1/8 : FROM       python:3
 ---> 2770b69c10e1
Step 2/8 : LABEL      maintainer="Sawood Alam <@ibnesayeed>"
 ---> Using cache
 ---> e127c0bb2911
Step 3/8 : WORKDIR    /app
 ---> Using cache
 ---> f5a4d8c2c7b2
Step 4/8 : COPY       requirements.txt /app/
 ---> Using cache
 ---> 686294e9bfda
Step 5/8 : RUN        pip install -r requirements.txt
 ---> Using cache
 ---> 3d0d9931238a
Step 6/8 : COPY       *.py /app/
 ---> Using cache
 ---> b14132097339
Step 7/8 : RUN        chmod a+x *.py
 ---> Using cache
 ---> 34b2ece844a7
Step 8/8 : CMD        ["./main.py"]
 ---> Using cache
 ---> 0a791f7624ed
Successfully built 0a791f7624ed
Successfully tagged linkextractor:step3
```
Then run the container in detached mode (-d flag) so that the terminal is available for other commands while the container is still running. Note that we are mapping the port 5000 of the container with the 5000 of the host (using -p 5000:5000 argument) to make it accessible from the host. We are also assigning a name (--name=linkextractor) to the container to make it easier to see logs and kill or remove the container.
```bash
$ docker container run -d -p 5000:5000 --name=linkextractor linkextractor:step3
88a23f7c50a37b686a0433b6b0fea32239abf6195ad49ee6913fb487b1640400
```

If things go well, we should be able to see the container being listed in Up condition:
```bash
[node1] (local) root@192.168.0.23 ~/linkextractor
$ docker container ls
CONTAINER ID   IMAGE                 COMMAND       CREATED          STATUS          PORTS NAMES
88a23f7c50a3   linkextractor:step3   "./main.py"   22seconds ago   Up 20 seconds   0.0.0.0:5000->5000/tcp linkextractor
```

We can now make an HTTP request in the form /api/<url> to talk to this server and fetch the response containing extracted links:
```bash
[node1] (local) root@192.168.0.23 ~/linkextractor
$ curl -i http://localhost:5000/api/http://example.com/
HTTP/1.0 200 OK
Content-Type: application/json
Content-Length: 79
Server: Werkzeug/1.0.1 Python/3.9.1
Date: Thu, 17 Dec 2020 09:16:43 GMT

[{"href":"https://www.iana.org/domains/example","text":"More information..."}]
```

Now, we have the API service running that accepts requests in the form /api/<url> and responds with a JSON containing hyperlinks and anchor texts of all the links present in the web page at give <url>.

Since the container is running in detached mode, so we can’t see what’s happening inside, but we can see logs using the name linkextractor we assigned to our container:
```bash
[node1] (local) root@192.168.0.23 ~/linkextractor
$ docker container logs linkextractor
 * Serving Flask app "main" (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
172.17.0.1 - - [17/Dec/2020 09:16:37] "GET /api/http://example.com/ HTTP/1.1" 200 -
172.17.0.1 - - [17/Dec/2020 09:16:43] "GET /api/http://example.com/ HTTP/1.1" 200 -
```

We can see the messages logged when the server came up, and an entry of the request log when we ran the curl command. Now we can kill and remove this container:
```bash
[node1] (local) root@192.168.0.23 ~/linkextractor
$ docker container rm -f linkextractor
linkextractor
```
In this step we have successfully ran an API service listening on port 5000. This is great, but APIs and JSON responses are for machines, so in the next step we will run a web service with a human-friendly web interface in addition to this API service.

## Step 4: Link Extractor API and Web Front End Services
Checkout the step4 branch and list files in it.
```bash
[node1] (local) root@192.168.0.23 ~/linkextractor
$ git checkout step4
Switched to branch 'step4'
Your branch is up to date with 'origin/step4'.
[node1] (local) root@192.168.0.23 ~/linkextractor
$ tree
.
├── README.md
├── api
│   ├── Dockerfile
│   ├── linkextractor.py
│   ├── main.py
│   └── requirements.txt
├── docker-compose.yml
├── logs
│   └── extraction.log
└── www
    └── index.php

3 directories, 8 files
```

In this step the following changes have been made since the last step:

The link extractor JSON API service (written in Python) is moved in a separate ./api folder that has the exact same code as in the previous step
A web front-end application is written in PHP under ./www folder that talks to the JSON API
The PHP application is mounted inside the official php:7-apache Docker image for easier modification during the development
The web application is made accessible at http://<hostname>[:<prt>]/?url=<url-encoded-url>
An environment variable API_ENDPOINT is used inside the PHP application to configure it to talk to the JSON API server
A docker-compose.yml file is written to build various components and glue them together
In this step we are planning to run two separate containers, one for the API and the other for the web interface. The latter needs a way to talk to the API server. For the two containers to be able to talk to each other, we can either map their ports on the host machine and use that for request routing or we can place the containers in a single private network and access directly. Docker has an excellent support of networking and provides helpful commands to deal with networks. Additionally, in a Docker network containers identify themselves using their names as hostnames to avoid hunting for their IP addresses in the private network. However, we are not going to do any of this manually, instead we will be using Docker Compose to automate many of these tasks.

So let’s look at the docker-compose.yml file we have:
```bash
[node1] (local) root@192.168.0.23 ~/linkextractor
$ cat docker-compose.yml
version: '3'

services:
  api:
    image: linkextractor-api:step4-python
    build: ./api
    ports:
      - "5000:5000"
  web:
    image: php:7-apache
    ports:
      - "80:80"
    environment:
      - API_ENDPOINT=http://api:5000/api/
    volumes:
      - ./www:/var/www/html
```

This is a simple YAML file that describes the two services api and web. The api service will use the linkextractor-api:step4-python image that is not built yet, but will be built on-demand using the Dockerfile from the ./api directory. This service will be exposed on the port 5000 of the host.

The second service named web will use official php:7-apache image directly from the DockerHub, that’s why we do not have a Dockerfile for it. The service will be exposed on the default HTTP port (i.e., 80). We will supply an environment variable named API_ENDPOINT with the value http://api:5000/api/ to tell the PHP script where to connect to for the API access. Notice that we are not using an IP address here, instead, api:5000 is being used because we will have a dynamic hostname entry in the private network for the API service matching its service name. Finally, we will bind mount the ./www folder to make the index.php file available inside of the web service container at /var/www/html, which is the default web root for the Apache web server.

Now, let’s have a look at the user-facing www/index.php file:
```bash
[node1] (local) root@192.168.0.23 ~/linkextractor
$ cat www/index.php
```
```php
<!DOCTYPE html>

<?php
  $api_endpoint = $_ENV["API_ENDPOINT"] ?: "http://localhost:5000/api/";
  $url = "";
  if(isset($_GET["url"]) && $_GET["url"] != "") {
    $url = $_GET["url"];
    $json = @file_get_contents($api_endpoint . $url);
    if($json == false) {
      $err = "Something is wrong with the URL: " . $url;
    } else {
      $links = json_decode($json, true);
      $domains = [];
      foreach($links as $link) {
        array_push($domains, parse_url($link["href"],PHP_URL_HOST));
      }
      $domainct = @array_count_values($domains);
      arsort($domainct);
    }
  }
?>

<html>
  <head>
    <meta charset="utf-8">
    <title>Link Extractor</title>
    <style media="screen">
      html {
        background: #EAE7D6;
        font-family: sans-serif;
      }
      body {
        margin: 0;
      }
      h1 {
        padding: 10px;
        margin: 0 auto;
        color: #EAE7D6;
        max-width: 600px;
      }
      h1 a {
        text-decoration: none;
        color: #EAE7D6;
      }
      h2 {
        background: #082E41;
        color: #EAE7D6;
        margin: -10px;
        padding: 10px;
      }
      p {
        margin: 25px 5px 5px;
      }
      section {
        max-width: 600px;
        margin: 10px auto;
        padding: 10px;
        border: 1px solid #082E41;
      }
      div.header {
        background: #082E41;
        margin: 0;
      }
      div.footer {
        background: #082E41;
        margin: 0;
        padding: 5px;
      }
      .footer p {
        margin: 0 auto;
        max-width: 600px;
        color: #EAE7D6;
        text-align: center;
      }
      .footer p a {
        color: #24C2CB;
        text-decoration: none;
      }
      .error {
        color: #DA2536;
      }
      form {
        display: flex;
      }
      input {
        font-size: 20px;
        padding: 3px;
        height: 40px;
      }
      input.text {
        box-sizing:border-box;
        flex-grow: 1;
        border-color: #082E41;
      }
      input.button {
        width: 150px;
        background: #082E41;
        border-color: #082E41;
        color: #EAE7D6;
      }
      table {
        width: 100%;
        text-align: left;
        margin-top: 10px;
      }
      table th, table td {
        padding: 3px;
      }
      table th:last-child, table td:last-child {
        width: 70px;
        text-align: right;
      }
      table th {
        border-top: 1px solid #082E41;
        border-bottom: 1px solid #082E41;
      }
      table tr:last-child td {
        border-top: 1px solid #082E41;
        border-bottom: 1px solid #082E41;
      }
    </style>
  </head>
  <body>
    <div class="header">
      <h1><a href="/">Link Extractor</a></h1>
    </div>

    <section>
      <form action="/">
        <input class="text" type="text" name="url" placeholder="http://example.com/" value="<?php echo $url; ?>">
        <input class="button" type="submit" value="Extract Links">
      </form>
    </section>

    <?php if(isset($err)): ?>
      <section>
        <h2>Error</h2>
        <p class="error"><?php echo $err; ?></p>
      </section>
    <?php endif; ?>

    <?php if($url != "" && !isset($err)): ?>
      <section>
        <h2>Summary</h2>
        <p>
          <strong>Page:</strong> <?php echo "<a href=\"" . $url . "\">" . $url . "</a>"; ?>
        </p>
        <table>
          <tr>
            <th>Domain</th>
            <th># Links</th>
          </tr>
          <?php
            foreach($domainct as $key => $value) {
              echo "<tr>";
              echo "<td>" . $key . "</td>";
              echo "<td>" . $value . "</td>";
              echo "</tr>";
            }
          ?>
          <tr>
            <td><strong>Total</strong></td>
            <td><strong><?php echo count($links); ?></strong></td>
          </tr>
        </table>
      </section>

      <section>
        <h2>Links</h2>
        <ul>
        <?php
          foreach($links as $link) {
            echo "<li><a href=\"" . $link["href"] . "\">" . $link["text"] . "</a></li>";
          }
        ?>
        </ul>
      </section>
    <?php endif; ?>

    <div class="footer">
      <p><a href="https://github.com/ibnesayeed/linkextractor">Link Extractor</a> by <a href="https://twitter.com/ibnesayeed">@ibnesayeed</a> from
        <a href="https://ws-dl.cs.odu.edu/">WS-DL, ODU</a>
      </p>
    </div>
  </body>
</html>
```

The $api_endpoint variable is initialized with the value of the environment variable supplied from the docker-compose.yml file as $_ENV["API_ENDPOINT"] (otherwise falls back to a default value of http://localhost:5000/api/). The request is made using file_get_contents function that uses the $api_endpoint variable and user supplied URL from $_GET["url"]. Some analysis and transformations are performed on the received response that are later used in the markup to populate the page.

Let’s bring these services up in detached mode using docker-compose utility:
```bash
[node1] (local) root@192.168.0.23 ~/linkextractor
$ docker-compose up -d --build
Creating network "linkextractor_default" with the default driver
Building api
Step 1/8 : FROM       python:3
 ---> 2770b69c10e1
Step 2/8 : LABEL      maintainer="Sawood Alam <@ibnesayeed>"
 ---> Using cache
 ---> e127c0bb2911
Step 3/8 : WORKDIR    /app
 ---> Using cache
 ---> f5a4d8c2c7b2
Step 4/8 : COPY       requirements.txt /app/
 ---> Using cache
 ---> f2d7a54af074
Step 5/8 : RUN        pip install -r requirements.txt
 ---> Using cache
 ---> 43fc5ff3c9fc
Step 6/8 : COPY       *.py /app/
 ---> Using cache
 ---> effa0c0165e8
Step 7/8 : RUN        chmod a+x *.py
 ---> Using cache
 ---> d77903d6e660
Step 8/8 : CMD        ["./main.py"]
 ---> Using cache
 ---> a2d99055ef16
Successfully built a2d99055ef16
Successfully tagged linkextractor-api:step4-python
Creating linkextractor_web_1 ... done
Creating linkextractor_api_1 ... done[node1] (local) root@192.168.0.23 ~/linkextractor
$ docker-compose up -d --build
Creating network "linkextractor_default" with the default driver
Building api
Step 1/8 : FROM       python:3
 ---> 2770b69c10e1
Step 2/8 : LABEL      maintainer="Sawood Alam <@ibnesayeed>"
 ---> Using cache
 ---> e127c0bb2911
Step 3/8 : WORKDIR    /app
 ---> Using cache
 ---> f5a4d8c2c7b2
Step 4/8 : COPY       requirements.txt /app/
 ---> Using cache
 ---> f2d7a54af074
Step 5/8 : RUN        pip install -r requirements.txt
 ---> Using cache
 ---> 43fc5ff3c9fc
Step 6/8 : COPY       *.py /app/
 ---> Using cache
 ---> effa0c0165e8
Step 7/8 : RUN        chmod a+x *.py
 ---> Using cache
 ---> d77903d6e660
Step 8/8 : CMD        ["./main.py"]
 ---> Using cache
 ---> a2d99055ef16
Successfully built a2d99055ef16
Successfully tagged linkextractor-api:step4-python
Creating linkextractor_web_1 ... done
Creating linkextractor_api_1 ... done
```

This output shows that Docker Compose automatically created a network named linkextractor_default, pulled php:7-apache image from DockerHub, built api:python image using our local Dockerfile, and finally, spun two containers linkextractor_web_1 and linkextractor_api_1 that correspond to the two services we have defined in the YAML file above.

Checking for the list of running containers confirms that the two services are indeed running:
```bash
[node1] (local) root@192.168.0.23 ~/linkextractor
$ docker container ls
CONTAINER ID   IMAGE                            COMMAND                  CREATED          STATUS          PORTS                    NAMES
e0cd8414425c   linkextractor-api:step4-python   "./main.py"              40 seconds ago   Up 39 seconds   0.0.0.0:5000->5000/tcp   linkextractor_api_1
13f1b9274bbf   php:7-apache                     "docker-php-entrypoi…"   40 seconds ago   Up 39 seconds   0.0.0.0:80->80/tcp       linkextractor_web_1
```

We should now be able to talk to the API service as before:
```bash
[node1] (local) root@192.168.0.23 ~/linkextractor
$ curl -i http://localhost:5000/api/http://example.com/
HTTP/1.0 200 OK
Content-Type: application/json
Content-Length: 79
Server: Werkzeug/1.0.1 Python/3.9.1
Date: Thu, 17 Dec 2020 09:21:56 GMT

[{"href":"https://www.iana.org/domains/example","text":"More information..."}]
```
To access the web interface click to open the Link Extractor. Then fill the form with https://training.play-with-docker.com/ (or any HTML page URL of your choice) and submit to extract links from it.

We have just created an application with microservice architecture, isolating individual tasks in separate services as opposed to monolithic applications where everything is put together in a single unit. Microservice applications are relatively easier to scale, maintains, and move around. They also allow easy swapping of components with an equivalent service. More on that later.

Now, let’s modify the www/index.php file to replace all occurrences of Link Extractor with Super Link Extractor:
```bash
[node1] (local) root@192.168.0.23 ~/linkextractor
$ sed -i 's/Link Extractor/Super Link Extractor/g' www/index.php
```

Reloading the web interface of the application (or clicking here) should now reflect this change in the title, header, and footer. This is happening because the ./www folder is bind mounted inside of the container, so any changes made outside will reflect inside the container or the vice versa. This approach is very helpful in development, but in the production environment we would prefer our Docker images to be self-contained. Let’s revert these changes now to clean the Git tracking:
```bash
[node1] (local) root@192.168.0.23 ~/linkextractor
$ git reset --hard
HEAD is now at 2a3ec3e Synchronize branch step4
```

Before we move on to the next step we need to shut these services down, but Docker Compose can help us take care of it very easily:
```bash
[node1] (local) root@192.168.0.23 ~/linkextractor
$ docker-compose down
Stopping linkextractor_api_1 ... done
Stopping linkextractor_web_1 ... done
Removing linkextractor_api_1 ... done
Removing linkextractor_web_1 ... done
Removing network linkextractor_default
```
In the next step we will add one more service to our stack and will build a self-contained custom image for our web interface service

## Step 5: Redis Service for Caching
Checkout the step5 branch and list files in it.
```bash
[node1] (local) root@192.168.0.28 ~/linkextractor
$ git checkout step5
Branch 'step5' set up to track remote branch 'step5' from 'origin'.
Switched to a new branch 'step5'
[node1] (local) root@192.168.0.28 ~/linkextractor
$ tree
.
├── README.md
├── api
│   ├── Dockerfile
│   ├── linkextractor.py
│   ├── main.py
│   └── requirements.txt
├── docker-compose.yml
└── www
    ├── Dockerfile
    └── index.php

2 directories, 8 files
```

Some noticeable changes from the previous step are as following:

Another Dockerfile is added in the ./www folder for the PHP web application to build a self-contained image and avoid live file mounting
A Redis container is added for caching using the official Redis Docker image
The API service talks to the Redis service to avoid downloading and parsing pages that were already scraped before
A REDIS_URL environment variable is added to the API service to allow it to connect to the Redis cache
Let’s first inspect the newly added Dockerfile under the ./www folder:
```bash
[node1] (local) root@192.168.0.28 ~/linkextractor
$ cat www/Dockerfile
FROM       php:7-apache
LABEL      maintainer="Sawood Alam <@ibnesayeed>"

ENV        API_ENDPOINT="http://localhost:5000/api/"

COPY       . /var/www/html/
```

This is a rather simple Dockerfile that uses the official php:7-apache image as the base and copies all the files from the ./www folder into the /var/www/html/ folder of the image. This is exactly what was happening in the previous step, but that was bind mounted using a volume, while here we are making the code part of the self-contained image. We have also added the API_ENDPOINT environment variable here with a default value, which implicitly suggests that this is an important information that needs to be present in order for the service to function properly (and should be customized at run time with an appropriate value).

Next, we will look at the API server’s api/main.py file where we are utilizing the Redis cache:
```bash
[node1] (local) root@192.168.0.28 ~/linkextractor
$ cat api/main.py
```
```python
#!/usr/bin/env python

import os
import json
import redis
from flask import Flask
from flask import request
from linkextractor import extract_links

app = Flask(__name__)
redis_conn = redis.from_url(os.getenv("REDIS_URL", "redis://localhost:6379"))

@app.route("/")
def index():
    return "Usage: http://<hostname>[:<prt>]/api/<url>"

@app.route("/api/<path:url>")
def api(url):
    qs = request.query_string.decode("utf-8")
    if qs != "":
        url += "?" + qs

    jsonlinks = redis_conn.get(url)
    if not jsonlinks:
        links = extract_links(url)
        jsonlinks = json.dumps(links, indent=2)
        redis_conn.set(url, jsonlinks)

    response = app.response_class(
        status=200,
        mimetype="application/json",
        response=jsonlinks
    )

    return response

app.run(host="0.0.0.0")
```
This time the API service needs to know how to connect to the Redis instance as it is going to use it for caching. This information can be made available at run time using the REDIS_URL environment variable. A corresponding ENV entry is also added in the Dockerfile of the API service with a default value.

A redis client instance is created using the hostname redis (same as the name of the service as we will see later) and the default Redis port 6379. We are first trying to see if a cache is present in the Redis store for a given URL, if not then we use the extract_links function as before and populate the cache for future attempts.

Now, let’s look into the updated docker-compose.yml file:
```bash
[node1] (local) root@192.168.0.28 ~/linkextractor
$ cat docker-compose.yml
version: '3'

services:
  api:
    image: linkextractor-api:step5-python
    build: ./api
    ports:
      - "5000:5000"
    environment:
      - REDIS_URL=redis://redis:6379
  web:
    image: linkextractor-web:step5-php
    build: ./www
    ports:
      - "80:80"
    environment:
      - API_ENDPOINT=http://api:5000/api/
  redis:
    image: redis
```

The api service configuration largely remains the same as before, except the updated image tag and added environment variable REDIS_URL that points to the Redis service. For the web service, we are using the custom linkextractor-web:step5-php image that will be built using the newly added Dockerfile in the ./www folder. We are no longer mounting the ./www folder using the volumes config. Finally, a new service named redis is added that will use the official image from DockerHub and needs no specific configurations for now. This service is accessible to the Python API using its service name, the same way the API service is accessible to the PHP front-end service.

Let’s boot these services up:
```bash
[node1] (local) root@192.168.0.28 ~/linkextractor
$ docker-compose up -d --build
Creating network "linkextractor_default" with thedefault driver
Building api
Step 1/9 : FROM       python:3
 ---> 2770b69c10e1
Step 2/9 : LABEL      maintainer="Sawood Alam <@ibnesayeed>"
 ---> Using cache
 ---> 500aee4007f2
Step 3/9 : ENV        REDIS_URL="redis://localhost:6379"
 ---> Running in 1b977431d880
Removing intermediate container 1b977431d880
 ---> 3e99599ecb9d
Step 4/9 : WORKDIR    /app
 ---> Running in a44b2d51abb5
Removing intermediate container a44b2d51abb5
 ---> a0881a4f9855
Step 5/9 : COPY       requirements.txt /app/
 ---> 23ab5bac74b9
Step 6/9 : RUN        pip install -r requirements.txt
 ---> Running in 69da6b71ff98
Collecting beautifulsoup4
  Downloading beautifulsoup4-4.9.3-py3-none-any.whl (115 kB)
Collecting soupsieve>1.2
  Downloading soupsieve-2.1-py3-none-any.whl (32 kB)
Collecting flask
  Downloading Flask-1.1.2-py2.py3-none-any.whl (94 kB)
Collecting click>=5.1
  Downloading click-7.1.2-py2.py3-none-any.whl (82 kB)
Collecting itsdangerous>=0.24
  Downloading itsdangerous-1.1.0-py2.py3-none-any.whl (16 kB)
Collecting Jinja2>=2.10.1
  Downloading Jinja2-2.11.2-py2.py3-none-any.whl (125 kB)
Collecting MarkupSafe>=0.23
  Downloading MarkupSafe-1.1.1.tar.gz (19 kB)
Collecting Werkzeug>=0.15
  Downloading Werkzeug-1.0.1-py2.py3-none-any.whl(298 kB)
Collecting redis
  Downloading redis-3.5.3-py2.py3-none-any.whl (72 kB)
Collecting requests
  Downloading requests-2.25.1-py2.py3-none-any.whl (61 kB)
Collecting certifi>=2017.4.17
  Downloading certifi-2020.12.5-py2.py3-none-any.whl (147 kB)
Collecting chardet<5,>=3.0.2
  Downloading chardet-4.0.0-py2.py3-none-any.whl (178 kB)
Collecting idna<3,>=2.5
  Downloading idna-2.10-py2.py3-none-any.whl (58 kB)
Collecting urllib3<1.27,>=1.21.1
  Downloading urllib3-1.26.2-py2.py3-none-any.whl(136 kB)
Building wheels for collected packages: MarkupSafe
  Building wheel for MarkupSafe (setup.py): started
  Building wheel for MarkupSafe (setup.py): finished with status 'done'
  Created wheel for MarkupSafe: filename=MarkupSafe-1.1.1-cp39-cp39-linux_x86_64.whl size=32226 sha256=5abf93b76b3334ca61259b9348ff9ec35f5d70ff706b5ea0dd7103768d41b58c
  Stored in directory: /root/.cache/pip/wheels/e0/19/6f/6ba857621f50dc08e084312746ed3ebc14211ba30037d5e44e
Successfully built MarkupSafe
Installing collected packages: MarkupSafe, Werkzeug, urllib3, soupsieve, Jinja2, itsdangerous, idna, click, chardet, certifi, requests, redis, flask,beautifulsoup4
Successfully installed Jinja2-2.11.2 MarkupSafe-1.1.1 Werkzeug-1.0.1 beautifulsoup4-4.9.3 certifi-2020.12.5 chardet-4.0.0 click-7.1.2 flask-1.1.2 idna-2.10 itsdangerous-1.1.0 redis-3.5.3 requests-2.25.1 soupsieve-2.1 urllib3-1.26.2
Removing intermediate container 69da6b71ff98
 ---> eeba7ab4e6dc
Step 7/9 : COPY       *.py /app/
 ---> 61b1be12577b
Step 8/9 : RUN        chmod a+x *.py
 ---> Running in 726d7acdfd07
Removing intermediate container 726d7acdfd07
 ---> ed5211d2f703
Step 9/9 : CMD        ["./main.py"]
 ---> Running in 72a09a70d7bf
Removing intermediate container 72a09a70d7bf
 ---> efea996a84f9
Successfully built efea996a84f9
Successfully tagged linkextractor-api:step5-python
Building web
Step 1/4 : FROM       php:7-apache
 ---> fd505f1f4cd8
Step 2/4 : LABEL      maintainer="Sawood Alam <@ibnesayeed>"
 ---> Running in f29fcb1d9f8b
Removing intermediate container f29fcb1d9f8b
 ---> a1149947d7dd
Step 3/4 : ENV        API_ENDPOINT="http://localhost:5000/api/"
 ---> Running in 4bbe8cfc53fa
Removing intermediate container 4bbe8cfc53fa
 ---> efb835cfc593
Step 4/4 : COPY       . /var/www/html/
 ---> fc5a5c5926d1
Successfully built fc5a5c5926d1
Successfully tagged linkextractor-web:step5-php
Pulling redis (redis:)...
latest: Pulling from library/redis
6ec7b7d162b2: Already exists
1f81a70aa4c8: Pull complete
968aa38ff012: Pull complete
884c313d5b0b: Pull complete
6e858785fea5: Pull complete
78bcc34f027b: Pull complete
Digest: sha256:0f724af268d0d3f5fb1d6b33fc22127ba5cbca2d58523b286ed3122db0dc5381
Status: Downloaded newer image for redis:latest
Creating linkextractor_redis_1 ... done
Creating linkextractor_web_1   ... done
Creating linkextractor_api_1   ... done
```

Now, that all three services are up, access the web interface by clicking the Link Extractor. There should be no visual difference from the previous step. However, if you extract links from a page with a lot of links, the first time it should take longer, but the successive attempts to the same page should return the response fairly quickly. To check whether or not the Redis service is being utilized, we can use docker-compose exec followed by the redis service name and the Redis CLI’s monitor command:
```bash
[node1] (local) root@192.168.0.28 ~/linkextractor
$ docker-compose exec redis redis-cli monitor
OK
```

Now, try to extract links from some web pages using the web interface and see the difference in Redis log entries for pages that are scraped the first time and those that are repeated. Before continuing further with the tutorial, stop the interactive monitor stream as a result of the above redis-cli command by pressing Ctrl + C keys while the interactive terminal is in focus.

Now that we are not mounting the /www folder inside the container, local changes should not reflect in the running service:
```bash
[node1] (local) root@192.168.0.8 ~/linkextractor
$ sed -i 's/Link Extractor/Super Link Extractor/g' www/index.php
```

Verify that the changes made locally do not reflect in the running service by reloading the web interface and then revert changes:
```bash
[node1] (local) root@192.168.0.8 ~/linkextractor
$ git reset --hard
HEAD is now at 3dbb7eb Synchronize branch step5
```


Now, shut these services down and get ready for the next step:
```bash
[node1] (local) root@192.168.0.8 ~/linkextractor
$ docker-compose down
Stopping linkextractor_api_1   ... done
Stopping linkextractor_web_1   ... done
Stopping linkextractor_redis_1 ... done
Removing linkextractor_api_1   ... done
Removing linkextractor_web_1   ... done
Removing linkextractor_redis_1 ... done
Removing network linkextractor_default
```

We have successfully orchestrated three microservices to compose our Link Extractor application. We now have an application stack that represents the architecture illustrated in the figure shown in the introduction of this tutorial. In the next step we will explore how easy it is to swap components from an application with the microservice architecture.

## Step 6: Swap Python API Service with Ruby
Checkout the step6 branch and list files in it.
```bash
[node1] (local) root@192.168.0.8 ~/linkextractor
$ git checkout step6
Branch 'step6' set up to track remote branch 'step6' from 'origin'.
Switched to a new branch 'step6'
[node1] (local) root@192.168.0.8 ~/linkextractor
$ tree
.
├── README.md
├── api
│   ├── Dockerfile
│   ├── Gemfile
│   └── linkextractor.rb
├── docker-compose.yml
├── logs
└── www
    ├── Dockerfile
    └── index.php

3 directories, 7 files
```

Some significant changes from the previous step include:

The API service written in Python is replaced with a similar Ruby implementation
The API_ENDPOINT environment variable is updated to point to the new Ruby API service
The link extraction cache event (HIT/MISS) is logged and is persisted using volumes
Notice that the ./api folder does not contain any Python scripts, instead, it now has a Ruby file and a Gemfile to manage dependencies.

Let’s have a quick walk through the changed files:
```bash
[node1] (local) root@192.168.0.8 ~/linkextractor
$ cat api/linkextractor.rb
```
```python
#!/usr/bin/env ruby
# encoding: utf-8

require "sinatra"
require "open-uri"
require "uri"
require "nokogiri"
require "json"
require "redis"

set :protection, :except=>:path_traversal

redis = Redis.new(url: ENV["REDIS_URL"] || "redis://localhost:6379")

Dir.mkdir("logs") unless Dir.exist?("logs")
cache_log = File.new("logs/extraction.log", "a")

get "/" do
  "Usage: http://<hostname>[:<prt>]/api/<url>"
end

get "/api/*" do
  url = [params['splat'].first, request.query_string].reject(&:empty?).join("?")
  cache_status = "HIT"
  jsonlinks = redis.get(url)
  if jsonlinks.nil?
    cache_status = "MISS"
    jsonlinks = JSON.pretty_generate(extract_links(url))
    redis.set(url, jsonlinks)
  end

  cache_log.puts "#{Time.now.to_i}\t#{cache_status}\t#{url}"

  status 200
  headers "content-type" => "application/json"
  body jsonlinks
end

def extract_links(url)
  links = []
  doc = Nokogiri::HTML(open(url))
  doc.css("a").each do |link|
    text = link.text.strip.split.join(" ")
    begin
      links.push({
        text: text.empty? ? "[IMG]" : text,
        href: URI.join(url, link["href"])
      })
    rescue
    end
  end
  links
end
```

This Ruby file is almost equivalent to what we had in Python before, except, in addition to that it also logs the link extraction requests and corresponding cache events. In a microservice architecture application swapping components with an equivalent one is easy as long as the expectations of consumers of the component are maintained.
```bash
[node1] (local) root@192.168.0.8 ~/linkextractor
$ cat api/Dockerfile
FROM       ruby:2.6
LABEL      maintainer="Sawood Alam <@ibnesayeed>"

ENV        LANG C.UTF-8
ENV        REDIS_URL="redis://localhost:6379"

WORKDIR    /app
COPY       Gemfile /app/
RUN        bundle install

COPY       linkextractor.rb /app/
RUN        chmod a+x linkextractor.rb

CMD        ["./linkextractor.rb", "-o", "0.0.0.0"]
```

Above Dockerfile is written for the Ruby script and it is pretty much self-explanatory.
```bash
[node1] (local) root@192.168.0.8 ~/linkextractor
$ cat docker-compose.yml
version: '3'

services:
  api:
    image: linkextractor-api:step6-ruby
    build: ./api
    ports:
      - "4567:4567"
    environment:
      - REDIS_URL=redis://redis:6379
    volumes:
      - ./logs:/app/logs
  web:
    image: linkextractor-web:step6-php
    build: ./www
    ports:
      - "80:80"
    environment:
      - API_ENDPOINT=http://api:4567/api/
  redis:
    image: redis
```

The docker-compose.yml file has a few minor changes in it. The api service image is now named linkextractor-api:step6-ruby, the port mapping is changed from 5000 to 4567 (which is the default port for Sinatra server), and the API_ENDPOINT environment variable in the web service is updated accordingly so that the PHP code can talk to it.

With these in place, let’s boot our service stack:
```bash
[node1] (local) root@192.168.0.8 ~/linkextractor
$ docker-compose up -d --build
Creating network "linkextractor_default" with thedefault driver
Building api
Step 1/10 : FROM       ruby:2.6
2.6: Pulling from library/ruby
6c33745f49b4: Already exists
c87cd3c61e27: Already exists
05a3c799ec37: Already exists
a61c38f966ac: Already exists
c2dd6d195b68: Already exists
5265e56e04c1: Pull complete
e4cba52ee328: Pull complete
52c657f93bbe: Pull complete
Digest: sha256:8fc798b044e47328c280a7c2b40156a02ccff82bda927234c1278bd7f2ff5148
Status: Downloaded newer image for ruby:2.6
 ---> 6ae7b3b686a8
Step 2/10 : LABEL      maintainer="Sawood Alam <@ibnesayeed>"
 ---> Running in abcb4b2e9334
Removing intermediate container abcb4b2e9334
 ---> e6525c47137b
Step 3/10 : ENV        LANG C.UTF-8
 ---> Running in d1a136724864
Removing intermediate container d1a136724864
 ---> 24b162643032
Step 4/10 : ENV        REDIS_URL="redis://localhost:6379"
 ---> Running in f702bc5c062c
Removing intermediate container f702bc5c062c
 ---> 604205c593fd
Step 5/10 : WORKDIR    /app
 ---> Running in 98d7bd273b06
Removing intermediate container 98d7bd273b06
 ---> 2e15508c7a1c
Step 6/10 : COPY       Gemfile /app/
 ---> ba2ea4e478d2
Step 7/10 : RUN        bundle install
 ---> Running in a25d05fac25a
Fetching gem metadata from https://rubygems.org/.......
Resolving dependencies...
Using bundler 1.17.2
Fetching mini_portile2 2.4.0
Installing mini_portile2 2.4.0
Fetching ruby2_keywords 0.0.2
Installing ruby2_keywords 0.0.2
Fetching mustermann 1.1.1
Installing mustermann 1.1.1
Fetching nokogiri 1.10.10
Installing nokogiri 1.10.10 with native extensions
Fetching rack-protection 2.1.0
Installing rack-protection 2.1.0
Fetching redis 4.2.5
Installing redis 4.2.5
Fetching tilt 2.0.10
Installing tilt 2.0.10
Fetching sinatra 2.1.0
Installing sinatra 2.1.0
Bundle complete! 3 Gemfile dependencies, 10 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
Removing intermediate container a25d05fac25a
 ---> 9accf7b5de58
Step 8/10 : COPY       linkextractor.rb /app/
 ---> ee4848892547
Step 9/10 : RUN        chmod a+x linkextractor.rb
 ---> Running in be3e44304940
Removing intermediate container be3e44304940
 ---> 32be2c4c2b33
Step 10/10 : CMD        ["./linkextractor.rb", "-o", "0.0.0.0"]
 ---> Running in 188154796c27
Removing intermediate container 188154796c27
 ---> 9426e2210b4e
Successfully built 9426e2210b4e
Successfully tagged linkextractor-api:step6-ruby
Building web
Step 1/3 : FROM       php:7-apache
 ---> fd505f1f4cd8
Step 2/3 : LABEL      maintainer="Sawood Alam <@ibnesayeed>"
 ---> Using cache
 ---> 9ace41e16c0d
Step 3/3 : COPY       . /var/www/html/
 ---> a3d1a946c865
Successfully built a3d1a946c865
Successfully tagged linkextractor-web:step6-php
Creating linkextractor_web_1   ... done
Creating linkextractor_redis_1 ... done
Creating linkextractor_api_1   ... done

/ops-stage3
```


We should now be able to access the API (using the updated port number)
```bash
Creating linkextractor_api_1   ... done
[node1] (local) root@192.168.0.8 ~/linkextractor
$ curl -i http://localhost:4567/api/http://example.com/
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 97
X-Content-Type-Options: nosniff
Server: WEBrick/1.4.2 (Ruby/2.6.6/2020-03-31)
Date: Thu, 17 Dec 2020 10:00:40 GMT
Connection: Keep-Alive

[
  {
    "text": "More information...",
    "href": "https://www.iana.org/domains/example"
  }
]
```

Now, open the web interface by clicking the Link Extractor and extract links of a few URLs. Also, try to repeat these attempts for some URLs. If everything is alright, the web application should behave as before without noticing any changes in the API service (which is completely replaced).

We can shut the stack down now:
```bash
[node1] (local) root@192.168.0.8 ~/linkextractor
$ docker-compose down
Stopping linkextractor_api_1   ... done
Stopping linkextractor_web_1   ... done
Stopping linkextractor_redis_1 ... done
Removing linkextractor_api_1   ... done
Removing linkextractor_web_1   ... done
Removing linkextractor_redis_1 ... done
Removing network linkextractor_default
```

Since we have persisted logs, they should still be available after the services are gone:
``bash
[node1] (local) root@192.168.0.8 ~/linkextractor
$ cat logs/extraction.log
1608199240      MISS    http://example.com/
```

This illustrates that the caching is functional as the second attempt to the http://example.com/ resulted in a cache HIT.

In this step we explored the possibility of swapping components of an application with microservice architecture with their equivalents without impacting rest of the parts of the stack. We have also explored data persistence using bind mount volumes that persists even after the containers writing to the volume are gone.

So far, we have used docker-compose utility to orchestrate the application stack, which is good for development environment, but for production environment we use docker stack deploy command to run the application in a Docker Swarm Cluster. It is left for you as an assignment to deploy this application in a Docker Swarm Cluster.

Conclusions
We started this tutorial with a simple Python script that scrapes links from a give web page URL. We demonstrated various difficulties in running the script. We then illustrated how easy to run and portable the script becomes onces it is containerized. In the later steps we gradually evolved the script into a multi-service application stack. In the process we explored various concepts of microservice architecture and how Docker tools can be helpful in orchestrating a multi-service stack. Finally, we demonstrated the ease of microservice component swapping and data persistence.

The next step would be to learn how to deploy such service stacks in a Docker Swarm Cluster.

As an aside, here are some introductory Docker slides.









