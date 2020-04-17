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
<h3 id="issue">1.  <em>Issue:</em></h3>
<p>No Response From Remote Kubectl</p>
<p><code>The connection to the server localhost:8080 was refused — did you specify the right host or port?</code></p>
<h4 id="rootcause">Rootcause:</h4>
<p>Most Kubernetes admins have seen this error at some point. It occurs when either the Kubernetes API is not reachable on the specified URL or there is an issue with the Kubernetes API.</p>
<p>One of the most common issues is the failure of the  <code>etcd</code>  service on one or more master nodes.</p>
<p>You can check the service status via:</p>
<pre><code>service etcd status
</code></pre>
<p>If the service reports  <em>dead</em>, your issue is there.</p>
<h4 id="solution">Solution:</h4>
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
<h3 id="issue-1">2.Issue:</h3>

