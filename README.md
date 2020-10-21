# k8s-stress-research

Yes, I'm aware there are existing projects that were created to apply stress to Kubernetes clusters, but I'm working on some internal Microsoft projects that are needed for custom infrastructure. As such, the research presented by this repository will be largely based around raw `stress-ng` with no non-Kubernetes wrappers.

The way I'm looking at it currently, there's three options for applying stress using `stress-ng`:
1. Use a daemonset to run a container with `stress-ng` that tries to vampire suck all the resources of the node
1. Use a sidecar of sorts to get `stress-ng` inside the application pods and apply stress directly
1. Somehow install `stress-ng` on the actual VM (potentially via a daemonset) and stress the whole box

## Setup

1. Install the [Kubernetes Metrics Server](https://github.com/kubernetes-sigs/metrics-server#requirements)

## Tests Performed

### Local Docker Test

1. Used Docker on my development machine to run a `stress-ng` load test at 80% CPU for a minute:
	```
	docker run -it --rm alexeiled/stress-ng:latest-ubuntu --cpu 0 --cpu-load 80 --timeout 60s --metrics-brief
	```

Saw host CPU usage hold at 80% across all cores as expected.

### Local Kubernetes Test

1. Created a local Kubernetes cluster using [Kind](https://github.com/kubernetes-sigs/kind)
1. Applied a pod spec to the cluster that contains `stress-ng`:
	```
	kubectl apply -f pod.yaml
	```
1. Triggered the `stress-ng` pod to load test at 80% CPU for a minute:
	```
	kubectl exec -it <stress-ng-pod> -- stress-ng --cpu 0 --cpu-load 80 --timeout 60s
	```

Saw host CPU usage hold at 80% across all cores as expected.

### Local Kubernetes Test with Multiple Stressor Pods

1. Created a local Kubernetes cluster using [Kind](https://github.com/kubernetes-sigs/kind)
1. Applied multiple pod specs to the cluster that contains `stress-ng`:
	```
	kubectl apply -f pod.yaml
	kubectl apply -f pod2.yaml
	```
1. Triggered the `stress-ng` pods to load test at 80% CPU for a minute:
	```
	kubectl exec -it <stress-ng-pod1> -- stress-ng --cpu 0 --cpu-load 80 --timeout 60s
	kubectl exec -it <stress-ng-pod2> -- stress-ng --cpu 0 --cpu-load 80 --timeout 60s
	```

Saw host CPU usage hold at 100% across all cores. The competing pods were not able to take CPU above 100% usage.

## Questions to Answer

- Are we trying to chaos test Kubernetes as a service, or chaos test a service that runs on Kubernetes?
- Is it possible to guarantee that a daemonset has unlimited resource constraints?
- Do we want a daemonset to have unlimited resource constraints, or to respect the cluster resource quote in the event of multitenant situations?
- Do daemonsets get rescheduled to other nodes when resources are consumed?
- Does Kubernetes react differently when the machine is stressed directly vs stressed from inside a container (e.g. Will it reschedule pods in one situation and not in another; we want to simulate real load as much as possible)?
- How does networking inside a daemonset work?
- How do we make sure the communication cert gets installed inside a daemonset container?
- Would we save time/effort by instead wrapping an existing k8s stressing framework like [Pumba](https://github.com/alexei-led/pumba)?

## Arguments for Containerized Daemonset

- Would work and be able to more easily respect multitenant resource namespacing
- Might be simpler to interact with Kubernetes networking from inside the Kubernetes network
