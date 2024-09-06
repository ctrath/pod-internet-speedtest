# pod-internet-speedtest

Deploys a daemonset with host networking on each node with the Ookla speedtest utility

## Deploy
Simply run:
`kubectl apply -f speedtestds.yaml`

## Running
It takes a couple minutes to install the necessary packages for the speedtest app.  Pod status
may show `Running`, but the speedtest app may not be installed yet.

#### To check if the speedtest app is installed:
- run `kubectl get po -l name=speedtest-ds -o wide`
  This will display all the speedtest pods.  There will be one per each node.
- Copy the name of a speedtest pod and run the following: `kubectl logs <podname>` where <podname> is substituted with the actual name like:
  `speedtest-ds-r5sp7`
- If the speedtest app has been deployed, you will see this line in the log `<<<<< READY!!! >>>>>`
- If not, continue to get the pod logs until that line appears

#### Executing speedtest
Once the speedtest app is installed in a give pod:
- run `kubectl get po -l name=speedtest-ds -o wide`
  This will display all the speedtest pods.  There will be one per each node.
- for each pod run `kubectl exec <podname> -- speedtest --accept-license`

Output should be similar to the following:
```
❯ kubectl exec speedtest-ds-r5sp7 -- speedtest --accept-license

   Speedtest by Ookla

      Server: KamaTera, Inc. - Cheney, KS (id: 64135)
         ISP: SoftLayer Technologies
Idle Latency:   138.72 ms   (jitter: 0.12ms, low: 138.60ms, high: 138.82ms)
    Download:  6193.98 Mbps (data used: 9.4 GB)
                217.28 ms   (jitter: 65.42ms, low: 143.38ms, high: 288.90ms)
      Upload:    73.08 Mbps (data used: 128.3 MB)
                214.31 ms   (jitter: 64.75ms, low: 142.09ms, high: 289.62ms)
 Packet Loss:     0.0%
  Result URL: https://www.speedtest.net/result/c/4991ef8f-555a-4e1d-adb7-153aa99aa302
  ```

  Results should be similar for each node.  If not, there is likely an issue with the node or network in which the test was run.


  # Running local iperf3 bandwidth test
  If you don't have internet connectivity to publish registry, you will need to copy the iperf3 container image to the IBM private registry or other private registry your cluster has access to.

  #### Copying iamge to IBM Cloud registry
  - On your local PC, with either docker or podman present, pull the iperf3 container image to your local registry: `podman pull networkstatic/iperf3:latest --platform=linux/amd64`
  - Make a new tag of the image with the host and namespace of the repository you will be pushing to: `podman tag docker.io/networkstatic/iperf3:latest au.icr.io/cale-speedtest/cale-iperf3:latest`
    - In the above example, I am planning to push to the IBM container registry in "au.icr.io" in namespace "cale-speedtest" with image name "cale-iperf3".  Change the registry host, namespace, image name and tag to whatever matches your registry.
  - If pushing to the IBM container registry, login to the registry from your PC:
  ```
  > ibmcloud login
  > ibmcloud cr login --podman #note: if using docker, change podman to docker
  ```
  - If you don't have a registry namespace in your remote registry or want a new namespace for the iperf3 image run: `ibmcloud cr namespace-add cale-speedtest`.  Replace `cale-speedtest` with whatever you want to name your namespace.
  - Push the image to the remote registry: `podman push au.icr.io/cale-speedtest/cale-iperf3:latest` changing the host, namnespace, image and tagname accordingly.
  - Check to make sure the image is present in the remote registry.  For the IBM registry run: `ibmcloud cr image-list`

  #### Deploying the iperf3 daemonset
  - First open localiperf.yaml and edit the `image:` line to reference the image you pushed to your private registry.
  - Deploy the daemonset: `kubectl apply -f localiperf.yaml`
  - Wait for the pods to start `kubectl get po -l name=localiperf-ds`
    - There should be one pod per node.  Wait until all pods are in "Running" status.

  #### Running the iperf3 test
  - Get a list of all the iperf3 daemonset pods and the associated node in which they reside: `kubectl get po -l name=localiperf-ds -o wide`
    -  Output should look like:
```
❯  kubectl get po -l name=localiperf-ds -o wide
NAME                  READY   STATUS    RESTARTS   AGE   IP             NODE           NOMINATED NODE   READINESS GATES
localiperf-ds-kjnvl   1/1     Running   0          22m   10.245.128.4   10.245.128.4   <none>           <none>
localiperf-ds-p6njb   1/1     Running   0          22m   10.245.128.5   10.245.128.5   <none>           <none>
localiperf-ds-vrdgl   1/1     Running   0          22m   10.245.128.6   10.245.128.6   <none>           <none>
```
  - To test bandwidth between to nodes, run `kubectl exec localiperf-ds-kjnvl -- iperf3 -c 10.245.128.5` where `localiperf-ds-kjnvl` is the pods name for node `10.245.128.4` and `10.245.128.5` is the iperf3 server to connect to.
    - Note: these pods use host networking, so this is testing the networking bandwidth between the hosts themselves without the pod network.