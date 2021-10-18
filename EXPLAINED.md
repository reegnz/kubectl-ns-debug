# Debugging kubernetes pods

It is good practice to bundle the bare minimum with your container images to
reduce image size and also attack vectors within your container.

If you package only the bare minimum in the container, how do you debug the
processes if something does go wrong eventually?

## Finding out the container we need to debug

Each kubernetes pod has metadata on the containers it is running.

```sh
kubectl get pod echo-server-5f7d86dd5f-dlkb6 -o json |
  jq '{node: .spec.nodeName, containers: (.status.containerStatuses|map({name,containerID}))}'
```

```json
{
  "node": "node-1c48c636-159a-47ec-b8f7-6c6f5e2c0d7a",
  "containers": [
    {
      "name": "echo-server",
      "containerID": "containerd://3f84398d46fa22d33da25ceba52872c94aca84cddff722a90ecddeabe0e02658"
    }
  ]
}
```

The containerID for each container can be looked up from the pod status. We now need to enter
the node the container is running on and find it's PID.



## nsenter-ing a node

There's an easy way of doing this with
[kubectl-node-shell](https://github.com/kvaps/kubectl-node-shell), but we're
going to take the hard route, to see how that tool operates.

We are going to use
[nsenter](https://man7.org/linux/man-pages/man1/nsenter.1.html) inside a
privileged pod to gain access to the kubelet host and figure out the container
PID.

```sh
image=ubuntu:20.04
node=node-1c48c636-159a-47ec-b8f7-6c6f5e2c0d7a
overrides="$(
jq --null-input \
  --arg image "$image" \
  --arg node "$node" \
'
{
  spec: {
    containers: [{
      image: $image,
      name: "debug",
      securityContext: {
        privileged: true,
      },
      stdin: true,
      stdinOnce: true,
      tty: true,
    }],
    hostPID: true,
    hostNetwork: true,
    nodeName: $node,
    restart: "Never",
  }
}
'
)"

kubectl run debug-it -it --rm --image $image --overrides="$overrides"
```

The above snippet spawns a new pod with hostpid and hostnet, so we can see into
the kubelet node.

## Getting the PID

With crictl:

```sh
nsenter -t 1 -m -u -i -n -p -- crictl inspect -o go-template --template '{{.info.pid}}' "${container_id#containerd://}"
```

If you don't have crictl on the host and are running docker (deprecated):

```sh
nsenter -t 1 -m -u -u -n -p -- docker inspect -f '{{.State.Pid}}' ${container_id#docker://}")"
```

Once you figured out the pid, it's time to get down to business.


## Commands to run in the container

One pitfall with nsenter when not using the mount namespace is that the /proc fs won't be showing
the correct process tree and ps and other tools utilizing the /proc fs won't work.
One way to get around that is to use [unshare](https://man7.org/linux/man-pages/man1/unshare.1.html) and
then remount the /proc fs once we've done an nsenter.

```sh
nsenter --target $TARGET_PID -u -i -n -p -C -- unshare -m sh -c 'mount -t proc procfs /proc/; exec bash'
```

Now you have a shell that is in the same pid, net ipc and uts namespace with the target container, but has
the mount of another container.
