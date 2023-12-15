title:: Debugging Your Kubernetes Nodes in the ‘Not Ready’ State | Nodenotready (highlights)
author:: [[Aniket Bhattacharyea]]
full-title:: "Debugging Your Kubernetes Nodes in the ‘Not Ready’ State | Nodenotready"
category:: #articles
url:: https://www.containiq.com/post/debugging-kubernetes-nodes-in-not-ready-state
![](https://assets-global.website-files.com/5fbfbba70f3f813561ef7b9f/6230b7a88ef3f90580729512_1.35.png)

- Highlights first synced by [[Readwise]] [[Dec 14th, 2023]]
	- Check the kube-proxy Pod
	  
	  First, ensure that each node has exactly one kube-proxy pod and is in the Running state.
	  
	    
	    kubectl get pods -n kube-system -o wide
	    
	  
	  The output might look like this:  
	  ‍
	  
	  NAME
	  
	  READY
	  
	  STATUS
	  
	  AGE
	  
	  IP
	  
	  NODE
	  
	  NOMINATED NODE
	  
	  READINESS GATES
	  
	  kube-proxy-nhbtp
	  
	  1/1
	  
	  Running
	  
	  2 (11h ago)
	  
	  2d16h
	  
	  192.168.99.10
	  
	  1 my-cluster
	  
	  <none>
	  
	  <none>
	  
	  kube-proxy-tkmsk
	  
	  1/1
	  
	  Running
	  
	  2 (11h ago)
	  
	  2d16h
	  
	  192.168.99.10
	  
	  3 my-cluster-m03
	  
	  <none>
	  
	  <none>
	  
	  kube-proxy-vk4ch
	  
	  1/1
	  
	  Running
	  
	  2 (11h ago)
	  
	  2d16h
	  
	  192.168.99.10
	  
	  2 my-cluster-m02
	  
	  <none>
	  
	  <none>
	  
	  If any one pod is in some state other than Running, use the following command to get more information:
	  
	    
	    kubectl describe pod yourPodName -n kube-system
	    
	  
	  The [Events](https://www.containiq.com/post/kubernetes-events) section logs the various events on the pod, and it could be an excellent place to start looking for any mishaps.
	  
	  ![](https://assets-global.website-files.com/5fbfbba70f3f813561ef7b9f/6195a6dc40b4fc35a1b9d888_OfXSfs0.png)
	  
	  The events section in the output
	  
	  You can get access to the pod logs by running the following command:
	  
	    
	    kubectl logs yourPodName -n kube-system
	    
	  
	  The [logs](https://www.containiq.com/post/kubernetes-logging) and the events list is a good place to start looking for any issues.
	  
	  If your node does not have a kube-proxy pod, then you need to inspect the kube-proxy daemonset, which is responsible for running one kube-proxy pod on each node.
	  
	    
	    kubectl describe daemonset kube-proxy -n kube-system
	    
	  
	  The output of this command might reveal any possible issue with the daemonset. ([View Highlight](https://read.readwise.io/read/01hhcmh5av4ga3e74kk6ymqj6v))
	- Verify kubelet is Running
	  
	  If all the Conditions fields show Unknown, it might hint that the kubelet process on the node has run into some issues.
	  
	  ![](https://assets-global.website-files.com/5fbfbba70f3f813561ef7b9f/6195a8115232296e32106ceb_LLzhWcQ.png)
	  
	  The conditions field shows unknown
	  
	  To debug this, first SSH into the node and check the status of the kubelet process. If it’s running as a systemd service, use the following command:
	  
	    
	    systemctl status kubelet
	    
	  
	  If the Active field shows inactive (dead), it means the kubelet process has stopped.
	  
	  ![](https://assets-global.website-files.com/5fbfbba70f3f813561ef7b9f/6195a84d23c75c183aeb445e_BFAcS0l.png)
	  
	  The active field of the output  
	  ‍
	  
	  To reveal the possible reason for the crash, check the logs with the following command:
	  
	    
	    journalctl -u kubelet
	    
	  
	  Once the issue is fixed, restart kubelet with:
	  
	    
	    systemctl restart kubelet ([View Highlight](https://read.readwise.io/read/01hhcmj9q2nc575sp1qs3t980w))
	- Verify Network Communication with the Control Plane
	  
	  If the Conditions field shows NetworkUnavailable, it indicates an issue in the network communication between the node and the control plane.
	  
	  A few possible fixes:
	  
	  •   If the node is configured to use a proxy, verify that the proxy allows access to the API server endpoints.
	  •   Ensure that the route tables are appropriately configured to avoid blocking communication with the API server.
	  •   If you’re using a cloud provider like AWS, verify that no VPC network rules block communication between the control plane and the node.
	  
	  You can run the following command from within the node to verify that it can reach the API server.
	  
	    
	    nc -vz <your-api-server-endpoint> 443
	    
	  
	  If the output shows succeeded, then network communication is working correctly. ([View Highlight](https://read.readwise.io/read/01hhcmkjzpfnedgr5dqy74ybq6))