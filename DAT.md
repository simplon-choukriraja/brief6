 ## ***DAT - Brief 6 : Kubernetes Raja***
 
 ## **Contexte du projet**
 
**Contexte du projet** : déployer l’application ***Azure Voting App*** et sa base de données ***Redis*** sur le cluster ***Kubernetes*** de Azure et ***AKS***.
 
 
## **Besoins fonctionnels**

Nous utilisons **Kubernetes** et **Aks** pour développer **l'application vote** sur une base de données Redis, l'application de vote consiste à permettre à l'utilisateur de pouvoir choisir entre chien et chat ou de réinitialiser le compteur.L'utilisateur pourra accéder à l'application vote via **DNS**. 

 
* **Résultat**

<img width="850" alt="Résultat" src="https://user-images.githubusercontent.com/108053084/196186146-88655fe7-f535-4cfc-8e0e-b22ad1c7dc34.PNG">


## **La représentation opérationnelle** 

La base de données Redis a été déployée via un script dans lequel a été ajouté un **PersistentVolumeClaim** utilisant également une **StorageClass**. Dans mon cas, j'ai utilisé le disque, car les données y seront stockées et enregistrées. Seules les personnes ayant des droits pourront accéder à ces données. **L'utilisateur** n'a pas la possibilité d'accéder à ces données, mais uniquement à **l'application de vote**.

## **La représentation fonctionnelle**

L'application permet à l'utilisateur de pouvoir choisir entre chien ou chat. Afin d'accéder à l'application, il est nécessaire d'aller sur le **web** (internet), nous avons créé un certificat TLS pour sécuriser le site et l'utilisateur peut ensuite accéder au l'application de vote via le **DNS** suivant : **https://brief6.simplon-raja.space/.**
Pour éviter une surcharge, nous avons créé un **test de charge** pour s'assurer que lorsque vous atteignez 70% de CPU ou dépassez ce pourcentage, une vm (machine virtuelle) se crée automatiquement pour éviter les problèmes de surcharge.


## **Topologie**
 
 ```mermaid 

graph TD
    A[Client] --> B[Fournisseur] 
    B[Fournisseur] -->|DNS| C(Web)
    C(Web) --> O{APP.VOTE}
    C(Web) --> | Microsoft Azure | D[App Gateway]
    R(Disk) --> | PVC | P(REDIS)
    D[App Gateway] --> H(NODES) 
    subgraph Cluster
    H -->|Un| I[Node]
    H -->|Deux| L[Node]
    H -->|Trois| M[Node]
    H -->|Quattre| N[Node]    
    I[node]--> |HorizontalPodAutoscaler| O{APP.VOTE}
    N[node]--> P{REDIS}
   
end 
```

## **La représentation technique**


* - Installer Kubernetes Servers : 
 
        * sudo apt update 
        * sudo apt -y upgrade 
        * sudo apt -y install curl apt-transport-https
        * curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add - 
        * echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list 
        * sudo apt update
        * sudo apt -y install vim git curl wget kubelet kubeadm kubectl
        * sudo apt-mark hold kubelet kubeadm kubectl


* Commande  pour lancé le deployement sur kubernetes: 
    
        *kubectl apply -f 
        
* Créer une groupes de ressources  : 
 
         *  az group create --location eastus --resource-group brief6-raja

* créer un cluster AKS avec l’add-on AGIC et 4 nodes***
        
         *  az aks create -g brief6-raja -n agic --ssh-key-value /Users/rajac/.ssh/id_rsa.pub --node-count 4 --enable-managed-identity -a ingress-appgw --appgw-name myApplicationGateway --appgw-subnet-cidr "10.225.0.0/16"
         
         *  az aks get-credentials --name agic --resource-group brief6-raja  


* Deployment Redis : 

    * Nous avons initialement créé un service de type Loadbalancer pour la base de données Redis. Plus tard, nous avons changé le service en ClusterIP. Le conteneur utilise un port 6379. 
    * Attribution d'un mot de passe à l'aide de Kubernetes Secret. 
    * Pour la base de données, nous avons créé un stockage de type PersistentVolumeClaim (pvc) avec un disque déployé par StorageClass. 
    * PersistentVolumeClaim vous permet de récupérer des données même si la base de données Redis est supprimée.
    * Nous avons deployé l'Application Vote.
    * L'image du conteneur pour l'application de vote est la suivante : **whujin11e/public:azure_voting_app**. 
    * Nous avons initialement créé un service de type Loadbalancer pour l'Application Vote. Plus tard, nous avons changé le service en ClusterIP. Le conteneur utilise un port 80. 
    * Nous avons créé un Ingress pour permettre à l'application Gateway de communiquer avec le certificat TLS et avec le DNS.
    * Nous avons créé un DNS via Gandi et sécurisé via HTTPS avec le port 443. 


# **Choix de l’architecture**

* Le fait de créer un PersistentVolumeClaim est avantageux car si nous avons des problèmes dans la base de données Redis et que nous somme obligés de la remplacer, nous ne perdrons pas nos données car elles seront enregistrées et archivées sur le disque.
* Le fait de mettre un test de charge permet, lorsque vous arrivez à surcharger le processeur en dépassant 70% du CPU, la création automatiquement et de manière autonome d'un maximum de 8 VMs (Virtuelle Machine). Cela évite une surcharge qui pourrait entraîner certains problèmes ou bugs.
