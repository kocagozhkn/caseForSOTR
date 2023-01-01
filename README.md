
<p align="center">
<b>CI/CD SPRING BOOT PIPELINE PROJECT</b>
</p>
<p align="center">
<b>ENVIRONMENT</b>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/kocagozhkn/caseForSOTR/main/images/media/image01.png" width=60% height=60%>

</p>

<p align="center">
<b>PIPELINE</b>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/kocagozhkn/caseForSOTR/main/images/media/image001.png" width=60% height=60%>

</p>

In this project we will build spring boot hello world app, with Jenkins,
it deploy to k8s cluster with argoCD which is deployed on oracle virtual platform.

**stage-1**
When jenkins build triggered it will download source code from github
and build it with maven and create artifact which is a jar file, 

**stage-2**
Second stage kaniko will build docker image according to Dockerfile, and when
the build finishes kaniko will begin tagging and pushing image to
DockerHub.

**stage-3**
Last stage is kubernetes deploy, when build and push succesfuly
finished, Jenkins will change the image tag in deployment file which
located in github repo, then argocd will be triggered from that change
and will start to deployment new version of deployment. We can check blue-green deployment and promote to new version if everything is ok.

Tech Stack
<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/8/87/Vagrant.png" width=50>
  <img src="https://upload.wikimedia.org/wikipedia/commons/d/d5/Virtualbox_logo.png" width=60>
  <img src="https://avatars.githubusercontent.com/u/1507452?s=200&v=4" width=60>
  <img src="https://avatars.githubusercontent.com/u/5429470?s=280&v=4" width=80>
   <img src="https://static-00.iconduck.com/assets.00/kubernetes-color-icon-256x256-t8ualzkj.png" width=50>
     <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/e/e9/Jenkins_logo.svg/1200px-Jenkins_logo.svg.png" width=40>

</p>


<p align="center">
 

<img src="https://cncf-branding.netlify.app/img/projects/argo/icon/color/argo-icon-color.png" width=70>
          <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/a/ab/Logo-ubuntu_cof-orange-hex.svg/1200px-Logo-ubuntu_cof-orange-hex.svg.png" width=70>
      <img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQskALxpNXUNt7naftdZ77u66lQ1oQft4nwt3jpd2U&s" width=70>
          <img src="https://git-scm.com/images/logos/logomark-black@2x.png" width=80>
      <img src="https://i.pinimg.com/736x/27/42/47/274247632a09f3a7750c2c6f43de403a.jpg" width=70>

     
</p>
<p align="center">
 
  <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/5/52/Apache_Maven_logo.svg/2560px-Apache_Maven_logo.svg.png" width=80>

<img src="https://miro.medium.com/max/1400/0*u3F5ndHj_h8SuKVe.png" width=80>
  
     
</p>

**Phase 1 -- Setup Virtual Environment**

Virtual enviroment created with oracle virtual box via vagrant file.
According to vagrant file we will create below 4 virtual machines.

-   Control node: We will use it to control other machines via ansible
    and for k8s cluster via kubectl.

-   Master Node: Master role for k8s cluster

-   Worker Node: Worker1 role for k8s cluster

-   Worker Node: Worker2 role for k8s cluster

With ```vagrant up``` command starting to build machines.

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image1.png)

```vagrant status``` command checkig status of machines, all seems ready.

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image2.png)

Connecting to control node with command ``vagrant ssh controlz``

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image3.png)

Connection to controlz node is succesfull, now we will start to
configure ssh keys for ansible to connect machines.

**Phase 2 -- Configure SSH keys**

``ssh-keygen`` command will create key.

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image4.png)

Before ``ssh-copy-id`` command we are going to connect each node ( vagrant
ssh command ) and change **/etc/ssh/sshd_config** ``PasswordAuthentication``
``no`` to ``yes``, because of prevent public key error due to``ssh-copy-id`` command.

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image5.png)

Then we are restarting **sshd service** with ``sudo systemctl restart sshd.service``

Now we are starting to copy generated key to all nodes with ``ssh-copy-id``
command

![metin, ekran görüntüsü, ekran içeren bir resim Açıklama otomatik
olarak oluşturuldu](./images/media/image6.png)

Connect each node and press yes to add shh key.

**Phase 3 -- Configuring Ansible and Createing k8s cluster with
kubespray**

First we will clone kubespray repo and installing Python libraries.

``git clone <https://github.com/kubernetes-sigs/kubespray.git>``

``cd kubespray ; sudo pip3 install -r requirements.txt``

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image7.png)

Creating ansible inventory file according to setup one master two worker
node.

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image8.png)

To check inventory file is correct we will use simple ansible ping command.
``ansible -i inventory/my-cluster/hosts.ini -m ping all``

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image9.png)

Now its time to build our k8s cluster.

