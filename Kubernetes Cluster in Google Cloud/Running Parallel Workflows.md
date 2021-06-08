# Producing the NanoAODs by Running Parallel Workflows

This was an attempt to run parallel workflows in Google Cloud using a Kubernetes cluster, 
to scale up the NanoAOD production of the Higgs to 4 leptons analysis samples from CERN's [Open Data Portal](http://opendata.cern.ch/record/5500). 
I followed Asdrubal Cruz's [CMS Parallel Job](https://github.com/asdru30/CMSParallelJobKubernetes) to understand what to do, to then attempt my scale up.

## Setup

Sign-up or Log-in to [Google Cloud](https://cloud.google.com/). New customers will get $300 credit to use in the platform. 
You can use these credits for things such as creating your Kubernetes cluster, running workflows and creating a disk storage for your workflow's outputs.

## Creating the Kubernetes cluster

Once you are in the Google Cloud platform, head to the navigation menu on the top-left of the tool bar, then to the Kubernetes Engine tab to create your cluster.
Choose to *create* and *configure* a standard cluster. You will get a screen to set-up your cluster. Do the followoing:
* You can leave the cluster's name as is
* You can leave the cluster's zone as the default 
* Choose the *default-pool* tab on the left and change the number of nodes to 6
* Choose the *Nodes* tab on the left and change the *Machine type* to e2-standard-4 (4 vCPU, 16 GB memory)
* Create the cluster

## Activate the Cloud shell, further set-up

Once the cluster is created, activate the Cloud Shell, clicking the icon on the top right of the tool bar
![Cloud Shell](https://user-images.githubusercontent.com/64757758/121252923-910a4e80-c876-11eb-9a68-2bc387d6f0ef.png)

In the Shell, log-in to your account
```bash
gcloud auth login
```
Then, create a configuration file for your cluster
```bash
gcloud container clusters get-credentials CLUSTERNAME --zone CLUSTERZONE --project YOURPROJECTNAME
```
Note:
* Substitute your cluster's name in *CLUSTERNAME*
* Substitute your cluster's zone in *CLUSTERZONE*
* Substitute your Google Cloud project's name in *YOURPROJECTNAME*

Install argo as the workflow engine
```bash
kubectl create ns argo
kubectl apply -n argo -f https://raw.githubusercontent.com/argoproj/argo/stable/manifests/quick-start-postgres.yaml
curl -sLO https://github.com/argoproj/argo/releases/download/v2.11.1/argo-linux-amd64.gz
gunzip argo-linux-amd64.gz
chmod +x argo-linux-amd64
sudo mv ./argo-linux-amd64 /usr/local/bin/argo
```

Give argo sufficient rights to manage a workflow pod
```bash
kubectl create clusterrolebinding YOURNAME-cluster-admin-binding --clusterrole=cluster-admin --user=ACCOUNTEMAIL
```
Note:
* Substitute your name in *YOURNAME*
* Substitute your Google Cloud account email in ACCOUNTEMAIL

Next, I downloaded Asdrubal's [CMS Parallel Job](https://github.com/asdru30/CMSParallelJobKubernetes) repository
```bash
git clone https://github.com/asdru30/CMSParallelJobKubernetes.git
```

Head to the repository and apply a patch to the default argo configuration file
```bash
cd CMSParallelJobKubernetes
kubectl patch configmap workflow-controller-configmap -n argo --patch "$(cat patch-workflow-controller-configmap.yaml)"
```

Create a NFS persistent volume where the output files produced from the jobs will be stored
```bash
gcloud compute disks create --size=300GB --zone=CLUSTERZONE gce-nfs-disk-1
```
Note:
* Substitute your cluster's zone
* *gce-nfs-disk-1* is the name given to the disk


Head to the *volume_config* directory to mount the created disk as the volume for the workflow
```bash
cd volume_config
kubectl apply -n argo -f 001-nfs-server.yaml
kubectl apply -n argo -f 002-nfs-server-service.yaml
```
Before applying the third file, *003-pv-pvc.yaml*, get your cluster IP to substitute it in the this file
```bash
kubectl get -n argo svc nfs-server |grep ClusterIP | awk '{ print $3; }’
```
Copy the given IP and edit *003-pv-pvc.yaml*, with a text editor, and paste the IP  
![cluster ip substitution](https://user-images.githubusercontent.com/64757758/121260262-17c32980-c87f-11eb-8ab2-7720390d8d6a.png)

Apply the third file,
```bash
kubectl apply -n argo -f 003-pv-pvc.yaml
```

## Submit the workflows with argo

Head to where the workflows are located (assuming you were still at *volume_config*) and submit one (for this example, *parallel-DYJetsToLL.yaml*),
```bash
cd ../parallel_wf
argo submit -n argo parallel-DYJetsToLL.yaml --watch
```
Note:
* The *--watch* flag allows you to see a continuos run of your workflow


## Errors I got 

When I ran this workflow, I got the following error:
![workflow error](https://user-images.githubusercontent.com/64757758/121265059-2e20b380-c886-11eb-9317-b24e8e3a5a4b.png)
I fixed it by editing the file's imagePullPolicy from *Never* to *Always*
![imagePullPolicy fix](https://user-images.githubusercontent.com/64757758/121265698-32999c00-c887-11eb-8c1d-1795c8cb7776.png)

The workflow ran, although never finished. I made a yaml file of the same workflow, but with only one rootfile to test if the job finishes when submitted, and it did. 
Although, when I ran it again at a later date, it did not finish running the job. 
You can go to the navigation menu on the top-left and, on the Kubernetes Engine tab, you can go to the *Workloads* tab to see details of your submitted jobs.
I found that the submitted workflow had a pod with an "unready" status, but could not find anything to fix that.  


## Accesing Your Workflow Output File via a http Server

This is done so you are able to download the workflow's output to your local computer. 

In *CMSParallelJob/http_server*, Asdrubal had already downloaded the needed configuration files to create the webserver. Assuming you are at the *parallel_wf* directory,
head to the *http_server* directory and patch the config file for the webserver
```bash
cd ../http_server
kubectl create configmap basic-config --from-file=conf.d -n argo
```

Next, deploy the file server and apply and expose the port as a LoadBalancer
```bash
kubectl create -f deployment-http-fileserver.yaml
kubectl expose deployment http-fileserver --type LoadBalancer --port 80 --target-port 80
```

Follow the status of the server until it gives you the External IP
```bash
kubectl get svc -n argo
```

Once you get the External IP, copy/paste it to a web browser tab. In order to enable the file browsing, delete the *index.html* file in the pod. 
First, get the pod's name. Then, substitute it in *PODNAME* on the second command
```bash
kubectl get pods
kubectl exec PODNAME -- rm /usr/share/nginx/html/index.html
```
Note: 
I could not get over this step as the pod remained with a pending status, that no host was assigned to it. I did not find anything to fix the issue.


Once the file is deleted, you should now be able to browse for your files and download them. Once done, delete the service as the IP can be accessed from anywhere in the world
```bash
kubectl delete svc/http-fileserver -n argo
```

## Avoid additional charges

To avoid additional charges on the Google Cloud platform, you can delete the created cluster, workflows, and disk.
* Delete your clusters on the *Clusters* tab on the *Kubernetes Engine* tab from the navigation menu.
* Delete your workflows on the *Workflows* tab on the *Kubernetes Engine* tab from the navigation menu.
* Delete your disks on the *Disks* tab on the *Compute Engine* tab from the navigation menu.







