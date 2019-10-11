# IBM Client Developer Advocacy App Modernization Series

## Lab - Migrating Legacy JEE apps to IBM Cloud Kubernetes Service

### Part 1 - Working with Helm

## Overview

Helm is a package manager for Kubernetes. Package managers automate the process of installing, configuring, upgrading, and removing computer programs. Examples include the Red Hat Package Manager (RPM), Homebrew, and Windows® PackageManagement.

An application in Kubernetes typically consists of at least two resource types: a deployment resource, which describes a set of pods to be deployed together, and a service resource, which defines endpoints for accessing the APIs in those pods. In additional to a deployment and a service, an application will typically include other Kubernetes resource types such as ConfigMaps, Secrets, and Ingress.

For any application in Kubernetes, you will need to run several Kubernetes commands (`kubectl`) to create and configure resources. With Helm, instead of manually creating each resource separately, you can create many resources with one command (`helm install`). This greatly simplifies the process and allows you to manage the related resources as a single unit called a Helm chart.

Helm charts are stored in a Helm chart repository, which is an HTTP server that houses packaged charts and an `index.yaml` file. The `index.yaml` file has an index of all the charts in the repository. A chart repository can be any HTTP server that can serve YAML and .tar files and can answer GET HTTP requests. Therefore, you have many options for hosting your chart repository such as a Google Cloud Storage bucket, an Amazon S3 bucket or you can create your own web server. For this lab, we will be using a local test repository provided by Helm to  test the hosting of a chart repository.

In this lab you'll create a Helm chart repository and use it to deploy a small Java EE app to the IBM Cloud Kubernetes Service using the open source Helm CLI.

The instructions were adapted from https://github.com/IBMAppModernization/app-modernization-helm-lab-iks.


### Pre-requisites

1. Obtaining and installing the lab tools on non-Windows 7 platform

    If you use web terminal during the lab, you may skip this section.

    You have done these installation steps, you may skip this section. You only do these tasks once.

    [Follow the instructions here](README_pre01.md)


### Setup

1. Lab Environment Setup

    Every time when you start a new terminal/command window, steps in the section must be performed to setup a new environment.

    [Follow the instructions here](README_pre02.md)

2. Initialize Helm client
```
   helm init --client-only
```


### Step 1: Clone the Github repo that contains the code, build an Open Liberty image of the app and then push it to the IBM Cloud Kubernetes container registry

