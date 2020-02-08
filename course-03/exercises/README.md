# Udagram Image Filtering Microservice

Udagram is a simple cloud application developed alongside the Udacity Cloud Engineering Nanodegree. It allows users to register and log into a web client, post photos to the feed, and process photos using an image filtering microservice.

The project is split into three parts:
1. [The Simple Frontend](/udacity-c3-frontend)
A basic Ionic client web application which consumes the RestAPI Backend. 
2. [The RestAPI Feed Backend](/udacity-c3-restapi-feed), a Node-Express feed microservice.
3. [The RestAPI User Backend](/udacity-c3-restapi-user), a Node-Express user microservice.

## Getting Setup

> _tip_: this frontend is designed to work with the RestAPI backends). It is recommended you stand up the backend first, test using Postman, and then the frontend should integrate.

### Installing Node and NPM
This project depends on Nodejs and Node Package Manager (NPM). Before continuing, you must download and install Node (NPM is included) from [https://nodejs.com/en/download](https://nodejs.org/en/download/).

### Installing Ionic Cli
The Ionic Command Line Interface is required to serve and build the frontend. Instructions for installing the CLI can be found in the [Ionic Framework Docs](https://ionicframework.com/docs/installation/cli).

### Installing project dependencies

This project uses NPM to manage software dependencies. NPM Relies on the package.json file located in the root of this repository. After cloning, open your terminal and run:
```bash
npm install
```
>_tip_: **npm i** is shorthand for **npm install**

### Setup Backend Node Environment
You'll need to create a new node server. Open a new terminal within the project directory and run:
1. Initialize a new project: `npm init`
2. Install express: `npm i express --save`
3. Install typescript dependencies: `npm i ts-node-dev tslint typescript  @types/bluebird @types/express @types/node --save-dev`
4. Look at the `package.json` file from the RestAPI repo and copy the `scripts` block into the auto-generated `package.json` in this project. This will allow you to use shorthand commands like `npm run dev`


### Configure The Backend Endpoint
Ionic uses enviornment files located in `./src/enviornments/enviornment.*.ts` to load configuration variables at runtime. By default `environment.ts` is used for development and `enviornment.prod.ts` is used for produciton. The `apiHost` variable should be set to your server url either locally or in the cloud.

***
### Running the Development Server
Ionic CLI provides an easy to use development server to run and autoreload the frontend. This allows you to make quick changes and see them in real time in your browser. To run the development server, open terminal and run:

```bash
ionic serve
```

### Building the Static Frontend Files
Ionic CLI can build the frontend into static HTML/CSS/JavaScript files. These files can be uploaded to a host to be consumed by users on the web. Build artifacts are located in `./www`. To build from source, open terminal and run:
```bash
ionic build
```
***

## Building and deploying the project

### Build docker images

Use following command to build local docker images for all the projects
```bash 
docker-compose -f ./udacity-c3-deployment/docker/docker-compose-build.yaml build --parallel
```

### Start docker containers 
1. Ensure that you have setup environment variables referenced in the file : ./udacity-c3-deployment/docker/docker-compose.yaml
2. start up docker containers using following command
```bash 
docker-compose -f ./udacity-c3-deployment/docker/docker-compose.yaml up
```
### Travis CI/CD Build

The master branch is linked to a Travis CI/CD build that triggers on every commmit , builds docker images from source code (using project specific dockerfiles) , the starts up a docker containers for those images and test that we are able to load up application welcome page on port 8100 : http://localhost:8100

## Kubernetes Deployment on AWS

The project can be deployed on K8S cluster , assuming you already have the cluster setup. We could issue following commands to deploy the application and perform changes.

All commands are executed from following location
```bash
cd ./udacity-c3-deployment/k8s
```

### Setup Environment variables

K8S deployment requires a couple of configuration details and some secrets. These will differ based upon the environment where this code is being deployed, thus needs to be specified first.

Configuration is specified in clear-text , but all the secrets are base-64 encoded

All configuration files mentioned below lives here : ./udacity-c3-deployment/k8s

1. aws-secret.yaml

base64 encoded AWS-Credentials , basically you would need to base64 encode : $HOME/.aws/credentials file

Can use following command to base64 encode the credentials file

```bash
openssl base64 -in <infile> -out <outfile>
```

2. env-configmap.yaml

These are application specific configuration , like details about AWS S3 bucket , database name , database hostname etc..

3. env-secret.yaml

postgres database username and password base64 encoded

can use following commands to generate base64 values

```bash
echo -n 'value to encode' | openssl base64
```

### Deploy Environment variables as configmaps and secrets

Deploy

```bash
kubectl apply -f aws-secret.yaml
kubectl apply -f env-secret.yaml
kubectl apply -f env-configmap.yaml
```

We can check the status of these using following commands
```bash
kubectl get configmaps
kubectl get secrets
```
### Deploy microservices 

Deploy scalable microservices 

```bash
kubectl apply -f backend-feed-deployment.yaml
kubectl apply -f backend-feed-service.yaml
kubectl apply -f backend-user-deployment.yaml
kubectl apply -f backend-user-service.yaml
kubectl apply -f frontend-deployment.yaml
kubectl apply -f frontend-service.yaml
kubectl apply -f reverseproxy-deployment.yaml
kubectl apply -f reverseproxy-service.yaml
```

Set up ports for consumption

```bash
kubectl port-forward service/reverseproxy 8080:8080
kubectl port-forward service/frontend 8100:8100
```
Can run following commands to check status of pods , deplyments , services etc..

```bash
kubectl get pods
kubectl get rs
kubectl get svc
kubectl get all
```

### Apply changes without disruption

Once you have done the changes to the microservice and pushed its image to docker hub , just run "apply" following command. It will create new pods with new version of your application, while still running old application in old pods. And then gardually move over all pods to new version of the application

```bash
kubectl apply -f backend-feed-deployment.yaml
```

### Scaling up and down

As per the load on application we may decide to run the application on more or fewer nodes

Scale Up

```bash
kubectl scale deployment/user --replicas=4
```

Scale Down

```bash
kubectl scale deployment/user --replicas=2
```
