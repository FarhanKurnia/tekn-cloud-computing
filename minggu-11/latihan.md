# Application Containerization and Microservice Orchestration

## Stage Setup
Let’s get started by first cloning the demo code repository, changing the working directory, and checking the demo branch out.
```bash
[node1] (local) root@192.168.0.23 
$ git clone https://github.com/ibnesayeed/linkextractor.git
[node1] (local) root@192.168.0.23 
$ cd linkextractor
[node1] (local) root@192.168.0.23 ~/linkextractor
$ git checkout demo
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
[node1] (local) root@192.168.0.23 ~/linkextractor
python linkextractor.py
Traceback (most recent call last):
  File "linkextractor.py", line 5, in <module>
    from bs4 import BeautifulSoup
ImportError: No module named bs4
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



