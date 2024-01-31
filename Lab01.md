# Lab 01 - Containers and Kubernetes

## Logging On

First you need to connect to an AWS EC2 instance in the cloud that will be your system to play with.

Your instructor team will help you with the details of the connection.

We will be using SSH to connect to the VM, and the username and password are as follows:

**username:** student
**password:** c0nygre

That's a zero not an oh, by the way.

If you're using PowerShell or GitBASH you can type in the following to connect to your VM.  Let's use an example, and we'll say that the VM's name is mskube01.neueda.me:

```bash
ssh student@mskube01.neueda.me
```

You'll be prompted for the student password.

NOTE: Nothing will display when you type in the password, so trust your typing, and type in **c0nygre**.  Then press **ENTER**.

You should see something like:

```
Last login: Mon Jan 29 17:00:30 2024 from cpc89512-gill19-2-0-cust354.20-1.cable.virginm.net
   ,     #_
   ~\_  ####_        Amazon Linux 2
  ~~  \_#####\
  ~~     \###|       AL2 End of Life is 2025-06-30.
  ~~       \#/ ___
   ~~       V~' '->
    ~~~         /    A newer version of Amazon Linux is available!
      ~~._.   _/
         _/ _/       Amazon Linux 2023, GA and supported until 2028-03-15.
       _/m/'           https://aws.amazon.com/linux/amazon-linux-2023/

[student@ip-172-31-34-89 ~]$
```

Now on to some serious magic.

## Minikube

Minikube is a developer version of Kubernetes and allows you to try out all the features of Kubernetes, including cluster and application management.

In this first lab we will:
- Start up minikube
- Enable a web proxy to allow us to see our applications
- View the cluster
- View details about containers that make up Kubernetes

### Before we begin

First we'll make sure there are no instances of Minikube left laying around on the system.  Run the following command:

```bash
minikube delete
```

### Starting Minikube

Try the following, as it should work with no other options:

```bash
minikube start --memory 4096 --kubernetes-version=latest
```

