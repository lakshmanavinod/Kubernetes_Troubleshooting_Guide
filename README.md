---


---

<h1 id="troubleshooting-guide-for-kubernetes-cluster">Troubleshooting Guide for Kubernetes Cluster</h1>
<h2 id="useful-links">Useful Links:</h2>
<p><strong>To troubleshoot the kubernetes cluster:</strong><br>
<a href="https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/">https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/</a></p>
<p><strong>To troubleshoot the applications deployed in kubernetes:</strong><br>
<a href="https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/">https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/</a></p>
<p><strong>Stackoverflow tagged on Kubernetes:</strong><br>
<a href="https://stackoverflow.com/questions/tagged/kubernetes">https://stackoverflow.com/questions/tagged/kubernetes</a></p>
<p><strong>Kubernetes official slack community:</strong><br>
<a href="https://kubernetes.slack.com/">https://kubernetes.slack.com/</a></p>
<p><strong>Kubernetes official discussion form:</strong><br>
<a href="https://discuss.kubernetes.io/">https://discuss.kubernetes.io/</a></p>
<p><strong>Github Kubernetes Issues:</strong><br>
<a href="https://github.com/kubernetes/kubernetes/issues">https://github.com/kubernetes/kubernetes/issues</a></p>
<p><strong>Troubleshooting One-Stop-Shop:</strong><br>
<a href="https://kubernetes.io/docs/tasks/debug-application-cluster/troubleshooting/">https://kubernetes.io/docs/tasks/debug-application-cluster/troubleshooting/</a></p>
<h2 id="use-cases-for-frequently-occuring-kubernetes-issues">Use-Cases for frequently occuring Kubernetes Issues</h2>
<h3 id="issue--no-response-from-remote-kubectl">1.  <em>Issue:</em>  <em><strong>No Response From Remote Kubectl</strong></em></h3>
<p><code>The connection to the server localhost:8080 was refused — did you specify the right host or port?</code></p>
<h4 id="rootcause"><em>Rootcause:</em></h4>
<p>Most Kubernetes admins have seen this error at some point. It occurs when either the Kubernetes API is not reachable on the specified URL or there is an issue with the Kubernetes API.</p>
<p>One of the most common issues is the failure of the  <code>etcd</code>  service on one or more master nodes.</p>
<p>You can check the service status via:</p>
<pre><code>service etcd status
</code></pre>
<p>If the service reports  <em>dead</em>, your issue is there.</p>
<h4 id="solution"><em>Solution:</em></h4>
<p>First, let’s make sure that <a href="https://kubernetes.io/docs/tasks/tools/install-kubectl/">kubectl</a> is installed and in the correct version</p>
<pre><code>kubectl version
</code></pre>
<p>If there are any other errors (command not found, etc):</p>
<ul>
<li>Ensure that you are running kubectl via a user that has access to perform kubectl commands.</li>
<li>Ensure that the kubectl paths are set up properly.</li>
<li>kubectl will not work with the “root” account by default.</li>
</ul>
<p>If kubectl itself is working as it should, we can move on to troubleshooting the cluster.</p>
<p><code>ssh</code>  to one of your Kubernetes master Leader  or Control plane end point :</p>
<p>If issue is with etcd service, then</p>
<p>One of the most common reasons for the failure of the <code>etcd</code> service (especially after master node reboot) is that swap is enabled on the node. This is particularly true in <a href="https://kubernetes.io/docs/reference/setup-tools/kubeadm/">kubeadm</a>-managed clusters.</p>
<p>As root, run:</p>
<pre><code>swapoff -ased -i '/ swap / s/^/#/' /etc/fstab
</code></pre>
<p>Then, disable swap from starting at boot:</p>
<pre><code>vi /etc/fstab # Comment out the second line mentioning swap  
# Save the file
</code></pre>
<p>Swap may also affect the  <code>[kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/)</code>  service.</p>
<p>You can check the service status via:</p>
<pre><code>service kubelet status
</code></pre>
<p>You can also find the logs for both services in  <code>/var/log</code>  for further troubleshooting.</p>
<p>Run these checks on the entire  <code>etcd</code>  cluster (all master nodes).</p>
<p>You should now have a healthy cluster and kubectl should work.</p>
<h3 id="issue-frequently-occuring-errors-such-as-imagepullbackoffcrashloopbackoff-and-runcontainererror">2.<em><strong>Issue: Frequently occuring errors such as ImagePullBackOff,CrashLoopBackOff and RunContainerError</strong></em></h3>
<h4 id="a.-imagepullbackoff">2.a. <em><strong>ImagePullBackOff</strong></em></h4>
<pre class=" language-bash"><code class="prism  language-bash">kubectl get pods
NAME                    READY     STATUS             RESTARTS   AGE
fail-1036623984   0/1       ImagePullBackOff   0          2m
</code></pre>
<p>For some additional information, we can  <code>describe</code>  the failing Pod:</p>
<pre class=" language-bash"><code class="prism  language-bash">
$ kubectl describe pod fail-1036623984
</code></pre>
<p>Please check the <code>Events</code> section for more details on the error.</p>
<p><em><strong>Rootcause:</strong></em></p>
<p>Most common problems are (a) having the wrong container image/tag specified and (b) trying to use private images without providing registry credentials and © kubernetes doesn’t have permissions to pull the image in private registry and (d) image is not existing in the container registry specified.</p>
<p><em><strong>Solution:</strong></em></p>
<p>From a local machine , try to pull the same image specified in deployment.yml ,</p>
<p>Example:</p>
<pre><code>docker pull jenkins/jenkins:2.108.2
</code></pre>
<p>If tag is a culprit, try to pull the image without specifying the tag, so that latest gets pulled.</p>
<pre><code>docker pull jenkins/jenkins
</code></pre>
<p>By default, Kubernetes uses the <a href="https://hub.docker.com/">Dockerhub</a> registry. If you’re using <a href="https://kukulinski.com/10-most-common-reasons-kubernetes-deployments-fail-part-1/quay.io">Quay.io</a>, <a href="https://aws.amazon.com/ecr/">AWS ECR</a>, or <a href="https://cloud.google.com/container-registry/">Google Container Registry</a>, you’ll need to specify the registry URL in the image string. For example, on Quay, the image would be [<a href="http://quay.io/jenkins/jenkins:2.108.2">quay.io/jenkins/jenkins:2.108.2</a>]</p>
<h4 id="b-crashloopbackoff">2.b <em><strong>CrashLoopBackOff</strong></em></h4>
<pre class=" language-bash"><code class="prism  language-bash">$ kubectl get pods
NAME                       READY     STATUS             RESTARTS   AGE
crasher-2443551393-vuehs   0/1       CrashLoopBackOff   2          54s
</code></pre>
<p>Ok, so <code>CrashLoopBackOff</code> tells us that Kuberenetes is trying to launch this Pod, but one or more of the containers is crashing or getting killed.</p>
<p><em><strong><strong>Rootcause:</strong></strong></em></p>
<p>Let’s <code>describe</code> the pod to get some more information:<br>
Sample snippet below.</p>
<pre class=" language-bash"><code class="prism  language-bash">kubectl describe pod crasher-2443551393

