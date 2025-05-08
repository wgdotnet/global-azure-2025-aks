---
tags:
  - k8s
---
**Source:** https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/

## Probes

**Source:** https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

### StartUp Probe

**Source:** https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-startup-probes

Sometimes, you have to deal with applications that require additional startup time on their first initialization. In such cases, it can be tricky to set up liveness probe parameters without compromising the fast response to deadlocks that motivated such a probe. The solution is to set up a startup probe with the same command, HTTP or TCP check, with a `failureThreshold * periodSeconds` long enough to cover the worst case startup time.

```yaml
startupProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 30
  periodSeconds: 10
```

Thanks to the startup probe, the application will have a maximum of 5 minutes (30 * 10 = 300s) to finish its startup. Once the startup probe has succeeded once, the liveness probe takes over to provide a fast response to container deadlocks. If the startup probe never succeeds, the container is killed after 300s and subject to the pod's `restartPolicy`.
### Liveness Probe

> ⚠️ **Caution** 
> Failure to report success by the liveness probe will result in pod termination

The [kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/) uses liveness probes to know when to restart a container. For example, liveness probes could catch a deadlock, where an application is running, but unable to make progress. Restarting a container in such a state can help to make the application more available despite bugs.

A common pattern for liveness probes is to use the same low-cost HTTP endpoint as for readiness probes, but with a higher failureThreshold. This ensures that the pod is observed as not-ready for some period of time before it is hard killed.

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 1
  periodSeconds: 10
```

### Readiness Probe

Sometimes, applications are temporarily unable to serve traffic. For example, an application might need to load large data or configuration files during startup, or depend on external services after startup. In such cases, you don't want to kill the application, but you don't want to send it requests either. Kubernetes provides readiness probes to detect and mitigate these situations. A pod with containers reporting that they are not ready does not receive traffic through Kubernetes Services.

```yaml
readinessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
```

## Pod Lifecycle Phases & Key Events

1. **Pending**
	- Pod is accepted by the Kubernetes system but not yet running
	- Waiting for scheduling, resource allocation or image pulling
2. **Container Initialization**
	- Init containers run sequentially
3. **Running**
	- Main containers start
	- **postStart Hook:** if defined, the hook will be executed for each container immediately after the container is started (may run in parallel with container's **ENTRYPOINT**)
	- Probes start
		- **Startup Probe** (one time)
		- **Liveness Probe** (ongoing)
		- **Readiness Probe** (ongoing)
4. **Termination**
	- Pod enters **Terminating** phase
	- **preStop Hook**: if defined the preStop hook is executed for each container before SIGTERM is sent to container's process
	- SIGTERM is sent to each container, allowing for graceful shutdown within `terminationGracePeriodSeconds`
	- SIGKILL is sent to containers failing to terminate gracefully
1. **Terminal States
	- **Succeeded:** All containers exited successfully (exit code 0)
	- **Failed:** At least one container existed with an error
	- **Unknown:** Pod state cannot be determined due to node communication issues

## Handling Client Requests Properly With Kubernetes

**Source:** https://freecontent.manning.com/handling-client-requests-properly-with-kubernetes/

Let’s see what happens at the other end of a pod’s life – when the pod is deleted and its containers are terminated. The pod’s containers should start shutting down cleanly as soon as they receive a SIGTERM signal (or even before that – when its pre-stop hook is executed), but does this ensure all client requests will be handled properly?

How should the app behave when it receives a termination signal? Should it continue accepting requests? What about requests that have already been received, but haven’t completed yet? What about persistent HTTP connections, which may be in between requests, but are open (there’s no active request on the connection)? Before we can answer those questions, we need to take a detailed look at the chain of events that unfolds across the cluster when a pod is deleted.

![[pod-deletion-sequence.png]]

In the A sequence of events, you’ll see that as soon as the Kubelet receives the notification that the pod should be terminated, it initiates the shutdown sequence (run the pre-stop hook, send SIGTERM, wait for some time and then forcibly kill the container if it hasn’t yet terminated on its own). If the app responds to the SIGTERM by immediately ceasing to receive client requests, any client trying to connect to it receives a Connection refused error. The time it takes for this to happen from the time the pod is deleted is relatively short, because of the direct path from the API server to the Kubelet.

Now, let’s look at what happens in the other sequence of events – the one leading up to the pod being removed from the iptables rules (sequence B in the figure). When the Endpoints controller (which runs in the Controller Manager in the Kubernetes Control Plane) receives the notification of the pod being deleted, it removes the pod as an endpoint in all services that the pod is a part of. It does this by modifying the Endpoints API object by sending a REST request to the API server. The API server then notifies everyone watching the Endpoints object. Among those watchers are the Kube-Proxies running on the worker nodes. Each of these proxies updates the _iptables_ rules on its node, which is what prevents new connections from being forwarded to the terminating pod. An important detail here is that removing the iptables rules has no effect on existing connections – clients who’re already connected to the pod can still send additional requests to the pod through those existing connections.

Both of these sequences of events happen in parallel. Most likely, the time it takes to shut down the app process in the pod is slightly shorter than the time required for the _iptables_ rules to be updated. This is because the chain of events that leads to iptables rules being updated is considerably longer (see figure 2), because the event must first reach the Endpoints controller, which sends a new request to the API server, and then the API server must notify the Kube-Proxy, before the proxy finally modifies the _iptables_ rules. This means there’s a high probability of the SIGTERM signal being sent well before the iptables rules are updated on all nodes.


![[pod-deletion-timeline.png]]
The result is that the pod may still receive client requests after it has received the termination signal. If the app stops accepting connections immediately, it causes clients to receive “connection refused” types of errors (like what happens at pod startup if your app isn’t capable of accepting connections immediately and you don’t define a readiness probe for it).

> **Solution:** 
> Add a 5s sleep to the preStop hook to solve 99% of the above problems

```yaml
lifecycle:
  preStop:
    exec:
      command:
      - sh
      - -c
      - "sleep 5"
```

## Design Considerations

In order to start autoscaling the workload, the workload need to properly follow the k8s lifecycle. Probes must be utilized to ensure that the pod initializes properly. In addition we need a guarantee, that the pod will always attempt to terminate gracefully, in order not to loose any requests it may be processing. Pod termination may handle due to multiple reasons. Scaling down or Pod evictions are some of them.