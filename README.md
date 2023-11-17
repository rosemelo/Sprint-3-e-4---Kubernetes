# Desafio 1
Fazendo o restore das VMs e instalando o minikube ou S3 para utilizar o kubernetes
## Aqui eu vou mostrar a instalação do MicroK8s



# Desafio 2
Deploy k8s cluster com 1 master e 1 node (ou 2 masters) no Oracle Linux configurando o ingress e fazendo o deploy do jenkins. 

´E breve estarei completando o passo a passo´
 
```bash
# Desabilite o swap
sudo swapoff -a

# Instale as dependências
sudo yum install -y kubelet kubeadm kubectl

# Inicie e habilite o serviço kubelet
sudo systemctl enable --now kubelet

**Passo 4: Configuração do Cluster Kubernetes**

No nó mestre, inicialize o cluster Kubernetes usando o kubeadm. Substitua `<SEU_IP>` pelo endereço IP do nó mestre.
----------(Eu não executei esse, deu erro)--------------
```bash
sudo kubeadm init --apiserver-advertise-address=192.168.0.40--pod-network-cidr=10.244.0.0/16
```

**Passo 5: Configuração do Nó de Trabalho (Node)**

No nó de trabalho, execute o comando `kubeadm join` que você obteve no final do passo 4.

**Passo 6: Configuração do Ingress Controller**

Para configurar o ingress controller, você pode usar o Nginx Ingress Controller. Siga as instruções do GitHub para instalar o controlador de ingresso Nginx: https://kubernetes.github.io/ingress-nginx/deploy/

**Passo 7: Implantação do Jenkins**

Agora que você tem seu cluster Kubernetes configurado e o ingress controller funcionando, você pode implantar o Jenkins usando um arquivo de manifesto YAML. Crie um arquivo chamado `jenkins.yaml` com o seguinte conteúdo:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      containers:
      - name: jenkins
        image: jenkins/jenkins:lts
        ports:
        - containerPort: 8080
        - containerPort: 50000
---
apiVersion: v1
kind: Service
metadata:
  name: jenkins
spec:
  selector:
    app: jenkins
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: NodePort
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: jenkins-ingress
spec:
  rules:
  - host: jenkins.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: jenkins
            port:
              number: 80
```

Este arquivo de manifesto cria um Deployment para o Jenkins, um serviço NodePort e um ingress para rotear o tráfego para o Jenkins. Certifique-se de substituir `jenkins.example.com` pelo seu domínio ou endereço IP.

Para implantar o Jenkins, use o comando:

```bash
kubectl apply -f jenkins.yaml
```