Containers:
  crasher:
    Container ID:    docker://51c940ab32016e6d6b5ed28075357661fef3282cb3569117b0f815a199d01c60
    Image:        jenkins/crashing-app
    Image ID:        docker://sha256:cf7452191b34d7797a07403d47a1ccf5254741d4bb356577b8a5de40864653a5
    Port:        
    State:        Terminated
      Reason:        Error
      Exit Code:    1
</code></pre>
<p>We see that this Pod is being <code>Terminated</code> due to the application inside the container crashing with <code>Exit Code: 1</code></p>
<p><em><strong>Solution:</strong></em><br>
Assuming you are sending your application logs to <code>stdout</code> , you can see the application logs using <code>kubectl logs</code></p>
<pre class=" language-bash"><code class="prism  language-bash">kubectl logs crasher-2443551393
</code></pre>
<p>If no valid output to debug it further, add --previous option to the command.</p>
<pre><code>kubectl logs crasher-2443551393 --previous
</code></pre>
<p>Incase , if the error is related to OOM failures , Please increase the cpu and memory limits in the container specifications.</p>
<h4 id="c-runcontainererror">2.c <em><strong>RunContainerError</strong></em></h4>
<pre class=" language-bash"><code class="prism  language-bash">kubectl get pods
NAME            READY     STATUS              RESTARTS   AGE
configmap-pod   0/1       RunContainerError   0          3s
</code></pre>
<p><em><strong>Rootcause:</strong></em></p>
<p>This may occur due to missing ConfigMap and Secret which is used by POD as environment variables.</p>
<pre class=" language-bash"><code class="prism  language-bash">
$ kubectl describe pod configmap-pod
</code></pre>
<p>Please check the <code>Events</code> section for more details on the error.</p>
<p><em><strong>Solution:</strong></em></p>
<p>Once we create the ConfigMap or Secret the Pod should restart and pull in the runtime data.</p>
<h3 id="issue-eviction-of-pods">3. <em><strong>Issue: Eviction of pods</strong></em></h3>
<pre class=" language-bash"><code class="prism  language-bash">NAME       READY   STATUS    RESTARTS   AGE
frontend   0/2     Evicted   0          10s
</code></pre>
<p><em><strong>Rootcause:</strong></em></p>
<p>When a node in a Kubernetes cluster is running out of memory or disk, it activates a flag signaling that it is under pressure. This blocks any new allocation in the node and starts the eviction process.</p>
<p>Hence,kubelet starts to reclaim resources, killing containers and declaring pods as <em>failed</em> until the resource usage is under the eviction threshold again.</p>
<p>First, kubelet tries to free node resources, especially disk, by deleting dead pods and its containers, and then unused images. If this isn’t enough, kubelet starts to evict end-user pods in the following order:</p>
<ul>
<li>Best Effort.</li>
<li>Burstable pods using more resources than its request of the starved resource.</li>
<li>Burstable pods using less resources than its<br>
request of the starved resource.</li>
</ul>
<p><em><strong><strong><strong>Solution:</strong></strong></strong></em><br>
If you are not setting requests and limits in your pods, this is a very good reason to do so. Setting those values properly can protect you from unexpected outages and the probability of eviction reduces.</p>
<h3 id="issue--exceeding-quota-and-insufficient-cpu"><em><strong>4.Issue:  Exceeding Quota and Insufficient CPU</strong></em></h3>
<h4 id="a-exceeding-quota">4.a Exceeding Quota</h4>
<p><strong>Error</strong> creating: pods “gateway-quota-551394438-” is forbidden: <strong>exceeded quota:</strong> compute-resources, requested: pods=1, used: pods=1, limited: pods=1</p>
<pre class=" language-bash"><code class="prism  language-bash">Events:
  FirstSeen    LastSeen    Count   From                SubObjectPath   Type        Reason          Message
  ---------    --------    -----   ----                -------------   --------    ------          -------
  11m        30s     33  <span class="token punctuation">{</span>replicaset-controller <span class="token punctuation">}</span>            Warning     FailedCreate        Error creating: pods <span class="token string">"gateway-quota-551394438-"</span> is forbidden: exceeded quota: compute-resources, requested: pods<span class="token operator">=</span>1, used: pods<span class="token operator">=</span>1, limited: pods<span class="token operator">=</span>1
