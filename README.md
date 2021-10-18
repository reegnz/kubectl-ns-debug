# kubectl ns-debug

Debug a container with a bring-your-own tooling image.

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
