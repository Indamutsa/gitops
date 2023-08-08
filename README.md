# Let us create a devops full with Jenkins and argo cd

Let us spin up an image where we can work
`ec2 run-instances --image-id ami-0557a15b87f6559cf --count 1 --instance-type t2.micro --key-name key-main --security-group-ids sg-0d718080abf254220 --subnet-id subnet-0b78dd8856d90b22c`

Let us enable direct loggin through ssh keys

To avoid logging in remote server with password or access key like 👍
`ssh -i key-main.pem ubuntu@54.225.6.176`

We proceed with the steps below: \

Let us first establish a secure with an ssh key \
_run_: 
`ssh-keygen -t rsa -b 4096` \
The command uses rsa to generate a 4096-bit key pair. The key pair is stored in the ~/.ssh directory. The private key is named id_rsa and the public key is named id_rsa.pub.

To copy the public key to the remote host \
_run_: \
`cat ~/.ssh/id_rsa.pub | ssh -i key-main.pem ubuntu@54.225.6.176 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"`


------------------------------

Once inside the newly created virtual machine,
let us install docker, jenkins, kind and kubectl

------
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


To install docker, run these commands:
```
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```

You may need to use docker woithout sudo \
`sudo chmod 777 /var/run/docker.sock`

-------
Install jenkins
`docker run -p 8080:8080 -d --name jenkins jenkins/jenkins`

To access it, go to the dashboard, and change the security group to accept 8080, in my case i enabled all tcp (not recommended, just for test)


Access the password for Jenkins: \
`docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword`

Access it in the browser and set it up!

-------

Install kind

```
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.17.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

````


---- 

Install kubectl

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin
```


In case you would like to install a loadbalancer, you can do so by running the following command:
Apply the LoadBalancer service deploy from the kube-service-load-balancer file:

debian@busybox:~$ `kubectl apply -f https://raw.githubusercontent.com/mvallim/kubernetes-under-the-hood/master/services/kube-service-load-balancer.yaml`
The response should look similar to this:

```
service/load-balancer-service created
Query the state of service load-balancer-service
```

debian@busybox:~$ `kubectl get service load-balancer-service -o wide`
The response should look similar to this:

```
NAME                    TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE   SELECTOR
load-balancer-service   LoadBalancer   10.103.128.109   <pending>     80:31969/TCP   7s    app=guestbook,tier=frontend
If you look at the status on the EXTERNAL-IP it is <pending> because we need configure MetalLB to provide IP to LoadBalancer service.
```

Deploy
To install MetalLB, apply the manifest:

Apply the MetalLB manifest namespace from the namespace.yaml file:

debian@busybox:~$ `kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/namespace.yaml`
The response should look similar to this:

namespace/metallb-system created
Apply the MetalLB manifest controller and speaker from the metallb.yaml file:

debian@busybox:~$ `kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/metallb.yaml`

```
The response should look similar to this:

podsecuritypolicy.policy/controller created
podsecuritypolicy.policy/speaker created
serviceaccount/controller created
serviceaccount/speaker created
clusterrole.rbac.authorization.k8s.io/metallb-system:controller created
clusterrole.rbac.authorization.k8s.io/metallb-system:speaker created
role.rbac.authorization.k8s.io/config-watcher created
role.rbac.authorization.k8s.io/pod-lister created
role.rbac.authorization.k8s.io/controller created
clusterrolebinding.rbac.authorization.k8s.io/metallb-system:controller created
clusterrolebinding.rbac.authorization.k8s.io/metallb-system:speaker created
rolebinding.rbac.authorization.k8s.io/config-watcher created
rolebinding.rbac.authorization.k8s.io/pod-lister created
rolebinding.rbac.authorization.k8s.io/controller created
daemonset.apps/speaker created
deployment.apps/controller created
Query the state of deploy
```

debian@busybox:~$ `kubectl get deploy -n metallb-system -o wide`
The response should look similar to this: \


```
NAME         READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                               SELECTOR
controller   0/1     1            0           11s   controller   quay.io/metallb/controller:v0.12.1   app=metallb,component=controller
This will deploy MetalLB to your cluster, under the metallb-system namespace. The components in the manifest are:
```

