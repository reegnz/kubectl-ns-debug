# kubectl ns-debug

Troubleshoot a pod with a bring-your-own debug image.

:warning: If you have [ephemeral
containers](https://kubernetes.io/docs/concepts/workloads/pods/ephemeral-containers/)
enabled in your cluster, use [kubectl
debug](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-running-pod/#ephemeral-container)
instead!


:exclamation: Running pods with hostPID and hostNetwork are generally discouraged
and you need to understand the risks.

:exclamation: Only use this plugin if you understand what it is doing:
* spawning a container with hostPID and hostNetwork on the kubelet node
* having root access to the entire kubelet node
* being able to run anything on the kubelet node

For detailed explanation see [EXPLAINED.md](./EXPLAINED.md)

## Usage

```sh
kubectl ns-debug -n example my-pod
```

## Installation

Using [krew](https://krew.sigs.k8s.io/): coming soon

Using curl:

```sh
curl -LO https://github.com/reegnz/kubectl-ns-debug/raw/master/kubectl-ns_debug
chmod +x ./kubectl-ns_debug
sudo mv ./kubectl-ns_debug /usr/local/bin/kubectl-ns_debug
```


## Usage examples

* debugging java applications running on a JRE
  * `kubectl ns-debug --image openjdk:11-jdk -n example java-pod`