``ansible-playbook -i inventory/my-cluster/hosts.ini cluster.yml --become
--become-user=root``

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image10.png)

**Phase 4 -- Installing Kubectl and Helm to Controller Node**

We are installing kubectl according to instructions at

<https://kubernetes.io/docs/tasks/tools/>

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image11.png)

When the installation is complete we check nodes with ``kubectl get
pods`` and we are getting error because of we dont have conf file of
cluster. Then we will connect master ``ssh vagrant@node1`` node and copy
**/etc/kubernetes/admin.conf** file to our contorller node path
**\~/.kube/conf**

![](./images/media/image12.png)

Now we check again ``kubectl get nodes``

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image13.png)
``kubectl get pods -A``

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image14.png)

**Success!!**

Now its time to install **Helm**. We are installing **helm** according to instructions at
<https://helm.sh/docs/intro/install/>

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image15.png)

We are checking helm installation with ``helm version``

**Phase 4 -- Installing Jenkins with helm and installing kubernetes
plugin to Jenkins.**

``helm repo add jenkins https://charts.jenkins.io``

``helm repo update``

``helm upgrade \--install myjenkins jenkins/Jenkins``

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image16.png)

**Jenkins** installation is complete now its time to get **admin passowrd**.

``kubectl exec --namespace default -it svc/myjenkins -c Jenkins \--
/bin/cat /run/secret/additional/chart-admin-password``

![](./images/media/image17.png)

When I check the **Jenkins** pod, relaised that it gives error because of
there is no persistentVolume, so I created hostpath volume for **Jenkins** with
below yaml.

Yaml for pv Jenkins

![](./images/media/image18.png)
``kubectl get pods``

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image19.png)

Jenkins login

![](./images/media/image20.png)

Now we will install Jenkins **kubernetes** plugin

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image21.png)

Configuring Cloud and adding kubernetes.

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image22.png)
We will use **pod template** to build and push our image

![](./images/media/image23.png)

We have created a basic **Freestyle Jenkins job** to check everything is ok.

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image24.png)

Freestyle Jenkins job finishes succesfully

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image25.png)

**Phase 4 -- Writing pipeline and checking kaniko as a kubernetes pod**

Firs we will create docker secret at kubernetes to pull our images from
docker hub.

``kubectl create secret docker-registry dockercred 
--docker-server=https://index.docker.io/v1/ 
--docker-username=**kocagoz**  --docker-password=
dckr_pat****** --docker-email=kocagoz@gmail.com>``

**create secret**

![](./images/media/image26.png)

We will check is **Kaniko** working so, I created a simple Kaniko Pod, created pod with ``kubectl apply -f kaniko.yaml ``

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image27.png)

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image28.png)

As a result kaniko **succesfully** push image to dockerhub.

![](./images/media/image29.png)

Now we will write our *pipeline*, but first we have to create *credentianls*
for github.

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image30.png)

Not its time to write Jenkins **pipeline**

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image31.png)

When we run pipeline job we get **succses* message and image pushed to
**dockerhub** and k8s manifest changed at **github**.

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image32.png)

**Phase 5 -- Installing argocd and writing deployment yaml files.**

First we will install argocd

``kubectl create namespace argocd``

``kubectl apply -n argocd -f
<https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml>``

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image33.png)

Argocd installation is complete now we will gather **admin** password

``kubectl -n argocd get secret argocd-initial-admin-secret -o
jsonpath=\"{.data.password}\" \| base64 -d; echo``

![](./images/media/image34.png)

We need to create argo project for our deployment.

``kubectll apply -f application.yaml``

![](./images/media/image39.png)


We will enter argo and create new project

![](./images/media/image35.png)

Lets check our app.

![](./images/media/image40.png)

Project details.

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image36.png)

Now we can install argo cd roolout plugin accrding to link ``https://argoproj.github.io/argo-rollouts/installation/``

check argocd rollout cli is working.
`` kubectl argo rollouts version``

```kubectl-argo-rollouts: v1.3.2+f780534```


cli is working, check rollout status current app.
``kubectl argo rollouts get rollout case-deployment``

![](./images/media/image41.png)



![](./images/media/image38.png)


Now its time to check our spring app from browser, we used **nodeport 300055**, 

![metin içeren bir resim Açıklama otomatik olarak
oluşturuldu](./images/media/image37.png)

To make new **roolout** we will start our **jenkins job** and it will change **image tag** when the pipeline is succes, then argocd will be triggered and it will create *blue version* of app.

![](./images/media/image42.png)

**blue version** of our app is deployed. Lets check is it working from **nodeport 30056**.

![](./images/media/image43.png)

:thumbsup: blue version is working, now promote new version via. 

``kubectl argo rollouts promote case-deployment``

``rollout 'case-deployment' promoted``

![](./images/media/image44.png)

We have prometed to **image version 20**

thanks your for reading. :thumbsup:





