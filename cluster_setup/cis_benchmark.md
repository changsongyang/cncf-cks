# CIS Benchmark

## What is CIS & CIS Benchmark?

CIS (stands for **C**enter for **I**nternet **S**ecurity) is a non-profit organization that helps people, businesses and goverments against cyber threats. 

CIS Benchmarks are best practice security configuration for systems, and Kubernetes is one of the supported server software.
- [CIS_Kubernetes_Benchmark_v1.9.0.pdf - 03-25-2024](../assets/CIS_Kubernetes_Benchmark_v1.9.0%20PDF.pdf) for Kubernetes v1.27 - v1.29

## CIS Benchmark in Action

### What is [kube-bench](https://github.com/aquasecurity/kube-bench)?

[kube-bench](https://github.com/aquasecurity/kube-bench) is a tool that runs checks against your Kubernetes cluster accordinly to security best practices as defined inthe CIS Kubernets Benchmark.

#### Running inside a container

```sh
docker run --pid=host -v /etc:/etc:ro -v /var:/var:ro -t docker.io/aquasec/kube-bench:latest --version 1.18
```

#### Run as a job

```sh
wget  https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job.yaml

k apply -f job.yaml

k logs kube-bench-*
```

## Resources

- [Martin White - Consistent Security Controls through CIS Benchmarks
](https://www.youtube.com/watch?v=53-v3stlnCo)

