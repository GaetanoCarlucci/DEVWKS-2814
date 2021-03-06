# DEVWKS-2814: Let's Play with Istio
<br>

You can reproduce the [DevNet](https://devnetsandbox.cisco.com/) workshop by reserving the Sandbox here: [Istio 1.2](https://devnetsandbox.cisco.com/RM/Diagram/Index/8b9512a7-d2e5-4699-8d7a-393d7434982f?diagramType=Topology) <br>
The Sandbox contains an up and running Kubernetes cluster with Istio enabled that you can use to play with Istio!

**Session Code:**  DEVWKS-2814 <br>
**WebEX Room:** http://cs.co/eventsbot#DEVWKS-2814 <br>
**AnyConnect link:** <ins>devnetsandbox-us-sjc.cisco.com:XXXXX</ins> <em>(Port is on your desk)</em><br>
**AnyConnect UserName:** EventUser <br> 
**AnyConnect Password:** XXXXX <em> (Password is on your desk)</em><br>  
**SSH to Kubernetes Master:** ssh developer@10.10.20.21 <br> 
**SSH password:** C1sco12345 <br> 
<br>

**Title:** DevNet Workshop: Let's Play with Istio: service meshes for high-scale container environments<br>
<br>
**Abstract:**<br>
Istio is an open source service mesh which is also packaged and supported in the Cisco Container Platform (CCP). With Istio, you can securely connect, control, and observe microservices on large scale deployments. The goal of this workshop is to give you the opportunity to use an Istio-enabled Kubernetes cluster which can be created with CCP and get familiar with some use cases like service routing, traffic shifting and service tracing. This will be done by using both manifest files manipulation and REST API calls. <br><br>
 **Agenda:**<br>
 <ol>
<li>Introduction</li>
<li>Cisco Container Platform</li>
<li>Istio Control Plane</li>
<li>BookInfo Application</li>
<li>	Istio usage example </li>
 <ol>
     <li>	Route based on application version</li>
    <li>	Route based on user identity   </li>
    <li>	Traffic shifting with/without API</li>
</ol>
 <li>Conclusion</li>
 </ol>

## Sandbox layout

  ![Sandbox layout](https://github.com/GaetanoCarlucci/DEVWKS-2814/blob/master/sandbox_layout.PNG)

## Bookinfo App: service mesh topology without Istio
Bookinfo Application is a virtual library for books description and ratings. It is a webpage that shows the book details, reviews, and ratings from readers. <br><br>
It consists of 4 four separate microservices (product page, detail, review, rating) written in different program languages. This is the value of microservice being each microservice completely independent from each other. <br><br>
Each box in the picture is a Kubernetes deployment with a Kubernetes service attached to it. Each deployment has one pod with one container inside. <br><br>
The review is divided into three deployments each one with a different version. **Version 1** does not have a connection to the rating service, **version 2** shows the ratings stars in black and **verions 3** in red. <br><br>
By default, without Istio the product page accesses to the review versions in a round-robin fashion.
  ![Bookinfo App](https://github.com/GaetanoCarlucci/DEVWKS-2814/blob/master/bookinfo_mesh_topology_without_Istio.PNG)

## Bookinfo App: service mesh topology with Istio
When we add Istio to our application service mesh, we add one sidecar container to our pods that intercepts the requests coming in and out the pod. <br><br>
In this way, all traffic that the mesh services send and receive (data plane traffic) is proxied through Envoy, making it easy to direct and control traffic around the service mesh without making any changes to your services.<br><br>
Each pod in the review service has a Kubernetes label that defines the version of the app. This is important for Istio to route traffic based on that version.<br><br>
  ![Bookinfo App](https://github.com/GaetanoCarlucci/DEVWKS-2814/blob/master/bookinfo_mesh_topology.PNG)


## Sequence of commands

### Get kubernetes cluster nodes
<pre>kubectl get nodes -o wide</pre>
##### Expected output
<pre>
NAME        STATUS   ROLES    AGE    VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION               CONTAINER-RUNTIME
netmaster   Ready    master   163m   v1.15.0   10.10.20.21   <none>        CentOS Linux 7 (Core)   3.10.0-957.21.3.el7.x86_64   docker://1.13.1
worker1     Ready    <none>   162m   v1.15.0   10.10.20.22   <none>        CentOS Linux 7 (Core)   3.10.0-957.21.3.el7.x86_64   docker://1.13.1
worker2     Ready    <none>   162m   v1.15.0   10.10.20.23   <none>        CentOS Linux 7 (Core)   3.10.0-957.21.3.el7.x86_64   docker://1.13.1
worker3     Ready    <none>   162m   v1.15.0   10.10.20.24   <none>        CentOS Linux 7 (Core)   3.10.0-957.21.3.el7.x86_64   docker://1.13.1
</pre>

### Verify Istio Control Plane Installation
<pre>kubectl get pods -n istio-system</pre>
##### Expected output
<pre>NAME                                      READY   STATUS      RESTARTS   AGE
grafana-6575997f54-vsfmx                  1/1     Running     0          3h42m
istio-citadel-894d98c85-bk6jk             1/1     Running     0          3h42m
istio-cleanup-secrets-1.2.2-trgwh         0/1     Completed   0          3h42m
istio-egressgateway-9b7866bf5-2lr6k       1/1     Running     0          3h42m
istio-galley-5b984f89b-5czkx              1/1     Running     0          3h42m
istio-grafana-post-install-1.2.2-t8n4b    0/1     Completed   0          3h42m
istio-ingressgateway-75ddf64567-n6949     1/1     Running     0          3h42m
istio-pilot-5d77c559d4-9rnsk              2/2     Running     0          3h42m
istio-policy-86478df5d4-p8c2s             2/2     Running     1          3h42m
istio-security-post-install-1.2.2-5x2cb   0/1     Completed   0          3h42m
istio-sidecar-injector-7b98dd6bcc-mjpnj   1/1     Running     0          3h42m
istio-telemetry-786747687f-8tkgs          2/2     Running     1          3h42m
istio-tracing-555cf644d-sb2hc             1/1     Running     0          3h42m
kiali-6cd6f9dfb5-84qhd                    1/1     Running     0          3h42m
prometheus-7d7b9f7844-l6ngg               1/1     Running     0          3h42m
</pre>

### Bookinfo Application without Istio
<pre>cd /home/developer/istio-1.2.2/samples/bookinfo/platform/kube </pre>
<pre>cat bookinfo.yaml </pre>

### Enabling Istio in Bookinfo Application
<pre>istioctl kube-inject -f bookinfo.yaml > bookinfo_with_istio.yaml </pre>
<pre>cat bookinfo_with_istio.yaml </pre>
<pre>kubectl apply -f bookinfo_with_istio.yaml  </pre>
<pre>kubectl get pods -o wide </pre>
##### Expected output
<pre>
NAME                             READY   STATUS    RESTARTS   AGE     IP           NODE      NOMINATED NODE   READINESS GATES
details-v1-677976f7dc-2zhzw      2/2     Running   0          2m34s   10.40.0.6    worker2   none           none
productpage-v1-f76db69c5-lsd9h   2/2     Running   0          2m33s   10.46.0.9    worker3   none           none
ratings-v1-7f66cd4c87-8vnbq      2/2     Running   0          2m34s   10.46.0.4    worker3   none           none
reviews-v1-77bcd5d4f5-zcfm4      2/2     Running   0          2m34s   10.38.0.6    worker1   none           none
reviews-v2-7cdb7475fb-wjqrl      2/2     Running   0          2m34s   10.46.0.10   worker3   none           none
reviews-v3-8496dbbbbf-4ftsq      2/2     Running   0          2m34s   10.38.0.5    worker1   none           none
</pre>

### Bookinfo Application add ingress gateway
<pre>cd /home/developer/istio-1.2.2/samples/bookinfo/networking/ </pre>
<pre>kubectl apply -f bookinfo-gateway.yaml </pre>
Retrive external IP:
<pre>kubectl get svc -n istio-system | grep ingress </pre>
##### Expected output
<pre>istio-ingressgateway     LoadBalancer   10.107.63.250    10.10.20.30   15020:30392/TCP,80:31380/TCP,443:31390/TCP,31400:31400/TCP,15029:30194/TCP,15030:30191/TCP,15031:31372/TCP,15032:32031/TCP,15443:31403/TCP   3h46m</pre>

### Bookinfo Application: try it out 
http://10.10.20.30/productpage 
##### Expected output
  ![Web Page](https://github.com/GaetanoCarlucci/DEVWKS-2814/blob/master/page.PNG)

### Route based on application version: DestinationRule
We need to create a Destination Rule to define service subset which for example we group pods by their version. <br>
Then we can use these service subsets in the routing rules of Istio virtual services to control the traffic to different instances of mesh services. <br>
In this example, we create 3 subsets named “v1”, “v2” and “v3” that match respectively pods with labels **“version: v1”**, **“version: v2”** and **“version: v3”**. The labels are defined in the Kubernetes pods in the booking manifest file.
<pre>cd /home/developer/istio-1.2.2/samples/bookinfo/networking/ </pre>
<pre>kubectl apply -f destination-rule-all.yaml </pre>
##### Expected output
<pre>kubectl  get destinationrule
NAME          HOST          AGE
details       details       13s
productpage   productpage   13s
ratings       ratings       13s
reviews       reviews       13s
</pre>
### Route based on application version: VirtualService - Route all traffic to version 1
<pre>cd /home/developer/istio-1.2.2/samples/bookinfo/networking/ </pre>
<pre>kubectl apply -f virtual-service-all-v1.yaml </pre>
##### Expected output
<pre>kubectl describe virtualservice review
Name:         reviews
....
  Hosts:
    reviews
  Http:
    Route:
      Destination:
        Host:    reviews
        Subset:  v1
</pre>
### Exercise: Route all traffic to Version 2
Modify **virtual-service-all-v1.yaml**
<pre>
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v2
</pre>
Then deploy it:
<pre>kubectl apply -f virtual-service-all-v1.yaml </pre>
##### Expected output
<pre>kubectl describe virtualservice review
Name:         reviews
....
  Hosts:
    reviews
  Http:
    Route:
      Destination:
        Host:    reviews
        Subset:  v2
</pre>

### Route based on user identity
<pre>cd /home/developer/istio-1.2.2/samples/bookinfo/networking/ </pre>
Modify **virtual-service-reviews-jason-v2-v3.yaml** by inserting your name and apply it.
<pre>kubectl apply -f virtual-service-reviews-jason-v2-v3.yaml </pre>
##### Expected output
Verify that the virtual service has been implemented as expected:
<pre>kubectl describe virtualservice review
Name:         reviews
...
Spec:
  Hosts:
    reviews
  Http:
    Match:
      Headers:
        End - User:
          Exact:  gaetano
    Route:
      Destination:
        Host:    reviews
        Subset:  v2
    Route:
      Destination:
        Host:    reviews
        Subset:  v3
</pre>
  ![Login](https://github.com/GaetanoCarlucci/DEVWKS-2814/blob/master/login.PNG)


### Traffic shifting: 80% v1 - 20% v2
<pre>cd /home/developer/istio-1.2.2/samples/bookinfo/networking/ </pre>
<pre>kubectl apply -f virtual-service-reviews-80-20.yaml </pre>
##### Expected output
Verify that the virtual service has been implemented as expected:
<pre>kubectl describe virtualservice review
Name:         reviews
...
Spec:
  Hosts:
    reviews
  Http:
    Route:
      Destination:
        Host:    reviews
        Subset:  v1
      Weight:    80
      Destination:
        Host:    reviews
        Subset:  v2
      Weight:    20
</pre>

### Rest API example: Traffic shifting: 20% v1 - 80% v2 with API
<pre>curl -H "Accept: application/json" -H "Content-Type: application/merge-patch+json" -X PATCH http://localhost:8001/apis/networking.istio.io/v1alpha3/namespaces/default/virtualservices/reviews -d '{"metadata":{"annotations":{"kubectl.kubernetes.io/last-applied-configuration":"{\"apiVersion\":\"networking.istio.io/v1alpha3\",\"kind\":\"VirtualService\",\"metadata\":{\"annotations\":{},\"name\":\"reviews\",\"namespace\":\"default\"},\"spec\":{\"hosts\":[\"reviews\"],\"http\":[{\"route\":[{\"destination\":{\"host\":\"reviews\",\"subset\":\"v1\"},\"weight\":20},{\"destination\":{\"host\":\"reviews\",\"subset\":\"v2\"},\"weight\":80}]}]}}\n"}},"spec":{"http":[{"route":[{"destination":{"host":"reviews","subset":"v1"},"weight":20},{"destination":{"host":"reviews","subset":"v2"},"weight":80}]}]}}'
</pre>
##### Expected output
Verify that the virtual service has been implemented as expected:
<pre>kubectl describe virtualservice review
Name:         reviews
...
Spec:
  Hosts:
    reviews
  Http:
    Route:
      Destination:
        Host:    reviews
        Subset:  v1
      Weight:    20
      Destination:
        Host:    reviews
        Subset:  v2
      Weight:    80
</pre>
