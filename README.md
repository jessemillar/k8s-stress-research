# k8s-stress-research

Yes, I'm aware there are existing projects that were created to apply stress to Kubernetes clusters, but I'm working on some internal Microsoft projects that are needed for custom infrastructure. As such, the research presented by this repository will be largely based around raw `stress-ng` with no non-Kubernetes wrappers.

## Setup

1. Install the [Kubernetes Metrics Server](https://github.com/kubernetes-sigs/metrics-server#requirements)

## Testing

### Docker Test

Run a `stress-ng` load test at 80% CPU for a minute:

```
docker run -it --rm alexeiled/stress-ng:latest-ubuntu --cpu 0 --cpu-load 80 --timeout 60s --metrics-brief
```
