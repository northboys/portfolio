---
layout: post
title: Container security best practices
---

Before your application inside a container is executed, there are several places where you can start applying different techniques to prevent threats from happening.

Prevention and applying container security as early as possible is key and will save you a lot of trouble, time, and money with minimal effort if you apply some good practices during the development and distribution of the container images.

To enabled access or connection logging in all sidecar and gateway. We can create EnvoyFilter that enabled access logging in the sidecar.

1. Integrate Code Scanning at the CI/CD Process
2. Reduce external vulnerabilities via dependency scanning
3. Use image scanning to analyze container images

### Container security tools
- [Dive](https://github.com/wagoodman/dive)
A tool for exploring a docker image, layer contents, and discovering ways to shrink the size of your Docker/OCI image.
- [Dockle](https://github.com/goodwithtech/dockle)
Container Image Linter for Security, Helping build the Best-Practice Docker Image, Easy to start 
- [Trivy](https://github.com/aquasecurity/trivy)
is a simple and comprehensive vulnerability/misconfiguration/secret scanner for containers and other artifacts. Trivy detects vulnerabilities of OS packages (Alpine, RHEL, CentOS, etc.) and language-specific packages (Bundler, Composer, npm, yarn, etc.). In addition, Trivy scans Infrastructure as Code (IaC) files such as Terraform and Kubernetes, to detect potential configuration issues that expose your deployments to the risk of attack. 

May this is help for you who implement DevSecOps. 
