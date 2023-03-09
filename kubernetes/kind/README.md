# Criando um Cluster Kubernetes Local usando o Kind

Criar um cluster Kubernetes local usando o Kind (Kubernetes IN Docker). O Kind é uma ferramenta que cria clusters Kubernetes em Docker containers. Ele é útil para testar, desenvolver e depurar aplicações Kubernetes localmente.

Não é em todas as empresas que temos a possibilidade (ou permissao) de instalar aplicativos na nossa estação de trabalho ou mesmo em um servidor, mesmo que de desenvolvimento. Portanto, se apenas tivermos o docker instalado, e o serviço no rodando, podemos facilmente criar um cluster kubernetes para uso como testes de nossos códigos, diminuindo a possibilidade de impactar o trabalho do resto da equipe.

## Passo 1: Instale o Docker e o Kind

Para começar, você precisa instalar o Docker e o Kind. Siga as instruções na documentação oficial para instalar o Docker e o Kind na sua plataforma.

- [Instalando o Docker](https://docs.docker.com/engine/install/)
- [Instalando o Kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)

## Passo 2: Crie um cluster Kubernetes

Agora que você tem o Docker e o Kind instalados, pode criar um cluster Kubernetes local. Para fazer isso, execute o seguinte comando no seu terminal:
```
kind create cluster --name redis-cluster1 --config kind-cluster-config.yaml
```
Este comando criará um cluster Kubernetes local com um nó controller e um nó worker. O Kind criará Docker containers para cada nó. Como resultado da execuçao do comando, o resultado sera semelhante a esse:

```
Creating cluster "redis-cluster1" ...
 ✓ Ensuring node image (kindest/node:v1.26.0) 🖼
 ✓ Preparing nodes 📦 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-redis-cluster1"
You can now use your cluster with:

kubectl cluster-info --context kind-redis-cluster1

Have a nice day! 👋
```
## Passo 3: Verifique se o cluster foi criado corretamente

Para verificar se o cluster foi criado corretamente, execute o seguinte comando:
```
kubectl cluster-info
```

Este comando deve retornar informações sobre o cluster, como o servidor API Kubernetes e o endpoint do Kubernetes.

```
Kubernetes control plane is running at https://127.0.0.1:50311
CoreDNS is running at https://127.0.0.1:50311/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

## Passo 4: Interaja com o cluster Kubernetes

Agora que o cluster Kubernetes local foi criado, você pode interagir com ele usando o kubectl, a ferramenta de linha de comando Kubernetes. Por exemplo, você pode executar o seguinte comando para ver todos os pods no cluster:
```
kubectl get pods --all-namespaces
```

Este comando deve retornar uma lista de todos os pods no cluster, semelhante a este: 

```
kubectl get pods --all-namespaces
NAMESPACE            NAME                                                   READY   STATUS    RESTARTS   AGE
kube-system          coredns-787d4945fb-c8pvv                               1/1     Running   0          4m14s
kube-system          coredns-787d4945fb-fnlzw                               1/1     Running   0          4m14s
kube-system          etcd-redis-cluster1-control-plane                      1/1     Running   0          4m28s
kube-system          kindnet-7tgk4                                          1/1     Running   0          4m11s
kube-system          kindnet-dgvhl                                          1/1     Running   0          4m14s
kube-system          kube-apiserver-redis-cluster1-control-plane            1/1     Running   0          4m27s
kube-system          kube-controller-manager-redis-cluster1-control-plane   1/1     Running   0          4m27s
kube-system          kube-proxy-hjlmt                                       1/1     Running   0          4m14s
kube-system          kube-proxy-vsqt5                                       1/1     Running   0          4m11s
kube-system          kube-scheduler-redis-cluster1-control-plane            1/1     Running   0          4m27s
local-path-storage   local-path-provisioner-c8855d4bb-m8sc8                 1/1     Running   0          4m14s
```

## Passo 5: Limpe o cluster

Quando você terminar de usar o cluster, poderá excluí-lo executando o seguinte comando:

```
kind delete cluster --name redis-cluster1
```

Este comando excluirá todos os containers Docker criados pelo Kind para o cluster.

## Conclusão

Neste tutorial, você aprendeu como criar um cluster Kubernetes local usando o Kind. Você também aprendeu como verificar se o cluster foi criado corretamente e como interagir com ele usando o kubectl. O Kind é uma ferramenta poderosa para testar, desenvolver e depurar aplicações Kubernetes localmente.
