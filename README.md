CI/CD SPRING BOOT PIPELINE PROJECT

In this project we will build spring boot hello world app, with Jenkins,
it deploy to k8s cluster which is based on oracle virtual platform.

When jenkins build triggered it will download source code from github
and build it with maven and create artifact which is a jar file, after
second phase kanik will build docker image according to Dockerfile, when
the build finishes third phase will begin taggin and pushing image to
DockerHub again with kaniko.

Last phase is kubernetes deploy, when build and push succesfuly
finished, Jenkins will change the image tag in deployment file which
located in github repo, then argocd will be triggered from that change
and will start to deployment.

Tech Stack

-   Vagrant

-   Oracle Virtual Box

-   Ansible

-   Docker

-   Kubernetes

-   Maven

-   Jenkins

-   Kaniko

-   ArgoCD

-   Git

-   Linux Ubuntu

-   Spring

-   Helm

**Phase 1 -- Setup Virtual Environment**

Virtual enviroment created with oracle virtual box via vagrant file.
According to vagrant file we will create below 4 virtual machines.

-   Control node: We will use it to control other machines via ansible
    and for k8s cluster via kubectl.

-   Master Node: Master role for k8s cluster

-   Worker Node: Worker1 role for k8s cluster

-   Worker Node: Worker2 role for k8s cluster

With Vagrant up command starting to build machines.

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image1.png){width="6.3in"
height="4.308333333333334in"}

Vagrant status command checkig status of machines, all seems ready.

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image2.png){width="6.3in"
height="1.8895833333333334in"}

Connecting to control node with command vagrant ssh controlz

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image3.png){width="6.3in"
height="4.267361111111111in"}

Connection to controlz node is succesfull, now we will start to
configure ssh keys for ansible to connect machines.

**Phase 2 -- Configure SSH keys**

Ssh-keygen command will create key.

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image4.png){width="6.03209208223972in"
height="3.5838331146106737in"}

Before ssh-copy-id command we are going to connect each node ( vagrant
ssh command ) and change **/etc/ssh/sshd_config** PasswordAuthentication
no to yes, because of ssh-copy-id command prevent public key error.

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image5.png){width="5.459095581802274in"
height="3.615087489063867in"}

Then we are restarting sshd with sudo systemctl restart sshd.service

Now we are starting to copy generated key to all nodes with ssh-copy-id
command

![metin, ekran görüntüsü, ekran içeren bir resim Açıklama otomatik
olarak oluşturuldu](./images/media/image6.png){width="6.3in"
height="1.3895833333333334in"}

Connect each node and press yes

**Phase 3 -- Configuring Ansible and Createing k8s cluster with
kubespray**

First we will clone kubespray repo and installing Python libraries.

git clone <https://github.com/kubernetes-sigs/kubespray.git>

cd kubespray ; sudo pip3 install -r requirements.txt

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image7.png){width="6.3in"
height="3.0131944444444443in"}

Creating ansible inventory file according to setup one master two worker
node.

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image8.png){width="6.3in"
height="3.207638888888889in"}

Check with ping inventory file is working.

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image9.png){width="6.3in"
height="3.36875in"}

Now its time to build our k8s cluster.

ansible-playbook -i inventory/my-cluster/hosts.ini cluster.yml \--become
\--become-user=root

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image10.png){width="6.3in"
height="4.847222222222222in"}

**Phase 4 -- Installing Kubectl and Helm to Controller Node**

We are installing kubectl according to instructions at
<https://kubernetes.io/docs/tasks/tools/>

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image11.png){width="6.3in" height="1.55in"}

When the installation is complete we check nodes with **kubectl get
pods** and we are getting error because of we dont have conf file of
cluster. Then we will connect master **ssh vagrant@node1** node and copy
/etc/kubernetes/admin.conf file to our contorller node path
\~/.kube/conf

![](./images/media/image12.png){width="6.637573272090989in"
height="2.511040026246719in"}

Now we check again **kubectl get nodes**

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image13.png){width="4.354774715660542in"
height="0.9897211286089239in"}

