
Explanation of microservices example
At the end of the day, what happens is that the workloads.yml is our script where we deploy microservices. Here, we created microservices (ActiveMq, position-simulator, position-tracker, api-gateway, webapp).
 
To enable the necessary microservices to get IPs, we defined services using services.yml, these microservices being ActiveMq, position-tracker, api-gateway, webapp.
 
Then we said, let's not lose past data when a pod is deleted, for this, let's install MongoDB and store the data not in the position tracker but there. For this, we needed to do two things: first, switch to release3 in the position-tracker microservice because here the feature of using Kubernetes DNS service for service discovery and finding the IP address of MongoDB was added. This way, it can discover and connect to MongoDB. The second thing was to create the MongoDB pod; we used the official MongoDB image. We decided to do everything related to Mongo in a separate YAML file and used mongo-stack.yml. This way, we saw that even if the position-tracker pod is deleted, we no longer lose the past because now the history is kept on the MongoDB pod.
 
This time, if the MongoDB pod is deleted, there was a possibility of losing the past.
To tackle this challenge, we made a development to ensure that MongoDB stores the data outside the container, for example, on our host. Our host = minikube (Hyper-V host) in this case. To achieve this, we used a persistent volume; we created the volume on our minikube host (name: mongo-persistent-storage; directory --> /mnt/some/directory/structure). We mounted the /data/db directory inside the container to the /mnt/some/directory/structure path on our minikube host (we used hostPath because it's for local development).
 
Now, even if we delete the MongoDB pod, the newly created pod does not lose the past data.
 
Then we said, let's improve the creation of this volume for possible future changes --> mongo-stack-PVC.yml and instead of hardcoding the hostPath there, let's use PersistentVolumeClaim, claimname=mongo-vpc, and create this volume using another independent script --> storage.yml.
mongo-stack.yaml is a specialist file for mongo database.
storage.yaml here is currently defining that we want to store the mongo data in a local directory on the host machine. 

 Here, we understood that PersistentVolume is physical storage, and PersistentVolumeClaim allocates from this created volume by Kubernetes. We mentioned that PersistentVolume could be EBS in AWS. With this storage script, we created local storage in our minikube --> path: "/mnt/some new/directory/structure" again, we used hostPath.
 
When we put this new db structure into operation, we lost the old data because this time we mounted the /data/db directory to a new directory on our minikube local host. But from that moment on, we saw that by deleting the MongoDB pod, past data is held thanks to the new mount.
![image](https://github.com/meneksece/vio-kubernetes-basics/assets/52826602/2050d848-8c70-490c-b1d3-c31732785fcf)
