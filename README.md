# kubernetes-hands-on

This set of hands-on covers fundamentals of Kubernetes.

It will take you through all required basics to get started with Kubernetes. By the end of this hands-on, you should able to deploy demo application.

## kubectl

`kubectl` is a Kubernetes CLI client. You will use it to create, delete and inspect various Kubernetes objects.

### Installation

#### For MacOS

```bash
brew install kubernetes-cli
```
Others can follow instructions from [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

#### Kubernetes Config

There is a kubeconfig file behind every working kubectl command. Default location for kubeconfig is: `$HOME/.kube/config`.You can specify other kubeconfig files by setting the `KUBECONFIG` environment variable or by setting the `--kubeconfig` flag.

- Example kubeconfig file

```bash
apiVersion: v1
kind: Config
preferences: {}
current-context: foo-cluster
clusters:
- cluster:
    server: http://foo.com:8080
    certificate-authority-data: XXXXXXXXXXXXXXX
  name: foo-cluster
- cluster:
    server: http://bar.com:8080
    certificate-authority-data: XXXXXXXXXXXXXXX
  name: bar-cluster
contexts:
- context:
    cluster: foo-cluster
    user: foo-user
  name: foo-cluster
- context:
    cluster: bar-cluster
    user: bar-user
  name: bar-cluster
kind: Config
preferences:
  colors: true
users:
- name: foo-user
  user:
    token: XXXXXXXXXXX
    client-certificate-data: XXXXXXXXX
    client-key-data: XXXXXXXXX
- name: bar-user
  user:
    token: XXXXXXXXXXX
    client-certificate-data: XXXXXXXXX
    client-key-data: XXXXXXXXX
```

```bash
$ kubectl cluster-info
```