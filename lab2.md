# Introduction to Kubernetes

Lab modified to be used with OpenShift:

## First contact with a Kubernetes

It's the first time that we interact with a Kubernetes Cluster, so let's get to know the basics.

1. Kubernetes clusters have nodes, those nodes can be control-plane nodes or compute nodes:

   ```sh
   kubectl get nodes

   NAME                         STATUS   ROLES                  AGE   VERSION
   test-cluster-control-plane   Ready    control-plane,master   17m   v1.21.1
   test-cluster-worker          Ready    <none>                 16m   v1.21.1
   ```

2. We have namespaces. Namespaces provides a mechanism for isolating groups of resources within a single cluster. Names of resources need to be unique within a namespace, but not across namespaces. Namespace-based scoping is applicable only for namespaced objects (e.g. Deployments, Services, etc) and not for cluster-wide objects (e.g. StorageClass, Nodes, PersistentVolumes, etc).

   ```sh
   kubectl get namespaces

   NAME                 STATUS   AGE
   default              Active   4m22s
   ingress-nginx        Active   2m19s
   kube-node-lease      Active   4m24s
   kube-public          Active   4m24s
   kube-system          Active   4m24s
   local-path-storage   Active   4m19s
   ```

3. We also have pods. Pods are the smallest deployable units of computing that you can create and manage in Kubernetes. A pod can contain from 1 to many containers inside.

   ```sh
   kubectl get pods -A

   NAMESPACE            NAME                                                 READY   STATUS    RESTARTS   AGE
   kube-system          coredns-558bd4d5db-85zhb                             1/1     Running   0          23m
   kube-system          coredns-558bd4d5db-b54xr                             1/1     Running   0          23m
   kube-system          etcd-test-cluster-control-plane                      1/1     Running   0          23m
   kube-system          kindnet-nhjtp                                        1/1     Running   0          23m
   kube-system          kindnet-r7dgx                                        1/1     Running   0          23m
   kube-system          kube-apiserver-test-cluster-control-plane            1/1     Running   0          23m
   kube-system          kube-controller-manager-test-cluster-control-plane   1/1     Running   0          23m
   kube-system          kube-proxy-r9grx                                     1/1     Running   0          23m
   kube-system          kube-proxy-tvhj8                                     1/1     Running   0          23m
   kube-system          kube-scheduler-test-cluster-control-plane            1/1     Running   0          23m
   local-path-storage   local-path-provisioner-547f784dff-jrh72              1/1     Running   0          23m
   ```

4. Services are also part of Kubernetes. A service is an abstract way to expose an application running on a set of Pods as a network service.

   ```sh
   kubectl get services -A

   NAMESPACE     NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
   default       kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  24m
   kube-system   kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   24m
   ```

5. There are a lot more objects that we're not going to cover, these are the basics and you will get introduced to new objects as you go.

## Deploying test application

In the previous lab we deployed a Pacman application, let's get it deployed on Kubernetes!

We have the required manifests present in these folders, we will use Kubectl to load them into the cluster:

- [Pacman game deployment files](./demo2-assets/pacman/)
- [MongoDB deployment files](./demo2-assets/mongo/)

1.  Deploy MongoDB

    1. Create the Namespace

       ```sh
       oc new-project <your-name>-mongo
       ```

    2. Create the Secret with Mongo credentials

       ```sh
       kubectl create -f demo2-assets/mongo/secret.yaml
       ```

    3. Create the Deployment

       ```sh
       kubectl create -f demo2-assets/mongo/deployment.yaml
       ```

    4. Create the Service

       ```sh
       kubectl create -f demo2-assets/mongo/service.yaml
       ```

    5. Check the Mongo Pod

       ```sh
       kubectl -n pacman-game-mongo get pods

       NAME                     READY   STATUS    RESTARTS   AGE
       mongo-545f984cb8-bfbss   1/1     Running   0          14s
       ```

2.  Deploy Pacman

        1. Create the Namespace

            ~~~sh
            oc new-project <your-name>-pacman
            ~~~
        2. Create the ClusterRole

            ~~~sh
            kubectl create -f demo2-assets/pacman/cluster-role.yaml
            ~~~
        3. Create the ClusterRoleBinding. But before, modify inside the file the namespace atribute to your pacman namespace

        ```yaml
        apiVersion: rbac.authorization.k8s.io/v1
        kind: ClusterRoleBinding
        metadata:
            name: pacman
        subjects:
            - kind: ServiceAccount
            name: pacman
            namespace: <your-name>-pacman
        roleRef:
            kind: ClusterRole
            name: pacman
            apiGroup: rbac.authorization.k8s.io
        ```

          ~~~sh
          kubectl create -f demo2-assets/pacman/cluster-role-binding.yaml
          ~~~
      4. Create the Secret with Mongo credentials

          ~~~sh
          kubectl create -f demo2-assets/pacman/secret.yaml
          ~~~
      5. Create the ServiceAccount

          ~~~sh
          kubectl create -f demo2-assets/pacman/service-account.yaml
          ~~~
      6. Create the Deployment

          ~~~sh
          kubectl create -f demo2-assets/pacman/deployment.yaml
          ~~~
      7. Create the Service

          ~~~sh
          kubectl create -f demo2-assets/pacman/service.yaml
          ~~~
      8. Create the Route. But before change the host attribute with the name of your namespace:

      ```yaml
        apiVersion: route.openshift.io/v1
        kind: Route
        metadata:
        annotations:
            haproxy.router.openshift.io/balance: roundrobin
            haproxy.router.openshift.io/disable_cookies: 'true'
        labels:
            name: pacman
        name: pacman
        spec:
        host: pacman-<your-name>-pacman.apps.gva-workshop.sandbox858.opentlc.com
        port:
            targetPort: 8080
        to:
            kind: Service
            name: pacman
            weight: 100
        wildcardPolicy: None
      ```

          > **NOTE:** Routes are a resource from OpenShift, not Kubernetes.

          ~~~sh
          kubectl create -f demo2-assets/pacman/route.yaml
          ~~~

      9. Check the Pacman Pod

          ~~~sh
          kubectl get pods

          NAME                      READY   STATUS    RESTARTS   AGE
          pacman-57859c8df7-879kt   1/1     Running   0          27s
          ~~~

3. Access the game in your Fedora35 node IP in port 80.

4. If at some point we need more capacity on the frontend of our application we can scale the deployment, run the scale command and check how if you access the app the request is handled by different pods:

   1. Scale the deployment so we have 5 pods

      ```sh
      kubectl scale deployment pacman --replicas 5
      ```

   2. Check the new pods that were created

      ```sh
      kubectl get pods

      NAME                      READY   STATUS    RESTARTS   AGE
      pacman-57859c8df7-879kt   1/1     Running   0          6m32s
      pacman-57859c8df7-9d7lf   1/1     Running   0          12s
      pacman-57859c8df7-g7bvn   1/1     Running   0          12s
      pacman-57859c8df7-glf7k   1/1     Running   0          12s
      pacman-57859c8df7-j5tfq   1/1     Running   0          12s
      ```

   3. Access the app and take a look at the `Host` text

## Cleanup

Delete both namespaces you have created in the cluster:

```sh
kubectl delete namespace <your-name>-mongo
kubectl delete namespace <your-name>-pacman
```
