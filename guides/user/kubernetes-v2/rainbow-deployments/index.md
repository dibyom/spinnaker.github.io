---
layout: single
title:  "Rainbow Deployments using Kubernetes"
sidebar:
  nav: guides
---

{% include alpha version="1.8" %}

{% include toc %}


This guide shows how you can implement a [Rainbow](TODO:LINK) deployment strategy for your applications deployed using the Kubernetes V2 provider. At a high level, the approach is to add a label whose value uniquely identifies the ReplicaSet being deployed and to then update the Kubernetes service to point to the new ReplicaSet.

TODO: Add more details on how it works i.e. execution id.

## Pre-Requisites/Assumptions

We assume you have a pipeline setup that deploys a manifest to a Kubernetes cluster. In addition, you need to have a basic service deployed:

```yml
apiVersion: v1
kind: Service
metadata:  
  name: patch-example-service
  namespace: default
spec:
  type: NodePort
  ports:  # Change the ports to match your applications
  - name: http
    port: 80 
    targetPort: 80
    nodePort: 30037
    protocol: TCP

```

## Configure deployment

Add a new label, say `deploy-color` to your manifest text in the Deploy (Manifest) stage of your pipeline. Set it's value to pipeline execution id using [pipeline expressions]():

```yaml
# Rest of manifest ....
    labels:
        deploy-color: "${execution.id}"
# Rest of manifest ...
```

## Configure service

Add a new Patch (Manifest) stage to your pipeline that depends on the previous Deploy (Manifest) stage. Configure the stage to patch the service created i.e for the above service, set namespace to default, kind to Service, and name to patch-example-service. 

TODO: Screenshot

Next, add the patch content to update the service's selectors to the `deploy-color` created in the deployment step:

```yaml
spec:
  selector:
    color: '${execution.id}'
```

## Managing old versions

Q. Why not just redeploy the service?
