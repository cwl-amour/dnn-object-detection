<!--Put badges at the very top -->
<!--change the repo -->
<!-- [![Build Status](https://travis-ci.org/IBM/watson-banking-chatbot.svg?branch=master)](https://travis-ci.org/IBM/watson-banking-chatbot) -->
<!--Add a new Title and fill in the blanks -->
# [Analyze real time CCTV images with Convolutional Neural Networks]
In this Code Pattern, we will deploy an application that will leverage neural networking models to analyze RTSP (Real Time Streaming Protocol) video streams using with OpenCV / Darknet.

There are many surveillance cameras that have been installed, but cannot be closely monitored throughout the day. Since events are more likely to occur while the operator is not watching, many significant events go undetected, even when they are recorded. Users can't be expected to trace through hours of video footage, especially if they're not sure what they're looking for.

This project aims to alleviate this problem by using deep learning algorithms to detect movement, and identify objects in a video feed. These algorithms can be applied to both live streams and previously recorded video. After each video frame has been analyzed, the labeled screenshot and corresponding metadata are also uploaded to a Cloudant database. This allows for an operator to invoke complex queries and run analytics against the collected data. A few example queries might be
- Select all screenshots in which a person was detected at camera 3 during the previous Monday
- Get total count of cars detected last Saturday

When the reader has completed this Code Pattern, they will understand how to:
* Connect to a RTSP video stream via Python + Opencv
* Use Opencv and Numpy to process video frames and determine when significant motion has occured
* Identify objects in a photograph or video using a pre-built Deep Learning model

<!--add an image in this path-->
<!-- ![](doc/source/images/architecture.png) -->
<p align="center">
<img src="https://i.imgur.com/0RtWuzP.png" >
</p>

## Flow
1. Connect a motion detection script to a RTSP stream
2. If motion is detected, capture screenshot and forward to Node.js server hosted locally or in IBM Cloud container service
3. Analyze screenshot using Darknet / YOLO object detection algorithm
4. Upload labeled screenshot and associated metadata (time, camera channel) to Cloudant database
<!-- Whenever motion is detected, a screenshot is captured and sent to a endpoint hosted on a IBM Cloud hosted Node.js server. The screenshot is then analyzed, and the labeled screenshot and associated metadata is -->

<!--Optionally, update this section when the video is created-->
<!-- # Watch the Video
[![](http://img.youtube.com/vi/Jxi7U7VOMYg/0.jpg)](https://www.youtube.com/watch?v=Jxi7U7VOMYg) -->

# Steps
Use the ``Deploy to IBM Cloud`` instructions **OR** create the services and run locally.

<!--Optionally, add a deploy to ibm cloud button-->

## Deploy to IBM Cloud
<!--Update the repo and tracking id-->
<!-- [![Deploy to IBM Cloud](https://bluemix.net/deploy/button.png)](https://bluemix.net/deploy?repository=https://github.com/IBM/watson-banking-chatbot.git) -->

<!-- 1. Press the above ``Deploy to IBM Cloud`` button and then click on ``Deploy``. -->

<!--optional step-->
<!-- 2. In Toolchains, click on Delivery Pipeline to watch while the app is deployed. Once deployed, the app can be viewed by clicking 'View app'.
![](doc/source/images/toolchain-pipeline.png) -->

<!--update with service names from manifest.yml-->
1. Use the IBM Cloud dashboard to create the following services (The free or "lite" version will suffice for both services)
    * [Cloudant-db](https://console.bluemix.net/catalog/services/cloudant)
    * [Kubernetes](https://console.bluemix.net/containers-kubernetes/catalog/cluster)

2. Install the IBM Cloud CLI on your development system using the following [instructions](https://console.bluemix.net/docs/containers/cs_cli_install.html)

3. Install the Kubernetes CLI

*Linux*
```
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
```

*MacOS*
```
brew install kubernetes-cli
```

4. Export the `KUBECONFIG` path. This should be presented just after creating the container cluster
```
export KUBECONFIG=/Users/$USER/.bluemix/plugins/container-service/clusters/mycluster/kube-config-hou02-mycluster.yml
```

5. Deploy the kubernetes application with the following command
```
kubectl apply -f kube-config.yml
```

6. Find the public ip address of the Kubernetes cluster
```
# Get id of cluster
ibmcloud ks clusters
# Print workers associated with cluster, take note of public ip
ibmcloud ks workers <cluster_id>
```

7. Confirm that the Node.js backend is up and running
```
curl <worker_public_ip>:30000/status
```

## Run locally
> NOTE: These steps are only needed when running locally instead of using the ``Deploy to IBM Cloud`` button.

<!-- there are MANY updates necessary here, just screenshots where appropriate -->

If Docker is installed on your system, simply running the following command will start the backend service
```
docker run -d -p 3000:3000 kkbankol/opencv_yolo_pod
```

If Docker is not installed, continue with the following steps
1. [Clone the repo](#1-clone-the-repo)
2. [Create Watson services with IBM Cloud]()
3. [Configure credentials]()
4. [Install backend dependencies](#4-run-the-application)
5. [Start the backend node.js server](#4-run-the-application)

### 1. Clone the repo

Clone the `dnn-object-detection` project locally. In a terminal, run:

```
$ git clone https://github.com/IBM/dnn-object-detection
```

### 2. Create Watson services in IBM Cloud dashboard

Create the following services:

* [**Cloudant DB**](https://console.bluemix.net/catalog/services/cloudant)

### 3. Configure credentials
The credentials for IBM Cloud services (Cloudant DB), can be found in the ``Services`` menu in IBM Cloud by selecting the ``Service Credentials`` option for each service.
> Store the **cloudant_username** and **cloudant_password** into the `.env` file
```
cloudant_username=<>
cloudant_password=<>
```

### 4. Install backend dependencies
```
npm install
```

### 5. Start the backend Node.js server
1. Install [Node.js](https://nodejs.org/en/) runtime or NPM.
1. Start the app by running `npm install`, followed by `npm start`.
> Note: server host can be changed as required in server.js and `PORT` can be set in `.env`.

<!--Add a section that explains to the reader what typical output looks like, include screenshots -->
```
Kalonjis-MacBook-Pro:dnn-object-detection kkbankol@us.ibm.com$ npm start

> cameras-app@0.0.0 start /Users/kkbankol@us.ibm.com/projects/smart-city-cameras/dnn-object-detection
> node ./bin/www
```

## Set up the Raspberry Pi Client
Now that we have a backend process up and running, we'll set up a device on the same local network as the CCTV cameras. The reasoning for this is that continuously pulling multiple video streams would be too demanding on the network bandwidth, and there would likely be latency issues. So as an alternative, we'll set up a Raspberry Pi on the same network as the CCTV system, and connect the two devices over the LAN instead.

We'll start by installing a few dependencies for our "motion detection" script.
```
sudo apt-get update
sudo apt-get install python ffmpeg -y
pip install numpy cv2 requests
```

Next, we can actually begin processing video that has either been pre-recorded or being live streamed. The script expects two arguments. The first argument is the video source (file or RTSP stream). The second argument is the ip address where images/metadata can be sent, this ip will either be 127.0.0.1 (if running node backend locally), or the public ip of the Kubernetes cluster.
```
# process pre-recorded video
python cv_object_detection.py sample_videos/vid10.mp4 <nodejs_ip>

# process live stream video
python cv_object_detection.py rtsp://<user>@<pass>192.168.1.2:8080/out.h264 <nodejs_ip>
```

Once this script begins, it'll iterate through each frame from the video source. As these iterations continue, the Opencv libary is used to calculate a "running average". Each frame gets compared to the running average, and if a significant difference is observed, the frame is then saved as an image and forwarded to the nodejs backend.

As images are uploaded to the Node backend, they are then processed by the YOLO object detection algorithm, and labeled
<p align="center">
<img src="https://i.imgur.com/Ez2hDGj.png" >
</p>

The labeled photo, identified classes, and metadata are then uploaded to Cloudant. Each Cloudant document is arranged like so
<p align="center">
<img src="https://i.imgur.com/1oMSAYE.png" >
</p>

<!--Optionally, include any troubleshooting tips (driver issues, etc)-->

# Troubleshooting

* Error: Environment {GUID} is still not active, retry once status is active

  > This is common during the first run. The app tries to start before the Discovery
environment is fully created. Allow a minute or two to pass. The environment should
be usable on restart. If you used `Deploy to IBM Cloud` the restart should be automatic.

* Error: Only one free environent is allowed per organization

  > To work with a free trial, a small free Discovery environment is created. If you already have
a Discovery environment, this will fail. If you are not using Discovery, check for an old
service thay you may want to delete. Otherwise use the .env DISCOVERY_ENVIRONMENT_ID to tell
the app which environment you want it to use. A collection will be created in this environment
using the default configuration.

<!--keep this-->


# Sources
- [YOLO](https://pjreddie.com/darknet/yolo/)
- [OpenCV Motion Detection Script (Used for cv_object_detection.py)](https://github.com/iandees/speedtrack)


# License
[Apache 2.0](LICENSE)
