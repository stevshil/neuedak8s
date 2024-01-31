# More Labs

If you finish the core labs, take a look at these and practice an area that looks of interest.

## APIs

Take a look at the APIs on your Kubernetes (K8s) server:

```bash
kubectl api-resources
```

Do some web searching to see what they do and if you can find the API documentation that would allow you to create a YAML manifest.  The documentation page for all APIs starts at [https://kubernetes.io/docs/concepts/overview/kubernetes-api/](https://kubernetes.io/docs/concepts/overview/kubernetes-api/).

Attributes within these APIs can be found by digging in to the programmatic documentation [https://kubernetes.io/docs/reference/kubernetes-api/](https://kubernetes.io/docs/reference/kubernetes-api/).

Think of the YAML document as a hierarchy, e.g.  **metadata.name**, or **metadata.labels.name** as in the first YAML file we created **myfirstapp.yml**.

For example the **Deployment** API is [https://kubernetes.io/docs/concepts/workloads/controllers/deployment/](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

The API documentation detail for attributes in the YAML manifest start at [https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/deployment-v1/](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/deployment-v1/) as there a links off to other pages that provide common attributes.

## Alternative deployments

### Stateful sets

1. Create a stateful set

    - [https://redhat-scholars.github.io/kubernetes-tutorial/kubernetes-tutorial/statefulset.html](https://redhat-scholars.github.io/kubernetes-tutorial/kubernetes-tutorial/statefulset.html)
    - Modifications required to StatefulSet YAML file

        ```yaml
        apiVersion: apps/v1
        ```
        
        Not v1beta1

    - Under the first **spec** section add

        ```yaml
        selector:
            matchLabels:
                app: quarkus-statefulset
        ```
        
    - Don't forget to add **-n myspace** to the end of all **kubectl** commands

### Scaling

1. All about scaling, including manual

    - [https://bluexp.netapp.com/blog/cvo-blg-kubernetes-scaling-the-comprehensive-guide-to-scaling-apps](https://bluexp.netapp.com/blog/cvo-blg-kubernetes-scaling-the-comprehensive-guide-to-scaling-apps)

2. Autoscaling

    - Horizontal Pod Autoscaling (HPA)
      - [https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)
    - Scaling Statefulsets
      - [https://kubernetes.io/docs/tasks/run-application/scale-stateful-set/](https://kubernetes.io/docs/tasks/run-application/scale-stateful-set/)

## Environments, variables and configuration

### ConfigMaps

1. [https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)

### Secrets

1. Working with secrets and using in a container

    - [https://redhat-scholars.github.io/kubernetes-tutorial/kubernetes-tutorial/secrets.html](https://redhat-scholars.github.io/kubernetes-tutorial/kubernetes-tutorial/secrets.html)

2. Creating secrets and using secrets in a container

    - [https://spacelift.io/blog/kubernetes-secrets](https://spacelift.io/blog/kubernetes-secrets)

## Storage

1. Persistent volumes, 2 containers sharing storage

    - [https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/](https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/)

2. Set up a Persistent Volume and assign to a Pod

    - [https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)

## Security

1. Pod securityContext

    - [https://kubernetes.io/docs/tasks/configure-pod-container/security-context/](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

2. Service Accounts

    - [https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)