Kubectl get pods -A

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image14.png){width="6.3in"
height="2.588888888888889in"}

**Success!!**

We are installing helm according to instructions at
<https://helm.sh/docs/intro/install/>

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image15.png){width="6.3in"
height="3.3805555555555555in"}

We are checking helm installation with helm version

**Phase 4 -- Installing Jenkins with helm and installing kubernetes
plugin to Jenkins.**

helm repo add jenkins https://charts.jenkins.io

helm repo update

helm upgrade \--install myjenkins jenkins/Jenkins

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image16.png){width="6.3in"
height="3.0833333333333335in"}

Jenkins installation is complete now its time to get admin passowrd.

Kubectl exec --namespace default -it svc/myjenkins -c Jenkins \--
/bin/cat /run/secret/additional/chart-admin-password

![](./images/media/image17.png){width="6.3in"
height="0.2388888888888889in"}

When I check the Jenkins pod, relaised that it gives error because of
there is no persistenvolume, so I make hostpath volume for Jenkins with
below yaml.

Yaml for pv Jenkins

![](./images/media/image18.png){width="4.479792213473316in"
height="2.9900010936132984in"}

Kubectl get pods

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image19.png){width="5.354913604549432in"
height="0.8542858705161854in"}

Jenkins login

![](./images/media/image20.png){width="6.3in"
height="4.043055555555555in"}

Now we will install Jenkins kubernetes plugin

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image21.png){width="6.3in"
height="3.573611111111111in"}

configure Cloud Jenkins add as an agent.

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image22.png){width="6.3in"
height="2.3826388888888888in"}

We will use pod template to build and push our image

![](./images/media/image23.png){width="6.3in"
height="3.2708333333333335in"}

We have created a basic Freestyle Jenkins job to check everything is ok.

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image24.png){width="6.3in"
height="4.138194444444444in"}

Freestyle Jenkins job finishes succesfully

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image25.png){width="6.3in"
height="2.8291666666666666in"}

**Phase 4 -- Writing pipeline and checking kaniko as a kubernetes pod**

Firs we will create docker secret at kubernetes to pull our images from
docker hub.

kubectl create secret docker-registry dockercred \\
\--docker-server=https://index.docker.io/v1/ \\
\--docker-username=**kocagoz** \\ \--docker-password=
dckr_pat\_\*\*\*\*\*\* \\ <--docker-email=kocagoz@gmail.com>

**create secret**

![](./images/media/image26.png){width="6.3in" height="0.7875in"}

Kaniko-pod create

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image27.png){width="6.157109580052493in"
height="4.615227471566055in"}

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image28.png){width="6.3in"
height="1.2173611111111111in"}

As a result kaniko succesfully send image to dockerhub

![](./images/media/image29.png){width="6.3in"
height="0.8006944444444445in"}

Now we will write our pipelines but first we have create credentianls
for github.

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image30.png){width="6.3in"
height="5.679861111111111in"}

Not its time to write Jenkins pipeline

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image31.png){width="6.3in"
height="4.024305555555555in"}

When we run pipeline job we get succses message and image pushed to
dockerhub.

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image32.png){width="5.9695833333333335in"
height="3.135854111986002in"}

**Phase 5 -- Installing argocd and writing deployment yaml files.**

First we will install argocd

kubectl create namespace argocd

kubectl apply -n argocd -f
<https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml>

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image33.png){width="6.3in"
height="1.4354166666666666in"}

Argocd installation is complete now we will gather admin password

kubectl -n argocd get secret argocd-initial-admin-secret -o
jsonpath=\"{.data.password}\" \| base64 -d; echo

![](./images/media/image34.png){width="6.3in"
height="0.49930555555555556in"}

We will enter argo and create new project

![](./images/media/image35.png){width="6.3in"
height="3.0236111111111112in"}

Project details.

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image36.png){width="6.3in"
height="5.302777777777778in"}

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image37.png){width="6.3in"
height="1.882638888888889in"}git token
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

sudo chown -R 1000:1000 /opt/volume/

![](./images/media/image38.png){width="6.3in"
height="2.222916666666667in"}