1. Login in [your Github account](https://github.com)

2. In the search bar at the top left type in `app-modernization-plants-by-websphere-jee6`

    ![Search results](images/ss0.png)

3. Select the repository `IBMAppModernization\app-modernization-plants-by-websphere-jee6` and then click on the **Fork** icon

4. Click the **Clone or download** button from your copy of the forked repo and copy the HTTPS URL to your clipboard

    ![Clone URL](images/ss00.png)

5. From a client terminal window clone the Git repo  with  the following commands  appending the HTTPS URL from your clipboard

    ```text
    cd  ~
    git clone [HTTPS URL for NEW REPO]
    cd app-modernization-plants-by-websphere-jee6
    ```
6. Build the application .ear file using Maven by typing in (or copying and pasting in) the following command

    ```text
    mvn package
    ```

7. Add a new name space in registry to store your docker image

    ```
    ibmcloud  cr  namespace-add  movius_[your initial]
    export CRNAMESPACE=uy_[your initial]
    ```
    > Note: the new name space `uy_[your initial]` must be unique in the entire registry. If the new name space is not unique, change it slightly and try again.

8. Build a docker image  by typing in (or copying and pasting in) the following (uncommented) commands

    >Note: if you don't have the environment variables in the command set, replace $CRNAMESPACE with your container registry namespace and $USERNAME with your lab user id.

    ```bash
    docker build -t us.icr.io/$CRNAMESPACE/$USERNAME/pbw-mariadb-web:1.0.0 .
    ```

9. Log in to the Container registry with the following command:

    ```bash
    ibmcloud  cr login
    ```

10. Push the image to the IBM Container registry by typing in (or copying and pasting in) the following (uncommented) commands

    >Note: if you don't have the environment variables in the command set, replace $CRNAMESPACE with your container registry namespace and $USERNAME with your lab user id.

    ```bash
    docker push us.icr.io/$CRNAMESPACE/$USERNAME/pbw-mariadb-web:1.0.0
    ```

### Step 2: Look at the structure of the Helm chart for the Plants By WebSphere app

1. Login in [your Github account](https://github.com)

2. Select your fork  of the repository **app-modernization-plants-by-websphere-jee6** that you forked in the previous lab.

3. Using the Github's UI file browser to  take a look at the files in the **chart** folder. This is a Helm chart with child charts for the web app and MariaDB  portions of the app. Since there already is a published chart for  MariaDB, it is listed  as a required child chart in the file **requirements.yaml** and you don't have to create a chart for the MariaDB portion of the app.


### Step 3: Add a new chart to the local Helm repository

1. From your client terminal, execute

    ```
    helm repo add ibmcom https://raw.githubusercontent.com/IBM/charts/master/repo/stable
    ```

    This makes another repository (https://raw.githubusercontent.com/IBM/charts/master/repo/stable) accessable to you. `ibmcom` is the name of the new repository.

2. Execute command to list all available repositories for the verification purpose. `ibmcom` should be one in the list.

    ```
    helm repo list
    ```

3. Execute command to update all dependency of your chart.

    ```
    helm dependency update chart/pbw-liberty-mariadb
    ```
    Your chart has a dependency of chart `ibm-mariadb-dev`. This adds the tar file `ibm-mariadb-dev-1.1.2.tgz` to folder `chart/pbw-liberty-mariadb/charts`. The tar file(chart) comes from the new repository added in the previous step.

4. Package your chart and generate the chart archive.

    ```
    helm package chart/pbw-liberty-mariadb -d ~/.helm/repository/local

    Successfully packaged chart and saved it to: /Users/lee/.helm/repository/local/pbw-liberty-mariadb-1.0.0.tgz
    ```
    One of the repositories available to you is `local`. By default, it points to folder `~/.helm/repository/local/`. The above command creates a .tar file `pbw-liberty-mariadb-1.0.0.tgz` in your local repository folder. 
    
    As the result, a new chart becomes available in your local repository.


### Step 4: Configure Helm to serve up the local repository via HTTP

1. In your terminal window, type the following command, to start the local test Helm repository substituting for [PORT_NUMBER]. If you're using a web based terminal as part of an IBM instructor led workshop, use a port number derived from your username so it will be unique and not conflict with other users. The pattern is `9 + USER_NUMBER`. For example if your username is ``user023`` use port ``9023``, if your username is ``user009`` use port ``9009`` and so on. If you're using a terminal on your own machine use any free port number.

    ```
    helm serve --address 127.0.0.1:[PORT_NUMBER] &
    ```

2. In your terminal window type the following command. Verify that the contents of *index.yaml* are returned and it contains the chart archive you just added. Use the same port number you used in the previous instruction.

    ```
    curl http://127.0.0.1:[PORT_NUMBER]/charts/index.yaml
    ```


### Step 5: Deploy the legacy JEE app from your new Helm repo

1. In your terminal window type the following command, substituting your terminal user for [YOUR_USERNAME].  **Note**: Helm charts can be deployed multiple  times but each deployment must have a unique name

    >Note: if you don't have the environment variables in the command set, replace $CRNAMESPACE with your container registry namespace and $USERNAME with your lab user id.

    ```
    helm install --name pbw-liberty-mariadb --set liberty.image.registry=us.icr.io \
    --set liberty.image.namespace=$CRNAMESPACE/$USERNAME local/pbw-liberty-mariadb
    ```

### Step 6: Launch your deployed app

You'll commands to get the endpoint and port number of your deployed Helm release.

1. Run the following command to get the port number of your deployed app
    ```
    echo `kubectl --namespace default get service pbw-liberty-mariadb-liberty -o jsonpath='{.spec.ports[0].nodePort}'`
    ```

2. Run the following command to get the external IP address  of the first worker node in your cluster

    >Note: if you don't have the environment variables in the command set, replace $USERNAME with your lab user id.

    ```bash
    ibmcloud ks workers $USERNAME-cluster | grep -v '^*' | egrep -v "(ID|OK)" | awk '{print $2;}' | head -n1
    ```
3. In your browser's address bar enter the URL of your deployed app. The URL will be the external IP address of the first worker in your cluster followed by a colon and then followed by the port number of your deployed app. For example if your external IP is 169.61.73.182 and the port is 30961 the URL will be ```http://169.61.73.182:30961```

4. Verify that the app's UI opens in another tab. Click on the **HELP** link.

![Running app](images/ss4.png)

5. Click on **Reset database** to populate the MariaDB database with data

6. Verify that browsing different sections of the online catalog shows product descriptions and images.

![Online catalog](images/ss5.png)


## Cleanup

To remove the newly created name space and the newly deployed application,

```
ibmcloud  cr  namespace-rm  $CRNAMESPACE  -f
helm del --purge pbw-liberty-mariadb
```


## Summary

With even small simple apps requiring multiple Kubernetes objects,  Helm charts greatly simplify the process of distributing and updating your Kubernetes based apps. Helm repos allow you to distribute your Helm charts via HTTP, further simplifying the process of distributing and deploying your apps.
