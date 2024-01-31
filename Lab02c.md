# Lab 2 C - Optional

This is an optional lab, for if you have time.

In this lab you will create a ConfigMap to change the page displayed by NGINX.

## Files that change

In this lab we will:

1. Create a new file for the ConfigMap that will be the **index.html** file.
2. Modify the **myweb.yml** to use the ConfigMap and replace the default **index.html** file located in **/usr/share/nginx/html/**

## The ConfigMap

A ConfigMap allows you to create variables or file data that can be updated.  In this exercise you will create the content for the HTML file.

1. Use **nano** to create the file mywebconfig.yml
    
    ```bash
    nano mywebconfig.yml
    ```

2. Add the following content to the file:

    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
        name: index-html-configmap
        namespace: myfirstapp
    data:
        index.html: |
            <html>
            <h1>Ye Ha! Looks like you found my Kube webpage!</h1>
            </br>
            <h1>This web page is housed on a Pod running Nginx</h1>
            </html>
    ```

    This file can be found in [mywebconfig.yml](Lab02/mywebconfig.yaml)

    The main features in this are:
    - name:
      - Under **metadata** this will be the name of this map.
    - data:
      - Maps can contain multiple items of data which are named
    - index.html:
      - This is the name given to the piece of data in our ConfigMap the | symbol denotes that it is multi-line data.  The content of our file

3. Apply the map:

    ```bash
    kubectl apply -f mywebconfig.yml
    ```

4. Check the map exists:

    ```bash
    kubectl get configmap -n myfirstweb
    ```

    This will list **index-html-configmap**

    You can use **decribe** to see the content within the map.

## Modifying the application

1. Use **nano** to create a new file myweb2.yml.

    ```bash
    nano myweb2.yml
    ```

    This file can be found in [myweb2.yml](Lab02/myweb2.yml)

2. In this file add the following content:

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
            initialDelaySeconds: 15
            periodSeconds: 3
            readinessProbe:
            httpGet:
                path: /
                port: 80
            initialDelaySeconds: 15
            periodSeconds: 10
            volumeMounts:
            - name: nginx-index-file
            mountPath: "/usr/share/nginx/html/"
        volumes:
        - name: nginx-index-file
            configMap:
            name: index-html-configmap
    ```

    This contains exactly the same information as the original **myweb.yml** file, except we've added some lines to the bottom of the file relating to **volumes**.

    The detail is as follows:
    - volumeMounts:
      - This tell Kubernetes where the file will be created, and is specified as a directory using the **mountPath** attribute.
      - The **name** denotes the name of the **volume** that will be used within the directory.
    - volumes:
      - Defines the link to the data and the mount.
      - The **configMap** and the **name** define which ConfigMap will be used.
      - Since our config map only has 1 item **index.html** that will act as the file name and the content associated to the **index.html** key will be the file data.

3. We all ready have a deployment called **myweb** so running this command will update and create a new replicaset:

    ```bash
    kubectl apply -f myweb2.yml
    ```

4. Check that the new pod is launching:

    ```bash
    kubectl get pods -n myfirstweb
    ```

    As you saw earlier you will see a new ID for the new pod and eventually the old pod will be removed, when the new pod gets 1/1.

### Checking the app

Refresh your web browser to see the new page.  You will see:

![New NGINX Page](images/NGINXPage2.png)