</code></pre>
<p><em><strong>Rootcause:</strong></em></p>
<p>Similar to resource limits, Kubernetes also allows admins to set <a href="https://kubernetes.io/docs/admin/resourcequota/">Resource Quotas</a> per namespace. These quotas can set soft &amp; hard limits on resources such as number of Pods, Deployments, PersistentVolumes, CPUs, Memory, and more which makes the Pods fail.</p>
<p><em><strong>Solution:</strong></em></p>
<ol>
<li>Request your cluster admin to increase the Quota for this namespace</li>
<li>Delete or scale back other Deployments in this namespace.</li>
</ol>
<h4 id="b-insufficientcpu">4.b InsufficientCPU</h4>
<p>Unless your cluster is configured for auto-scaling , you would be end up with insufficient CPU</p>
<p><strong>failed to fit in any node</strong><br>
fit failure on node ( rhel-machine-1): <strong>Insufficient cpu</strong></p>
<pre class=" language-bash"><code class="prism  language-bash">Events:
  FirstSeen    LastSeen    Count   From            SubObjectPath   Type        Reason          Message
  ---------    --------    -----   ----            -------------   --------    ------          -------
  3m        3m      1   <span class="token punctuation">{</span>default-scheduler <span class="token punctuation">}</span>            Warning     FailedScheduling    pod <span class="token punctuation">(</span>cpu-scale-908056305-phb4j<span class="token punctuation">)</span> failed to fit <span class="token keyword">in</span> any node
