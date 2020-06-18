# Lab - Getting Started with OpenShift for Developers


* [Overview](#overview)
* [Deploying the Workshop](#deploying-the-workshop)
  * [Deploying on Red Hat Product Demo System](#deploying-on-red-hat-product-demo-system)
  * [Deploying to an OpenShift Cluster](#deploying-to-an-openshift-cluster)
* [Running the Workshop](#running-the-workshop)
* [Deploying the Lab Guides Only](#deploying-the-lab-guides-only)
* [Development](#development)
* [Deleting the Workshop](#deleting-the-workshop)

## Overview

| | |
--- | ---
| Audience Experience Level | Beginner |
| Supported Number of Users | Up to 100 per cluster |
| Average Time to Complete | 90 minutes |

This workshop is intended to give you a hands-on introduction to using OpenShift from the perspective of a developer.

Containers are a standardized way to package apps with all of their dependencies to simplify deployment and speed delivery. Unlike virtual machines, containers do not bundle to the operating system. Only the application code, run time, libraries, and settings are packaged inside of containers. Thus, containers are more lightweight, portable, and efficient as compared to virtual machines.

For developers looking to kickstart their projects, OpenShift enables efficient application development through streamlined workflows and validated integrations.

### Objectives

* Using the OpenShift command-line client and web console.
* Deploying an application using a pre-existing container image.
* Working with application labels to identify component parts.
* Scaling up your application in order to handle web traffic.
* Exposing your application to users outside of the cluster.
* Viewing and working with logs generated by your application.
* Accessing your application container and interacting with it.
* Giving access to other users to collaborate on your application.
* Deploying an application from source code in a Git repository.
* Deploying a database from the OpenShift developer catalog.
* Configuring an application so it can access a database.
* Setting up webhooks to enable automated application builds.
* Additional topics may also be covered relevant to the specific programming language used by the applications being deployed.

There are 4 programming language variants of the workshop:
* Java
* Node.js
* Python
* PHP

### Components Used

The full workshop contains several components:
* Etherpad - So users can claim a username
* A GOGS server and GOGS repositories for each user
* Nexus - Currently only used by the Java version of the workshop
* OCP Ops View - An instance of the [ops-view](https://github.com/hjacobs/kube-ops-view) cluster visualizer
* An instance of the homeroom workshop chooser application and the 4 starter lab guides (Java, Node.js, Python, PHP) from this repository.

## Deploying the Workshop
This workshop is designed to be deployed from [Red Hat Product Demo System (RHPDS)](https://rhpds.redhat.com). 

### Deploying on Red Hat Product Demo System
Upon logging into RHPDS, highlight the **Services** sidebar, and select the **Catalogs** menu. The workshop is found in the catalog under the **Workshops** folder and is named **OCP4 - Getting Started Workshop**.

Follow the directions in the [Running the Workshop](#running-the-workshop) section to begin the workshop itself.

**Note:** It will take up to 75 minutes to deploy the OCP4 cluster via RHPDS.

### Deploying to an OpenShift Cluster

The recommended way to deploy this workshop is directly from the RHPDS catalog as described above. If you'd like to deploy it manually, you can order the base OpenShift 4.2 Workshop and deploy the Getting Started workshop via the instructions below.

**Prerequisites**
* An OpenShift 4.3 or 4.4 Workshop cluster from [Red Hat Product Demo System (RHPDS)](https://rhpds.redhat.com). This cluster is available in the catalog in the **Workshops** folder and is named **OpenShift 4.4 Workshop**.

[AgnosticD](https://github.com/redhat-cop/agnosticd) is used to deploy the workshop, which provides a deploying infrastructure to build and configure application environments.

**Deploying**

1. First, log into the OpenShift cluster where you want to deploy the workshop. You'll need to log in with cluster admin permissions.

2. Next, clone the AgnosticD repository (or your fork of it, if you are making changes):

```
git clone https://github.com/redhat-cop/agnosticd
```

3. In your terminal, cd into the agnosticd repository and run:

```
docker run -it --rm -v $(pwd):/opt/app-root/src -v $HOME/.kube:/opt/app-root/src/.kube \
--entrypoint bash quay.io/osevg/agnosticd-runner
```
   This will get you a bash shell into an AgnosticD enabled environment. In this environment, you'll be able to run or test AgnosticD workloads.

4. cd into the ansible directory:
```
cd ansible
```

5. Run the following script to deploy all the components of the starter workshop. Change the value of `num_users` and `user_count` to match the number of users you want to provision for the workshop.

```
TARGET_HOST=localhost
GUID=sampleuser
ocp_username=opentlc-mgr
# WORKLOAD SPECIFICS
WORKSHOP_PROJECT=lab
workloads=("ocp-workload-etherpad" \
           "ocp-workload-ocp-ops-view" \
           "ocp-workload-gogs" \
           "ocp4-workload-nexus-operator" \
           "ocp-workload-gogs-load-repository" \
           "ocp4-workload-homeroomlab-starter-guides")

for WORKLOAD in ${workloads[@]}
do
  ansible-playbook -c local -i ${TARGET_HOST}, configs/ocp-workloads/ocp-workload.yml \
      -e ansible_python_interpreter=/opt/app-root/bin/python \
      -e ocp_workload=${WORKLOAD} \
      -e guid=${GUID} \
      -e project_name=${WORKSHOP_PROJECT} \
      -e etherpad_project=${WORKSHOP_PROJECT} \
      -e gogs_project=${WORKSHOP_PROJECT} \
      -e opsview_project=${WORKSHOP_PROJECT} \
      -e '{ "ocp4_workload_nexus_operator_vars": {"project": "${WORKSHOP_PROJECT}"} }' \
      -e skip_tls_verify=true \
      -e project_name=${WORKSHOP_PROJECT} \
      -e ocp_username=${ocp_username} \
      --extra-vars '{"num_users": 5, "user_count": 5, "ACTION": "create"}'
done
```

**Note:** If you would like participants to be able to do the optional cluster logging aspect of the lab (with Kibana) you need to add the following to the list of workloads (or run this workload separately after all the other workloads):
```
workloads=("ocp4-workload-logging")
```
and you need to provide an --extra-var that tells agonstic about the cloud provider.  For instance, if your cluster is running in AWS, you need to add the following to the `ansible-playbook` command line above

```
-e cloud_provider="ec2"
```
## Running the Workshop

### Starting the Workshop for Participants
Once the deployment finishes, navigate to the OpenShift administrator perspective. After, go to the `labs` project and view the Routes there. The structure of the Route URLs is as follows. If your GUID is, for example, `abc-1234`, and the route name is `myroute`, the Route URL will be http://`myroute`-labs.apps.cluster-`abc-1234`.`abc-1234`.example.opentlc.com

* The `etherpad` route is for the etherpad deployment. Append `/p/workshop` to the end of this route and share that URL with lab participants so they can claim a username.
* The `homeroom` route is the one that launches the workshop chooser. Give this URL to lab participants after they've claimed a username.




## Deploying the Lab Guides Only
**Note**: For this workshop, you will typically want to deploy the full workshop, per the instructions above. Deploying the lab guides only is normally only done if you are making changes to the lab guide content and want to quickly verify and view your changes.

1. To deploy the lab guides only, first clone this Git repository (or your fork of it, if you are making changes) to your own machine. Use the command:

```
git clone --recurse-submodules https://github.com/openshift-labs/starter-guides.git
```

The ``--recurse-submodules`` option ensures that Git submodules are checked out. If you forget to use this option, after having cloned the repository, run:

```
git submodule update --recursive --remote
```

2. Next, create a project in OpenShift into which the workshop is to be deployed. You must be logged in as cluster admin to deploy the guides.

```
oc new-project workshops
```

3. From within the top level of the Git repository, now run:

```
.workshop/scripts/deploy-spawner.sh
```

The name of the deployment will be ``lab-getting-started``.

4. You can determine the hostname for the URL to access the workshop by running:

```
oc get route lab-getting-started
```

When the URL for the workshop is accessed you will be prompted for a user name and password. Use your email address or some other unique identifier for the user name. This is only used to ensure you get a unique session and can attach to the same session from a different browser or computer if need be. The password you must supply is ``openshift``.

## Development


The deployment created above will use an image from [Quay.io](https://quay.io/) for this workshop, a container automation platform, based on the ``ocp-4.2`` branch of the repository.

To make changes to the workshop content and test them, edit the files in the Git repository and then run:

```
.workshop/scripts/build-workshop.sh
```

This will replace the existing image used by the active deployment.

If you are running an existing instance of the workshop select "Restart Workshop" from the menu top right of the workshop environment dashboard.

When you are happy with your changes, push them back to the remote Git repository.

## Deleting the Workshop


To delete the spawner and any active sessions, including projects, run:

```
.workshop/scripts/delete-spawner.sh
```

To delete the build configuration for the workshop image, run:

```
.workshop/scripts/delete-workshop.sh
```


