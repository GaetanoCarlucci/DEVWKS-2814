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
<pre>$kubectl get nodes -o wide</pre>

### Verify Istio Control Plane Installation
<pre>$kubectl get pods -n istio-system</pre>

### Bookinfo Application without Istio
<pre>$cd /home/developer/istio-1.2.2/samples/bookinfo/platform/kube </pre>
<pre>$cat bookinfo.yaml </pre>

### Bookinfo Application with Istio
<pre>$istioctl kube-inject -f bookinfo.yaml > bookinfo_with_istio.yaml </pre>
<pre>$cat bookinfo_with_istio.yaml </pre>
<pre>$kubectl apply –f bookinfo_with_istio.yaml  </pre>
<pre>$kubectl get pods -o wide </pre>

### Bookinfo Application add ingress gateway
<pre>$cd /home/developer/istio-1.2.2/samples/networking/ </pre>
<pre>$kubectl apply –f bookinfo-gateway.yaml </pre>

### Bookinfo Application: retrive external IP
<pre>$kubectl get svc -n istio-system | grep ingress </pre>

### Bookinfo Application: try it out 
http://10.10.20.30/productpage 

### Route based on application version: DestinationRule
<pre>$cd /home/developer/istio-1.2.2/samples/networking/ </pre>
<pre>$kubectl apply –f destination-rule-all.yaml </pre>

### Route based on application version: VirtualService
<pre>$cd /home/developer/istio-1.2.2/samples/networking/ </pre>
<pre>$kubectl apply –f virtual-service-all-v1.yaml </pre>

### Route based on user identity
<pre>$cd /home/developer/istio-1.2.2/samples/networking/ </pre>
Modify **virtual-service-reviews-jason-v2-v3.yaml** by inserting your name and apply it.
<pre>$kubectl apply –f virtual-service-reviews-jason-v2-v3.yaml </pre>
Verify that the virtual service has been implemented as expected:
<pre>$kubectl describe virtualservice review</pre>
### Traffic shifting: 80% v1 - 20% v2
<pre>$cd /home/developer/istio-1.2.2/samples/networking/ </pre>
<pre>$kubectl apply –f virtual-service-reviews-80-20.yaml </pre>
Verify that the virtual service has been implemented as expected:
<pre>$kubectl describe virtualservice review</pre>
### Rest API example: Traffic shifting: 20% v1 - 80% v2 with API
<pre>curl -H "Accept: application/json" -H "Content-Type: application/merge-patch+json" -X PATCH http://localhost:8001/apis/networking.istio.io/v1alpha3/namespaces/default/virtualservices/reviews -d '{"metadata":{"annotations":{"kubectl.kubernetes.io/last-applied-configuration":"{\"apiVersion\":\"networking.istio.io/v1alpha3\",\"kind\":\"VirtualService\",\"metadata\":{\"annotations\":{},\"name\":\"reviews\",\"namespace\":\"default\"},\"spec\":{\"hosts\":[\"reviews\"],\"http\":[{\"route\":[{\"destination\":{\"host\":\"reviews\",\"subset\":\"v1\"},\"weight\":20},{\"destination\":{\"host\":\"reviews\",\"subset\":\"v2\"},\"weight\":80}]}]}}\n"}},"spec":{"http":[{"route":[{"destination":{"host":"reviews","subset":"v1"},"weight":20},{"destination":{"host":"reviews","subset":"v2"},"weight":80}]}]}}'
</pre>

Verify that the virtual service has been implemented as expected:
<pre>$kubectl describe virtualservice review</pre>

