# Lab 3 - Helm

In this lab you will:

1. Install Helm
2. Add a helm repository
3. Launch an application using helm

## Install Helm

1. Make sure you are SSH'd on to your AWS EC2 Instance.
    ```bash
    ssh student@mskube1.neueda.me
    ```

2. Install help from script:

    ```bash
    curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
    chmod +x get_helm.sh
    ./get_helm.sh
    ```

3. Test helm works:

    ```bash
    helm --help
    ```

    You should see a lot of text with options and information.

## Adding a Helm repository

Here we will add the bitnami repository.

1. Run the following command to add the repository:

    ```bash
    helm repo add bitnami https://charts.bitnami.com/bitnami
    ```

    If successful you will see:

    ```
    "bitnami" has been added to your repositories
    ```

    The command tells helm to create a local name for the repository called **bitnami** and link it to the URL that relates to the repository information.

2. Let's search the repository

    ```bash
    helm search repo bitnami
    ```

3. Create a namespace to launch our app into:

    ```bash
    kuebctl create namespace ap
    ```

4. Let's install the apache application.  An application is better known as a chart.

    ```bash
    helm install apache bitnami/mediawiki --namespace ap
    ```

5. Check that it's coming up with:
   
   ```bash
   kubectl get all -n ap
   ```

6. We now need an Ingress controller, since this is not supplied in their chart.
   
7. Create a file using **nano** called **apingerss.yml**, and in that file add the following:

    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
    name: ap-ingress
    namespace: ap
    annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /$1
    spec:
    rules:
    - host: "ap.mskube1.neueda.me"
        http:
        paths:
        - pathType: Prefix
            path: "/"
            backend:
            service:
                name: apache
                port:
                number: 80
    ```

    **NOTE:** Don't forget to change the **host:** value to your DNS.  Keep the **ap** bit at the front though.

8. Save the file with **CTRL X**, **Y** and **ENTER**.

9.  Now apply this configuration:

    ```bash
    kubectl apply -f apingress.yml
    ```

10. Check the ingress is there:

    ```bash
    kubectl get ingress -n ap
    ```

11. Once the pods are **1/1** and your ingress is ready, point your web browser at your DNS value of the **host:** attribute in your ingress.

12. If all is working you will set **It Works!**

## Other commands

Try out the following commands.

List installed repositories:

```bash
helm repo list
```

List applications:

```bash
helm list --namespace ap
```

This will show your current deployment of Apache.

Show status information on an application:

```bash
helm status apache --namespace ap
```

Uninstall a release:

```bash
helm uninstall apache --namespace ap
```

Similar to delete.

## Further learning

Continue the rest of the lab at [https://helm.sh/docs/intro/using_helm/](https://helm.sh/docs/intro/using_helm/)

Try installing other repositories and using [https://artifacthub.io/](https://artifacthub.io/) which is like Docker Hub but for Helm charts.

Remove an application:

```bash
helm delete apache --namespace ap
```