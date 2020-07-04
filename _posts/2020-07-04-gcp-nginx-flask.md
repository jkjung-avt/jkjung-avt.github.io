---
layout: post
comments: true
title: "Setting Up a NGINX + Flask Server on GCP"
excerpt: "This post is a brief note documenting how I set up a NGINX + Flask server using a Compute Engine on Google Cloud Platform (GCP)."
date: 2020-07-04
category: "gcp"
tags: gcp
---

I have always been wanting to set up a dashboard on the cloud which I could use to monitor IoT products and devices on the field.  I was aware that AWS, Azure and GCP all have offerings of IoT Core services.  But I think they are overkill for basic device monitoring.  Those solutions offered by the cloud vendors are generally too complicated for basic device monitoring and too expensive.

As an alternative, I'd like to look into possibilities of setting up an HTTP server (dashboard) with a free (or very-low-cost) compute instance on the cloud.  I made such an attempt on Google Cloud first.  This post documents the key steps I applied to set up a NGINX + Flask server using a entry-level Compute Engine on GCP.

# Prerequisite

* Sign up for an account on GCP: [https://cloud.google.com/free/](https://cloud.google.com/free/).  (You get a credit of USD 300 when signing up for the first time.)

* In addition to accessing [GCP console](https://console.cloud.google.com/home/dashboard) with a web browser, I also installed [Google Cloud SDK](https://cloud.google.com/sdk/) on my local PC for easy access from the command line.  I used a Ubuntu Linux PC for that.  (You could actually use whatever PC/Mac platform of your own preference.)

# Step #1 - Create the Compute Engine on GCP

Reference (in Chinese): [https://www.yannyann.com/2018/02/wp-ssl-ubuntu-lamp-nginx-varnish-redis-2/](https://www.yannyann.com/2018/02/wp-ssl-ubuntu-lamp-nginx-varnish-redis-2/)

Key steps:

1. Create a new project, e.g. "nginx-test" with "nginx-test-282303" as the project id.  Make sure the newly created project is active on the [GCP console](https://console.cloud.google.com/home/dashboard).

2. Under "Compute Engine", do "Create a new VM instance".
    - I left the "Name" as "instance-1".
    - In order to lower network latency, I chose "asia-east1 (Taiwan)" as the "Region" and "asia-east1-b" as the "Zone".
    - In order to lower the cost, I chose "N1" "Series" and "f1-micro (1vCPU, 614 MB memory)" "Machine Type".  This resulted in "$5.00 monthly estimate".  (You might be able to use the free tier f1-micro instance if you choose the Compute Engine from a US Region.)
    - As to "Boot disk", I chose "Ubuntu" and "Ubuntu 18.04 LTS".
    - I checked both boxes of "Allow HTTP traffic" and "Allow HTTP traffic".
    - Under "Management, security, disks, networking, sole tenancy" -> "networking", I set "Hostname" as "instance-1.nginx-test".
    - I hit the "Create" button, and waited for the Compute Engine to be created.

3. I also did the following in order to set a fixed IP address for my "instance-1".
    - I selected "instance-1" from "Compute Engine" -> "VM instances".
    - Go to "Networking"/"VPC Network" -> "External IP addresses".
    - Promote the ephemeral external IP address of "instance-1" to "Static".

# Step #2 - Set up Google Cloud SDK locally and connect to the Compute Engine ("instance-1")

Reference: [Quickstart for Debian and Ubuntu](https://cloud.google.com/sdk/docs/quickstart-debian-ubuntu)

1. I followed the Quickstart document and installed gcloud on my local Ubuntu PC.

2. During `gcloud init`, I chose project "nginx-test-282303" and compute zone "asia-east1-b".

3. When done, I was able to easily ssh from my local PC to my Compute Engine with:

    ```shell
    ### On my local PC
    $ gcloud compute ssh instance-1
    ```

# Step #3 - Set up NGINX on the Compute Engine ("instance-1")

Reference: [How To Install Nginx on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-18-04)

1. Install NGINX on the Compute Engine.  Note the following should be executed on the Compute Engine (ssh'ed on "instance-1").

    ```shell
    ### On the CGP Compute Engine instance
    $ sudo apt update
    $ sudo apt install nginx
    ```

2. By default, network firewall is not enabled on the Compute Engine.  It could be verified as below.  As a result, I did not need to alter firewall settings on the Compute Engine.

    ```shell
    $ sudo ufw status
    Status: inactive
    ```

3. At this point, the NGINX server should have been running and could accept connections.  Find out the "external IP" address of the Compute Engine ("instance-1" on GCP console, e.g. "34.80.xxx.yyy".  Then open the default web page with a web browser: "http://34.80.xxx.yyy".  You should see the following content.

    ![NGINX default page](https://assets.digitalocean.com/articles/nginx_1604/default_page.png)

# Step #4 - Set up Flask

Reference: [How To Serve Flask Applications with uWSGI and Nginx on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-uswgi-and-nginx-on-ubuntu-18-04)

1. Install Flask.

    ```shell
    $ sudo apt update
    $ sudo apt install python3-pip python3-dev build-essential \
                       libssl-dev libffi-dev python3-setuptools
    $ sudo python3 -m pip install -U pip wheel
    $ sudo python3 -m pip install flask
    ```

2. Create the Flask app.

    ```shell
    $ mkdir ${HOME}/flask
    $ nano $HOME/flask/version.py
    ```

    And put the following content into "$HOME/flask/version.py".

    ```python
    from flask import Flask, jsonify
    app = Flask(__name__)

    @app.route("/api/v1/ver")
    def get_api():
        version = { 'version': '0.1' }
        return jsonify(version)
    ```

3. Add the proxy (forwarding) rule for Flask in NGINX' settings.  More specifically, edit "/etc/nginx/sites-available/default", e.g. `sudo nano /etc/nginx/sites-available/default`.  Add the following lines.

    ```
            location / {
                    # First attempt to serve request as file, then
                    # as directory, then fall back to displaying a 404.
                    try_files $uri $uri/ =404;
            }

    +       location ^~ /api/v1 {
    +               proxy_pass http://127.0.0.1:5000;
    +               proxy_set_header Host $host;  # preserve HTTP header for proxy requests
    +       }
    ```

    Restart NGINX with the new setting.

    ```shell
    $ sudo nginx -s reload
    ```

4. Run the Flask app.

    ```shell
    $ python3 ${HOME}/flask/version.py
    ```

5. Open the web page with a web browser again: "http://34.80.xxx.yyy/api/v1/ver".  The output should read: `{"version":"0.1"}`