fit failure on node <span class="token punctuation">(</span>gke-ctm-1-sysdig2-35e99c16-wx0s<span class="token punctuation">)</span>: Insufficient cpu
</code></pre>
<p><em><strong>Rootcause:</strong></em></p>
<p>Cluster Administrators can limit the amount of CPU or memory a developer can request to be allocated to a Pod or container.</p>
<p>Let’s say you have a Kubernetes cluster with 1 Node that has 1 CPU. Your Kubernetes cluster has <code>1000m</code> of available CPU to schedule.</p>
<p>Ignoring other pods, you will be able to deploy 10 Pods (with 1 container each at  <code>100m</code>) to your single-Node cluster.</p>
<p><code>10 Pods * (1 Container * 100m) = 1000m == Cluster CPUs</code></p>
<p>Incase , if an 11th pod is deployed it results in Insufficient CPU.</p>
<p><em><strong>Solution:</strong></em></p>
<ol>
<li>Request your cluster admin to scale up the cluster resources.</li>
<li>If the cluster is hosted on Cloud, It would be good to enable the auto-scaling feature.</li>
</ol>
<h3 id="issue-networking-related-issues"><em>5. Issue: Networking Related Issues:</em></h3>
<h4 id="pod-cidr-conflicts">Pod CIDR Conflicts</h4>
<p>Issue occurs when pod network subnets start conflicting with host networks.</p>
<p>Pod to pod communication is disrupted with routing problems.</p>
<pre class=" language-bash"><code class="prism  language-bash">$ curl http://172.28.128.132:5000
curl: <span class="token punctuation">(</span>7<span class="token punctuation">)</span> Failed to connect to 172.28.128.132 port 5000: No route to host
</code></pre>
<p><em><strong>Rootcause:</strong></em><br>
When Pod IP range matches with cluster IP range.<br>
Check,</p>
<pre><code>kubectl get pods -o wide
</code></pre>
<p>And,</p>
<pre><code>ip addr list
</code></pre>
<p><em><strong>Solution:</strong></em><br>
IP address range could be specified in your <a href="https://kubernetes.io/docs/concepts/cluster-administration/network-plugins/#cni">CNI plugin</a> or <a href="https://kubernetes.io/docs/concepts/cluster-administration/network-plugins/#kubenet">kubenet</a> pod-cidr parameter.</p>
<p>Double-check what RFC1918 private network subnets are in use in your network, VLAN or VPC and make certain that there is no overlap.</p>
<p>Once you detect the overlap, update the Pod CIDR to use a range that avoids the conflict.</p>
<ul>
<li>For more Networking related troubleshooing guide Refer,<br>
<a href="https://gravitational.com/blog/troubleshooting-kubernetes-networking/">https://gravitational.com/blog/troubleshooting-kubernetes-networking/</a></li>
</ul>
<h3 id="issue-what-if-service-ip-range--overlaps-with-pod-ip-range.">6.Issue: What if Service IP range  overlaps with POD IP range.</h3>
<p>In this case, the IP range would be full sooner, if both K8s services and K8s pods assign the IPs from the same subnet range.</p>
<p><em><strong>Rootcause:</strong></em></p>
<p>By default , Kubernetes service IPs are in range 192.168.0.0/24 and pod IPs are in range 192.168.0.0/16.</p>
<p>This limits us to only 255 service IPs</p>
<p>The only reason there’s no clash here is because flannel allocates a /24 to each node, and I think the 192.168.0.0/24 is assigned to one of the proxy nodes (which will never have pods in them).</p>
<p>These should be totally independent network ranges that also must not clash with anything else. We need to reassign these pretty soon.</p>
<p>Note on flannel TTL: <a href="https://github.com/coreos/flannel/blob/master/subnet/local_manager.go#L31">https://github.com/coreos/flannel/blob/master/subnet/local_manager.go#L31</a></p>
<p>flannel docs <a href="https://github.com/coreos/flannel">https://github.com/coreos/flannel</a></p>
<p><em><strong>Resolution:</strong></em></p>
<p>Subnet the existing 192.168.0.0/16 (flannel) – overlaps with Services allocation of 192.168.0.0/24 into two subnets of 192.168.0.0/17 for Services and 192.168.128.0/17 for Flannel</p>
<p>Details steps mentioned here</p>

