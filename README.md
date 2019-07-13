# kubernetes-hands-on

This set of hands-on covers fundamentals of Kubernetes.

It will take you through all required basics to get started with Kubernetes. By the end of this hands-on, you should able to deploy demo application.

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

    ```sh
    kubectl get pods nginx
    ```

    If you need more detailed information about a particular object, you can use **describe**.

    `kubectl describe <resource-name> <object-name>`

    ```sh
    kubectl describe pods nginx
    ```

    You can use different output format `-o,--output=''` in kubectl.

    ```sh
    kubectl describe pods nginx -o yaml
    ```

- **Creating, Updating, and Destroying Kubernetes Resources**

    `kubectl apply` is the the recommended way of managing Kubernetes applications.  

     ```sh
    kubectl apply -f examples/pods/pod.yaml

    # Here you dont need to specify resource type, as its part of the yaml.
    ```

    You can still make changes to this `examples/pods/pod.yaml` and apply it again to update the object, or you can use `edit` command to make changes interactively.

    `kubectl edit <resource-name> <obj-name>`

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
    ```sh
    kubectl delete -f examples/pods/pod.yaml
    ```

- **Debugging**

    Important debugging commands 
    ```sh
    kubectl logs <pod-name>
    kubectl exec -it <pod-name> -- bash
    ```
    

