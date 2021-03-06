## Creating custom Preflight checks

As mentioned in the previous step, `Preflight` checks can be customized to suit your organizational needs by writing custom checks in `yaml` format. Let's practice by creating our own `preflight.yaml` to suit the Katacoda environment.

On the `controlplane`, create a new file named `preflight.yaml` with the following content:

````
apiVersion: troubleshoot.sh/v1beta2
kind: Preflight
metadata:
  name: preflight-tutorial
spec:
  analyzers: []
````

This is the starter framework of our check. You can run the check as-is with the command `kubectl preflight ./preflight.yaml`{{execute}} however this won't return much useful info as we have not specified any `analyzers`. 

To begin, let's edit `preflight.yaml` to add an `analyzer` under `spec:` to verify that Kubernetes is up to date:

````
apiVersion: troubleshoot.sh/v1beta2
kind: Preflight
metadata:
  name: preflight-tutorial
spec:
  analyzers:
    - clusterVersion:
        outcomes:
          - fail:
              when: "< 1.16.0"
              message: The application requires at least Kubernetes 1.16.0, and recommends 1.18.0.
              uri: https://kubernetes.io
          - warn:
              when: "< 1.18.0"
              message: Your cluster meets the minimum version of Kubernetes, but we recommend you update to 1.18.0 or later.
              uri: https://kubernetes.io
          - pass:
              message: Your cluster meets the recommended and required versions of Kubernetes.
````

First, we specify the `analyzer` type, in this case it is `ClusterVersion`, which will be evaluated by the `preflight` checks once we run the updated `yaml`. In the following lines, we define `pass`, `fail`, and `warn` parameters for our check.

Let's take our first check a step further by adding an additional  `analyzer` for `nodeResources`. 

Edit `preflight.yaml` with the following:

````
    - nodeResources:
        checkName: Must have 1 node with 4 GB (available) memory and 2 cores (on a single node)
        filters:
          allocatableMemory: 4Gi
          cpuCapacity: "2"
        outcomes:
          - fail:
              when: "count() < 1"
              message: This application requires at least 1 node with 4GB available memory and 2 cpu cores
          - pass:
              message: This cluster has a node with enough memory and cpu cores
````

The above `analyzer` will verify that the Katacoda environment we are working in has deployed a single worker node with at least 4GB of RAM & 2 CPU cores available.

Now, let's save & close the file, then see the results of our check by running the command:

`kubectl preflight ./preflight.yaml`{{execute}}

Want more?

There are `analyzers` available for everything ranging from K8s node count to `Ceph` storage status. You can review available `analyzers` at `https://troubleshoot.sh/docs/analyze/`{{open}}