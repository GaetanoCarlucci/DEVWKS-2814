# DEVWKS-2814
<br>

**Session Code:**  DEVWKS-2814 <br>
**WebEX Room:** http://cs.co/eventsbot#DEVWKS-2814 <br>
<br>

**Title:** DevNet Workshop: Let's Play with Istio: service meshes for high-scale container environments<br>
<br>
**Abstract:**<br>
Istio is an open source service mesh which is also packaged and supported in the Cisco Container Platform (CCP). With Istio, you can securely connect, control, and observe microservices on large scale deployments. The goal of this workshop is to give you the opportunity to use an Istio-enabled Kubernetes cluster which can be created with CCP and get familiar with some use cases like service routing, traffic shifting and service tracing. This will be done both by using manifest files manipulation and REST API calls. <br><br>
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

## Bookinfo App: service mesh topology with Istio

  ![Sandbox layout](https://github.com/GaetanoCarlucci/DEVWKS-2814/blob/master/bookinfo_mesh_topology.PNG)


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

### Bookinfo Application without Istio
Bookinfo Application is a virtual library for books description and ratings. It is a webpage that shows the book details, reviews, and ratings from readers. <br>
It consists of 4 four separate microservices (product page, detail, review, rating) written in different program languages. This is the value of microservice having each microservice completely independent from each other.<br>
Each box in the picture is a Kubernetes deployment with a Kubernetes service attached to it. Each deployment has one pod with one container inside.<br>
The review is divided into three deployments each one with a different version. One that does not have a connection to the rating service, one that shows the ratings in black and one in red.<br>
By default, without Istio the product page accesses to the review versions in a round-robin fashion.<br>

<pre>cd /home/developer/istio-1.2.2/samples/bookinfo/platform/kube </pre>
<pre>cat bookinfo.yaml </pre>

### Bookinfo Application with Istio
When we add Istio to our application service mesh we add one sidecar container to our pods that intercept the requests coming in and out the pod. <br>
In this way, all traffic that the mesh services send and receive (data plane traffic) is proxied through Envoy, making it easy to direct and control traffic around the service mesh without making any changes to your services.<br>
Each pod in the review service has a Kubernetes label that defines the version of the app. This is important for Istio to route traffic based on that version.<br>
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
<pre>cd /home/developer/istio-1.2.2/samples/networking/ </pre>
<pre>kubectl apply -f bookinfo-gateway.yaml </pre>
Retrive external IP:
<pre>kubectl get svc -n istio-system | grep ingress </pre>
##### Expected output

### Bookinfo Application: try it out 
http://10.10.20.30/productpage 

### Route based on application version: DestinationRule
<pre>cd /home/developer/istio-1.2.2/samples/networking/ </pre>
<pre>kubectl apply -f destination-rule-all.yaml </pre>
##### Expected output

### Route based on application version: VirtualService
<pre>cd /home/developer/istio-1.2.2/samples/networking/ </pre>
<pre>kubectl apply -f virtual-service-all-v1.yaml </pre>
##### Expected output
<pre>$kubectl describe virtualservice review</pre>

### Route based on user identity
<pre>cd /home/developer/istio-1.2.2/samples/networking/ </pre>
Modify **virtual-service-reviews-jason-v2-v3.yaml** by inserting your name and apply it.
<pre>kubectl apply -f virtual-service-reviews-jason-v2-v3.yaml </pre>
##### Expected output
Verify that the virtual service has been implemented as expected:
<pre>kubectl describe virtualservice review</pre>
### Traffic shifting: 80% v1 - 20% v2
<pre>cd /home/developer/istio-1.2.2/samples/networking/ </pre>
<pre>kubectl apply -f virtual-service-reviews-80-20.yaml </pre>
##### Expected output
Verify that the virtual service has been implemented as expected:
<pre>kubectl describe virtualservice review</pre>
### Rest API example: Traffic shifting: 20% v1 - 80% v2 with API
<pre>curl -H "Accept: application/json" -H "Content-Type: application/merge-patch+json" -X PATCH http://localhost:8001/apis/networking.istio.io/v1alpha3/namespaces/default/virtualservices/reviews -d '{"metadata":{"annotations":{"kubectl.kubernetes.io/last-applied-configuration":"{\"apiVersion\":\"networking.istio.io/v1alpha3\",\"kind\":\"VirtualService\",\"metadata\":{\"annotations\":{},\"name\":\"reviews\",\"namespace\":\"default\"},\"spec\":{\"hosts\":[\"reviews\"],\"http\":[{\"route\":[{\"destination\":{\"host\":\"reviews\",\"subset\":\"v1\"},\"weight\":20},{\"destination\":{\"host\":\"reviews\",\"subset\":\"v2\"},\"weight\":80}]}]}}\n"}},"spec":{"http":[{"route":[{"destination":{"host":"reviews","subset":"v1"},"weight":20},{"destination":{"host":"reviews","subset":"v2"},"weight":80}]}]}}'
</pre>
##### Expected output
Verify that the virtual service has been implemented as expected:
<pre>$kubectl describe virtualservice review</pre>

