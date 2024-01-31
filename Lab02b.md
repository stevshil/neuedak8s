# Lab 02B - Kubernetes Fundamentals

## Creating a deployment

Let us now work on deploying an application onto Kubernetes.

For this we will use a simple web server as the first part, and then an optional part will be to add a ConfigMap to make a web page that will be served instead of the default page.

## Launching a Web Server

Using our **myfirstapp** namespace created in the previous lab, we will create 3 new files to launch our web service.

1. The deployment of the NGINX web server container
2. A service that will front the container and act like a load balancer
3. An ingress to enable our application to be viewed externally

### The deployment configuration

1. First we need to create a file, let's call it **myweb.yml** and we'll do that using the following command:
2. 
    ```bash
    nano myweb.yml
    ```
3. In that file we will add the following content:

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: myweb
    labels:
        app: myweb
    namespace: myfirstapp
    spec:
    replicas: 1
    selector:
        matchLabels:
        app: myweb
    template:
        metadata:
        labels:
            app: myweb
            app.kubernetes.io/name: myweb
        spec:
        containers:
        - name: myweb
            image: nginx
            ports:
            - containerPort: 80
            resources:
                requests:
                    memory: "300Mi"
                limits:
                    memory: "1Gi"
            livenessProbe:
                httpGet:
                    path: /
                    port: 80
                initialDelaySeconds: 60
                periodSeconds: 3
            readinessProbe:
                httpGet:
                    path: /
                    port: 80
                initialDelaySeconds: 60
                periodSeconds: 10
    ```

    You can also see the file in the Lab02 folder [myweb.yml](Lab02/myweb.yml).

    In this file we have some notable lines:
    - replicas: 1
      - This determines how many containers we wish to launch on start up.  Useful if you are running high volume services
    - app.kubernetes.io/name: myweb
      - This line will be used by the service file we will create next to target these pods.
    - image: nginx
      - This is the image from [https://hub.docker.com](https://hub.docker.com) that we will use
    - The **resources:** section allows us to add some checks to determine if the container has started and whether it is still alive and has not used too much memory.  This is optional, but is good practice to include especially in production environments to prevent applications running away with all the CPU and memory, and to restart containers if they die.

4. Let's launch this container configuration.

    ```bash
    kubectl apply -f myweb.yml
    ```
5. Check that it starts

    ```bash
    kubectl get all -n myfirstapp
    ```

    The **-n myfirstapp** is required since our application is in that namespace.

    We are looking for a **1/1** under **READY**.  0/1 means it is still starting.  We also need **STATUS** to be **Running**.

    ```
    NAME                         READY   STATUS        RESTARTS   AGE
    pod/myweb-84c468cdcb-gdfqq   1/1     Running       0          21s
    ```

#### Troubleshooting

If your container has a different status from **Running**, such as **CrashBackoff**, or something else then there may be a typo in the YAML file.

We can check a containers log files by running:

```bash
kubectl logs pod/myweb-84c468cdcb-gdfqq -n myfirstapp
```

Where we would replace the pod name and namespace accordingly.

If you wish to follow the log file for live view add **-f** as follows:

```bash
kubectl logs -f pod/myweb-84c468cdcb-gdfqq -n myfirstapp
```

This is similar to using **tail -f** in Linux.

You will need to press **CTRL C** to quit the follow.

## Add the Service

Now we need to be able to target our service, and make it scalable.  Here we will create the **Service** configuration.

1. Create a new text file as follows:

    ```bash
    nano mywebservice.yml
    ```

2. In this file add the following:

    ```yaml
    apiVersion: v1
    kind: Service
    metadata: 
    name: myweb-service
    labels:
        svc: myweb-service
    namespace: myfirstapp
    spec:
    selector: 
        app.kubernetes.io/name: myweb
    ports:
        - protocol: TCP
        port: 80
        targetPort: 80
    ```

    This file can be found in Lab02 folder [mywebservice.yml](Lab02/mywebservice.yml).

    NGINX is a web server and listens on port 80 by default.  The **ports** section defines that the service will listen on **port** 80 and the service will connect to the **targetPort** of the container.

    The **selector:** section contains the **app.kubernetes.io/name: myweb** label from our deployment, so any container with this label will be accessible by the service.

    All containers within this namespace will be able to access this service using the **name:** myweb-service as though it was a hostname.

3. Let's now launch the service:

    ```bash
    kubectl apply -f mywebservice.yml
    ```

4. Check it worked

    ```bash
    kubectl get services -n myfirstapp
    ```

    This will show

    ```
    NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
    myweb-service   ClusterIP   10.103.31.26   <none>        80/TCP    53m
    ```

    This doesn't really tell us much, so you could change **get** with **describe** for more detail.

    The main things we see is that the service is a **ClusterIP** which means it is internal to the cluster.  It is possible to use **NodePort** to make the service use a port number from the real Kubernetes nodes across the whole cluster.  Google for **Kubernetes NodePort service** for more detail.

## Let's see this on the web

### Enabling Minikube to be visible externally

To be able to view our application through the Web browser we will need to do the following to enable minikube to access the cluster from the outside world.

1. Open another terminal (PowerShell or GitBASH) so that you can SSH onto the AWS EC2 instance.
2. SSH on to the server in this new terminal window.
3. Once connected run the following command:

    ```bash
    minikube tunnel --bind-address=172.31.34.89
    ```

    **NOTE:** you will need to change **172.31.34.89** to your instances IP address.  You can find the IP address in your shell prompted:

    ```bash
    [student@ip-172-31-34-89 ~]$
    ```

    In this case you can see **ip-172-31-34-89**, so removing the **ip-** and changing the **-** to **.** will be your IP that you set for the **bind-address** value.

4. You will be prompted for your password as this command needs root privileges.  It might not appear immediately, so keep checking.

5. Leave this window open.

### Making our service visible

Here we will create an Ingress rule.  An Ingress rule is required by Kubernetes to make your service available to the outside world through the Ingress controller using a DNS name or URL path.

In this example we will be using a DNS name.  The DNS name will be based on the name you use with SSH, but with an extra bit at the front.

For example, if you SSH as follows:

```bash
ssh student@mskube1.neueda.me
```

Your DNS name is:
**mskube1.neueda.me**

The name we'll be giving the application will be:
**myweb.mskube1.neueda.me**

1. Create a new file as follows:

    ```bash
    nano mywebingress.yml
    ```

2. In this file add the following:

    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
    name: myweb-ingress
    namespace: myfirstapp
    annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /$1
    spec:
    rules:
    - host: "myweb.mskube1.neueda.me"
        http:
        paths:
        - pathType: Prefix
            path: "/"
            backend:
            service:
                name: myweb-service
                port:
                number: 80
    ```

    This file is available in the Lab02 folder [mywebingress.yml](Lab02/mywebingress.yml).

    In this file, here are some highlights:
    - apiVersion:
      - Note how this is different to other manifests.  Earlier in the notes we showed how to list the apis.
      - For example:
        
        ```bash
        kubectl api-resources | grep ingress
        ```

        You'll see the version in the second column.
    - annotations:
      - Deals with any URL rewrites
    - host:
      - Defines the DNS hostname
      - You need to change this to your DNS name
    - service:
      - This links the ingress rule to your service by name and port

3. Apply the ingress rule:
    
    ```bash
    kubectl apply -f mywebingress.yml
    ```

4. Check the ingress has been created:

    ```bash
    kubectl get ingress -n myfirstapp
    ```

    The output should show your rule with your DNS name:

    ```
    NAME            CLASS   HOSTS                     ADDRESS        PORTS   AGE
    myweb-ingress   nginx   myweb.mskube1.neueda.me   192.168.58.2   80      77m
    ```

### View the app

You should now be able to open your web browser and type in the DNS name:

**myweb.mskube1.neueda.me**

Substituting the **mskube1.neueda.me** for the name of your server.  If it is working and all configured you should see:

![NGINX Default Page](images/NGINXPage.png)