**NOTE:** if you find issues with ingress controllers failing to run, or tunneling not working (you'll see these later), then increase the **4096** to **8192**.  The only reason you would need to do this is if you've tried launching lots of containers or memory hungry applications on the server.  These labs have been tested and all work with 4096.

If the above returns an error try:

```bash
minikube start --memory 4096 --driver=containerd --bootstrapper=kubeadm --kubernetes-version=latest
```

This command takes a little while to run while it sets up the cluster, but not longer than 3 - 5 minutes.

We also need to enable an **ingress** controller so that we will be able to see our web based container applications.  To do this run the following command:

```bash
minikube addons enable ingress
```

This command takes a while to run, so be patient as it has to download the container and provision it in the cluster.  During the configuration you will see:

```
💡  ingress is an addon maintained by Kubernetes. For any concerns contact minikube on GitHub.
You can view the list of minikube maintainers at: https://github.com/kubernetes/minikube/blob/master/OWNERS
🔎  Verifying ingress addon...
```

This can take up to 3 minutes.

### Viewing the Cluster

It is possible in Kubernetes to view the nodes that make up the actual cluster.

#### Credentials

Unlike most systems Kubernetes requires you to have a configuration file in place to connect to your server.

If you are working with multiple servers you may have multiple files, or a single file with multiple configurations.  For this exercise there is only one configuration file and one **context**.

The file is located in your home directory.

1. Run the following command to list the credentials file directory content:

    ```bash
    ls ~/.kube
    ```

    NOTE: The output will be:

    ```
    cache config
    ```

2. **config** is the file we are interested in, so let's take a look inside. Run the following command:

    ```bash
    less ~/.kube/config
    ```

    Kubernetes works on **contexts** to determine the profile you wish to use to connect to a cluster.  In our case we only have 1 context called **minikube**.  We can see this as follows:

    ```yaml
    contexts:
    - context:
        cluster: minikube
        extensions:
        - extension:
            last-update: Mon, 29 Jan 2024 17:03:48 UTC
            provider: minikube.sigs.k8s.io
            version: v1.29.0
          name: context_info
        namespace: default
        user: minikube
      name: minikube
    ```

    The **name:** is the values that can be provided to change the Kubernetes cluster you wish to work with.

3. You can change your cluster by using the following command:

    ```bash
    kubectl config --kubeconfig=$HOME/.kube/config use-context minikube
    ```

    Where **minikube** after **use-context** is changed to the name of the cluster context you want to connect.

    The **--kubeconfig=** specifies the name of the file containing the contexts.

    Running the above command should return:

    ```
    Switched to context "minikube".
    ```

4. Try running the following:
   
    ```bash
    kubectl config --kubeconfig=$HOME/.kube/config use-context dev
    ```

    This will fail and return:
    ```
    error: no context exists with the name: "dev"
    ```

It is also possible that if you're using a single file you can set a shell variable called **KUBECONFIG**.  The value of the shell variable is the absolute path to the file containing the contexts.

1. Let's try changing the variable.  First let's try a file that does not exist:

    ```bash
    KUBECONFIG=/broken/config
    export KUBECONFIG
    kubectl get nodes
    ```

    When we run the above 3 commands we are first setting the variable to a non-existing file called **/broken/config**.

    We then **export** the variable so that a command called **kubectl** can be used to talk to the cluster using the default context within the file.

    When we run the **kubectl get nodes** command we get an error similar to:

    ```
    W0129 17:30:23.159941   59036 loader.go:222] Config not found: /broken/config
    E0129 17:30:23.167034   59036 memcache.go:238] couldn't get current server API group list: Get "http://localhost:8080/api?timeout=32s": dial tcp 127.0.0.1:8080: connect: connection refused
    ```
    
2. Let's now set the variable to the valid configuration file:

    ```bash
    KUBECONFIG=$HOME/.kube/config
    echo $KUBECONFIG
    kubectl get nodes
    ```

    This time we reference the same file we did in the previous exercise.

    NOTE: we did not need the **export** command since we've already ran that command and the variable is already available to the current terminal.  Instead we used the **echo** command to see the content of the variable.

    This time running the **kubectl** command we get back a list of nodes that make up our cluster:

    ```
    NAME       STATUS   ROLES           AGE   VERSION
    minikube   Ready    control-plane   30m   v1.26.1
    ```

3. We can also unset the variable as it will default to **$HOME/.kube/config**

    ```bash
    unset KUBECONFIG
    echo $KUBECONFIG
    kubectl get nodes
    ```

    The command still works, and you'll notice that **echo** prints blank as there is no content in the variable.

## Looking around

Let's take a quick look around what's in our Kubernetes mini cluster.

1. We've already seen how to view the nodes (hosts) that make up the cluster and run all the containers and manage the environments:

    ```bash
    kubectl get nodes
    ```

    The **get** command provides basic information about a resource in Kubernetes.  In this case the resource is **nodes**.

    This command allows us to see some basic information about the cluster:

    ```
    NAME       STATUS   ROLES           AGE   VERSION
    minikube   Ready    control-plane   31m   v1.26.1
    ```

    We can see the name given to the cluster, and the **ROLE** given to the node.  Since we are only running a single node cluster we only see a **control-plane**.  This is acting as both the manager of the cluster and the **worker** node.

    Worker nodes are where your applications will run.

2. If we want to see more detailed information about a resource by changing **get** to **describe**.

    ```bash
    kubectl describe nodes
    ```

    You'll see a lot of detail here, but the values have keys to let you know what they are.  You see things such as:

    - CreationTimeStamp for when the cluster was created.
    - Labels which will allow you to target groups of resources with the same labeling.
    - Node details, such as memory, CPU, IP addressed, etc.
    - Pods.  These are containers running that make up the Kubernetes cluster.
    - Events.  These are like looking at log files and tell you what is happening on the nodes and what Kubernetes is doing.

    We'll look more at these commands later and see how they change when we look at different resources.

3. Let's get more detail, but in YAML format, to see what the node configuration looks like in Kubernetes code, called a manifest.

    ```bash
    kubectl get nodes minikube -o yaml
    ```

    NOTE: This time we targeted a specific node called **minikube** as mentioned in the **get** and **describe** commands.  We have also add an extra option **-o yaml** which provides detailed output in Kubernetes manifest format, using the YAML file format syntax.

    You should see some similarities to the **describe**, but this is information stored within Kubernetes control-plane that defines how the node is configured.

    We'll discuss manifest in more detail in the next section that relate to applicaitons.

# END OF LAB 1