# kubernetes-hands-on

This set of hands-on covers fundamentals of Kubernetes.

It will take you through all required basics to get started with Kubernetes. By the end of this hands-on, you should able to deploy demo application.

1. [**Kubectl** - Kubernetes Command line tool](##kubectl)
1. [**Pod** - Basic building block](#pod)
1. [**ReplicaSet**](#ReplicaSet)

## kubectl

`kubectl` is a Kubernetes CLI client. You will use it to create, delete and inspect various Kubernetes objects.

### **Installation**

#### For MacOS

```sh
brew install kubernetes-cli
```

Others can follow instructions from [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

Try this:

```sh  
kubectl version

# Print the client and server versions for the current context
```

#### Kubernetes Config

There is a kubeconfig file behind every working kubectl command. Default location for kubeconfig is: `$HOME/.kube/config`.

You can specify other kubeconfig files by setting the `KUBECONFIG` environment variable or by setting the `--kubeconfig` flag.

`kubectl` follows below preference for the kubeconfig:

- --kubeconfig flag, if specified  
- KUBECONFIG environment variable, if specified
- $HOME/.kube/config file

```sh
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: <CA-DATA>
    server: https://<APISERVER-HOST>:<APISERVER-PORT>
  name: <CLUSTER-NAME>
contexts:
- context:
    cluster: <CLUSTER-NAME>
    user: <USER>
  name: <USER>@<CLUSTER-NAME>
kind: Config
users:
- name: <USER>
  user:
    client-certificate-data: <CLIENT-CRT-DATA>
    client-key-data: <CLIENT-KEY-DATA>
    token: <BEARER TOKEN>
```

You can use [sample](https://raw.githubusercontent.com/C123R/kubernetes-hands-on/master/kubeconfig/sample-kubeconfig.yaml) kubeconfig for playing around with `kubectl config` command.

```sh
curl https://raw.githubusercontent.com/C123R/kubernetes-hands-on/master/kubeconfig/sample-kubeconfig.yaml > ~/.kube/config
```

### **Useful kubectl commands - You need to know**

- **Managing Config/Context/Cluster**

    ```sh
    kubectl config view
    kubectl config use-context
    kubectl config get-contexts
    ```

    ```sh
    kubectl cluster-info
    ```

- **Viewing Kubernetes Resources**

    The most basic command for viewing Kubernetes objects via kubectl is **get**.  

    `kubectl get <resource-name> <object-name>`

    For example:

    ```sh
    kubectl get pods nginx
    ```

    If you need more detailed information about a particular object, you can use **describe**.

    `kubectl describe <resource-name> <object-name>`

    For example:

    ```sh
    kubectl describe pods nginx
    ```

    You can use different output format `-o,--output=''` in kubectl.

    ```sh
    kubectl describe pods nginx -o yaml
    ```

- **Creating, Updating, and Destroying Kubernetes Resources**

    `kubectl apply` is the the recommended way of managing Kubernetes applications.  
    For example:

     ```sh
    kubectl apply -f examples/pods/pod.yaml

    # Here you dont need to specify resource type, as its part of the yaml.
    ```

    You can still make changes to this `examples/pods/pod.yaml` and apply it again to update the object, or you can use `edit` command to make changes interactively.

    `kubectl edit <resource-name> <obj-name>`

    For example:

    ```sh
    kubectl edit pod nginx
    ```

    (Imperative way)
    You could use `create` to make a namespace,secret,deployment etc. For instance:

    ```sh
    kubectl create production
    # create namespace

    kubectl create deployment nginx --image=nginx
    # start a single instance of nginx
    ```

    When you want to delete an object, you can simply run:

    `kubectl delete <resource-name> <obj-name>`

    and if object was created using apply:

    For example:

    ```sh
    kubectl delete -f examples/pods/pod.yaml
    ```

- **Troubleshoot and Debugging Commands**

    To get the logs for a container in a pod

    `kubectl logs <resource-name> <obj-name>`

    For example:

    ```sh
    kubectl logs nginx
    ```

    You can also execute command into container

    `kubectl exec -it <pod-name> -- bash`

    For example:

    ```sh
    kubectl exec -it nginx -- bash
    kubectl exec -it nginx -- date
    ```

    There is way to create a proxy between localhost and Kubernetes API Server.

     ```sh
    kubectl proxy
    ```

    So you can practically access all kubernetes internal services from localhost now. For instance, you can access Kubernetes Dashboard:

    http://localhost:8001/api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy/#!/overview?namespace=default

## **Basic Kubernetes Objects**

### **Namespace**

```sh
kubectl get namespaces

kubectl create namespace $(whoami)
```

### **POD**

A Pod is the basic building block of K8s Objects.

```sh
apiVersion: v1  
kind: Pod           # type of k8s object
metadata:
  name: nginx
  labels:
    env: demo
# spec consists of the core information about pod
spec:
  containers:
  - name: nginx
    image: nginx
  ports:
    - name: http
      containerPort: 80
```

Lets create a pod running nginx container:

```sh
kubectl apply -f https://raw.githubusercontent.com/C123R/kubernetes-hands-on/master/examples/pods/pod.yaml -n $(whoami)
```

```sh
kubectl get pods --watch

kubectl get pods -n $(whoami)
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          31s
```

You can get the complete pod manifest with output option -o yaml

```sh
kubectl get pods -n $(whoami) -o yaml
```

Now view the details:

```sh
kubectl describe pod nginx -n $(whoami)
```

To check the logs:

```sh
kubectl logs nginx -n $(whoami)
```

Now you can access nginx pod with `port-forward`:

```sh
kubectl port-forward nginx 8080:80 -n $(whoami)
```

Lets read nginx.conf file from our nginx container:

```sh
kubectl exec -it nginx -n $(whoami) -- cat /etc/nginx/nginx.conf
```

**But as you know, Pod is a smallest deployable unit and not a single container**, you can run multiple containers in one POD, which will share the network namespace(IP and Port space etc.) and volumes.

For example:

```sh
apiVersion: v1
kind: Pod
metadata:
  name: nginx-mc
  labels:
    env: test
spec:
  containers:
  - name: nginx             #1 Container running nginx
    image: nginx
    ports:
    - name: http
      containerPort: 80
    volumeMounts:
    - name: content
      mountPath: /usr/share/nginx/html

  - name: content-writer    #2 Container updating nginx default html page.
    image: debian
    volumeMounts:
    - name: content
      mountPath: /html
    command: ["/bin/sh", "-c"]
    args:
      - while true; do
            echo "<h1>Current time:$(date +%T)</h1>" > /html/index.html;
          sleep 1;
        done

  volumes:                  # Volume which will be shared
  - name: content
    emptyDir: {}            # If POD gets removed, this data will be lost...but its safe if container is crashing
```

Try this:

```sh
kubectl apply -f https://raw.githubusercontent.com/C123R/kubernetes-hands-on/master/examples/pods/multiContainerPod.yaml -n $(whoami)

kubectl port-forward nginx-mc -n $(whoami) 8080:80
```

### **ReplicaSet**

One of the main reason bare pods cant be deployed directly on Production environment as it could die for many reasons like Node failure, ran out of resources etc. So we cant keep track on number of running PODs.

**ReplicaSet/ReplicaController** is one level above the Pod that ensures a certain number of PODs are always running.

```sh
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
spec:
  replicas: 3                               # Number of PODs
  selector:                                 # Pod Label Selector
    matchLabels:
      env: demo
  template:                                 # Pod Template
    metadata:
      labels:
        env: demo
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - name: http
          containerPort: 80
```

```sh
kubectl create -f https://raw.githubusercontent.com/C123R/kubernetes-hands-on/master/examples/replicaSet/replicaSet.yaml  -n $(whoami)
```

```sh
kubectl get rs -n $(whoami)
```

The difference between a replica set and a replication controller - replica set supports **set-based selector** requirements whereas a replication controller only supports **equality-based selector** requirements.

**ReplicaSet**
```sh
spec:
  replicas: 3
  selector: 
    matchLabels: 
      tier: frontend
```

**ReplicaController**
```sh
spec:
  replicas: 1  
  selector:    
    name: frontend
````

### **Deployment**

**Deployment** is one level above the ReplicaSet/Replication Controller which keeps set of identical pods running and upgrade them in a controlled way.

```sh
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

Create a deployment:
```sh
kubectl apply -f https://k8s.io/examples/controllers/nginx-deployment.yaml -n $(whoami)
````

Get the deployments:
```sh
kubectl get deployments -n $(whoami)

kubectl get rs -n $(whoami)

kubectl get pods --show-labels -n $(whoami)
```

Note:  `pod-template-hash` allows us to know which pods have been created by which ReplicaSet.

Horizontal Pod Autoscalers (HPA)!!!