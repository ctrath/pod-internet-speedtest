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