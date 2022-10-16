# build-deploy-guestbook-app

Project Worked on Apart of Introduction to Containers w/ Docker, Kubernetes & OpenShift Course.
Builds, Deploys, and Autoscales Basic application using Docker initially and then uses Openshift Internal Registry to Deploy with Redis master and Redis slaves for Redeploying app without relying on in-memory datastore to save lists.

# Building Guestbook App With Docker

Below is the basic Guestbook App that takes in user input and lists out the names on the page.
<img width="960" alt="headerchange" src="https://user-images.githubusercontent.com/97641116/196009068-8deb0692-5167-4b64-b358-a94e4250afb2.PNG">

With the Dockerfile, we used the command below to build the guestbook app using Docker.
The second part of the command is a the enxt command to push the image created to IBM Cloud Container Registry.

`docker build . -t us.icr.io/$MY_NAMESPACE/guestbook:v1 && docker push us.icr.io/$MY_NAMESPACE/guestbook:v1`

We next applied to the deployment withe file `demployment.yml` using the command:

`kubectl apply -f deployment.yml`

Using `kubectl port-forward deployment.apps/guestbook 3000:3000` we can launch the application on the set port number to view the live application.

# Autoscale using Horizontal Pod Autoscaler:

To deploy with autoscale we used the command:

`kubectl autoscale deployment guestbook --cpu-percent=5 --min=1 --max=10`

Here we set our minumum pod to 1 and max to 10 for the ReplicaSet with a threshold of 5 percent so stimualte the scaling.

With the `kubectl get hpa guestbook` command we can see the status of the autoscaler.
<img width="460" alt="hpa" src="https://user-images.githubusercontent.com/97641116/196009651-df5db431-3563-49f4-9a83-1f585fa43b8f.PNG">

Using `kubectl run -i --tty load-generator --rm --image=busybox:1.35.0 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- sn-labs-munagalarish; done"` in a different terminal to simulate stress

We can see the number of pods increasing in real time as the app passes the threshold with the command:
`kubectl get hpa guestbook --watch`

<img width="483" alt="hpa2" src="https://user-images.githubusercontent.com/97641116/196009704-91835c56-97b3-4f65-9020-c6c0a87ac421.PNG">

# Delpoy Guestbook App with Openshift Internal Registry:

This implementation had thwe goal of reducing latency when scanning and pulling images. 

We start by creating an image stream that points to the IBM Cloud Container Registry with:
`oc tag us.icr.io/$MY_NAMESPACE/guestbook:v1 guestbook:v1 --reference-policy=local --scheduled`

Then with OpenShift Console, we create a Container Image from our internal registry.
<img width="958" alt="creatingContainer" src="https://user-images.githubusercontent.com/97641116/196009841-1538de18-4dd5-4201-b9bd-df8c3da370aa.PNG">

<img width="951" alt="openshiftautoscale" src="https://user-images.githubusercontent.com/97641116/196009890-07c21402-f4ba-43cb-a124-ae80b2be390b.PNG">

This is the link found at the bottom of the resources tab for our deployed application:
http://build-deploy-guestbook-app1-sn-labs-munagalarish.labs-prod-openshift-san-a45631dc5778dc6371c67d206ba9ae5c-0000.us-east.containers.appdomain.cloud/

Now when we make changes and use docker to build and push the app, Openshift will regualarly import the new images pushed.

<img width="959" alt="displayedupdate" src="https://user-images.githubusercontent.com/97641116/196009943-d3556b5b-14fb-4c44-8226-85083f21e122.PNG">

# Deploy Redis master and slave:

We now delpoy the Redis master for storage and Redis slaves for transfering the data. This can help preserve the data even when new updates are rolled out keeping persistent storage.

With these set of commands:
`oc apply -f redis-master-deployment.yaml`
`oc apply -f redis-master-service.yaml`
`oc apply -f redis-slave-deployment.yaml`
`oc apply -f redis-slave-service.yaml`

We can deploy Redis slave and master as shown in our Topology:

<img width="957" alt="redis" src="https://user-images.githubusercontent.com/97641116/196010074-439c7024-7566-4975-b8af-58a971f82756.PNG">

Finally we deploy the guestbook app using OpenShift with the Dockerfile from the repository. Now the app is no longer depended on using in-memory datastore and can preserve the data.

<img width="960" alt="deployedfromgithub" src="https://user-images.githubusercontent.com/97641116/196010118-59999fac-6f90-4069-b98b-cce5cf054970.PNG">
