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

## Rest API example: Traffic shifting: 20% v1 - 80% v2 with API
<pre>curl -H "Accept: application/json" -H "Content-Type: application/merge-patch+json" -X PATCH http://localhost:8001/apis/networking.istio.io/v1alpha3/namespaces/default/virtualservices/reviews -d '{"metadata":{"annotations":{"kubectl.kubernetes.io/last-applied-configuration":"{\"apiVersion\":\"networking.istio.io/v1alpha3\",\"kind\":\"VirtualService\",\"metadata\":{\"annotations\":{},\"name\":\"reviews\",\"namespace\":\"default\"},\"spec\":{\"hosts\":[\"reviews\"],\"http\":[{\"route\":[{\"destination\":{\"host\":\"reviews\",\"subset\":\"v1\"},\"weight\":20},{\"destination\":{\"host\":\"reviews\",\"subset\":\"v2\"},\"weight\":80}]}]}}\n"}},"spec":{"http":[{"route":[{"destination":{"host":"reviews","subset":"v1"},"weight":20},{"destination":{"host":"reviews","subset":"v2"},"weight":80}]}]}}'
</pre>