The metallb-system/controller deployment. This is the cluster-wide controller that handles IP address assignments.
The metallb-system/speaker daemonset. This is the component that speaks the protocol(s) of your choice to make the services reachable.
Service accounts for the controller and speaker, along with the RBAC permissions that the components need to function.
The installation manifest does not include a configuration file. MetalLB’s components will still start, but will remain idle until you define and deploy a configmap. The memberlist secret contains the secretkey to encrypt the communication between speakers for the fast dead node detection.
```
Reference : https://metallb.universe.tf/installation/#installation-by-manifest

Configure
Based on the planed network configuration (here) we will have a metallb-config.yaml as below:

apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.2.2-192.168.2.125
```

Apply the MetalLB configmap from the metallb-config.yaml file: \

debian@busybox:~$ `kubectl apply -f https://raw.githubusercontent.com/mvallim/kubernetes-under-the-hood/master/metallb/metallb-config.yaml`
Query the state of deploy

debian@busybox:~$ `kubectl get deploy controller -n metallb-system -o wide`
The response should look similar to this:

NAME         READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                               SELECTOR
controller   1/1     1            1           88s   controller   quay.io/metallb/controller:v0.12.1   app=metallb,component=controller
Query the state of service load-balancer-service

debian@busybox:~$ `kubectl get service load-balancer-service -o wide`
The response should look similar to this:
```
NAME                    TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE     SELECTOR
load-balancer-service   LoadBalancer   10.103.128.109   192.168.2.2   80:31969/TCP   4m51s   app=guestbook,tier=frontend
```

----------


```
===> We need to setup jenkins to be able to execute docker commands
docker run --rm -d -u root -p  8080:8080 --name jenkins  -v jenkins-data:/var/jenkins_home -v $(which docker):/usr/bin/docker -v /var/run/docker.sock:/var/run/docker.sock -v "$HOME":/home jenkins/jenkins:lts
```

```
===> That's the initial password
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword    # This is the initial password
```

Once we have jenkins installed, 
`Add the credentials from `manage credentials` of the github account and docker account.`


We get the token from personal token access which is in the settings, developer settings. 

When generating the token, we enabled the first which is related to the repository.

The pipeline in jenkins is created by clicking on new item, give the name and choose pipeline.
Add the description and enable `GitHub hook trigger for GITScm polling`

Then configure the repository:
```
Head to the pipeline, pipeline syntax at the bottom and then configure git.
- Enter the repository url
- branch name
- choose the credentials created
- generate pipeline script
- copy the generated pipeline script into the pipeline (jenkins file)
```


We are run the pipeline from vscode:
copy this: 
```
{
    "jenkins-runner.jobs": {
        "pipeline-name": {
            "runWith": "jenkins-master",
            "name": "gitops",
            "isDefault": true
        }
    },
    "jenkins-runner.hostConfigs": {
        "jenkins-master": {
            "url": "http://localhost:8080",
            "user": "****",
            "password": "****",
            "useCrumbIssuer": true

        }
    }
}
```

in .vscode/settings.json

Create a jenkinsfile, and then enter some tests:
```
pipeline {
    agent any
    stages {
        stage('Print build Number') {
            steps {
                sh "echo ${BUILD_NUMBER}"
            }
        }
    }
}
```

===========

Install argocd

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```


Let us get a workspace container:
`docker run -it --rm -v ${HOME}:/root/ -v ${PWD}:/work -w /work --net host ubuntu bash`

Let us # install curl & kubectl 
```
apt update && apt upgrade -y && apt install curl wget nano -y
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x ./kubectl
mv ./kubectl /usr/local/bin/kubectl
```

Let us install argocd cli: 

```
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```

Access The Argo CD API Server 
```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

The username is : admin \
The password can be found: 
`
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode && echo
`


You can Login Using The CLI \
`argocd login <ARGOCD_SERVER> ---> argocd login localhost:8020`

change the password \
`argocd account update-password`


----------------------------------------------------------------

Create withcredentials in jenkins for git to update github after we have pushed the code to git in order to trigger argocd.

Go to pipeline syntax, and select `withCredentials: Bind credentials to variables \
Choose username and password separated in the bindig. 

And then generate the script. Follow the instructions

----------------------------------------------------------------

We then created the webhook on github. \
The url is obtained ngrok: `ngrok http 8080`

Change something in the code base, and then push. The webhook will be triggered on push
