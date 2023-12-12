---
title: Install Voyager
description: Installation guide for Voyager
menu:
  docs_{{ .version }}:
    identifier: install-voyager-enterprise
    name: Voyager
    parent: installation-guide
    weight: 20
product_name: voyager
menu_name: docs_{{ .version }}
section_menu_id: setup
---

# Install Voyager

## Get a Free Trial License

In this section, we are going to show you how you can get a **30 days trial** license for Voyager. You can get a license for your Kubernetes cluster by going through the following steps:

- At first, go to [AppsCode License Server](https://appscode.com/issue-license?p=voyager) and fill up the form. It will ask for your Name, Email, the product you want to install, and your cluster ID (UID of the `kube-system` namespace).
- Provide your name and email address. **You must provide your work email address**.
- Then, select `Voyager` in the product field.
- Now, provide your cluster ID. You can get your cluster ID easily by running the following command:

```bash
kubectl get ns kube-system -o=jsonpath='{.metadata.uid}'
```

- Then, you have to agree with the terms and conditions. We recommend reading it before checking the box.
- Now, you can submit the form. After you submit the form, the AppsCode License server will send an email to the provided email address with a link to your license file.
- Navigate to the provided link and save the license into a file. Here, we save the license to a `license.txt` file.

Here is a screenshot of the license form.

<figure align="center">
  <img alt="Voyager License Form" src="/docs/images/setup/enterprise_license_form.png">
  <figcaption align="center">Fig: Voyager License Form</figcaption>
</figure>

You can create licenses for as many clusters as you want. You can upgrade your license any time without re-installing Voyager by following the upgrading guide from [here](/docs/setup/upgrade/index.md#updating-license).

> Voyager licensing process has been designed to work with CI/CD workflow. You can automatically obtain a license from your CI/CD pipeline by following the guide from [here](https://github.com/appscode/offline-license-server#api-reference).

## Purchase Voyager License

If you are interested in purchasing Voyager license, please contact us via sales@appscode.com for further discussion. You can also set up a meeting via our [calendly link](https://calendly.com/appscode/30min).

If you are willing to purchase Voyager license but need more time to test in your dev cluster, feel free to contact sales@appscode.com. We will be happy to extend your trial period.

## Install

<ul class="nav nav-tabs" id="installerTab" role="tablist">
  <li class="nav-item">
    <a class="nav-link active" id="helm3-tab" data-toggle="tab" href="#helm3" role="tab" aria-controls="helm3" aria-selected="true">Helm 3 (Recommended)</a>
  </li>
  <li class="nav-item">
    <a class="nav-link" id="script-tab" data-toggle="tab" href="#script" role="tab" aria-controls="script" aria-selected="false">YAML</a>
  </li>
</ul>
<div class="tab-content" id="installerTabContent">
  <div class="tab-pane fade show active" id="helm3" role="tabpanel" aria-labelledby="helm3-tab">

## Using Helm 3

Voyager can be installed via [Helm](https://helm.sh/) using the [chart](https://github.com/voyagermesh/installer/tree/{{< param "info.version" >}}/charts/voyager) from [AppsCode Charts Repository](https://github.com/appscode/charts). To install, follow the steps below:

```bash
# provider=acs
# provider=aks
# provider=aws
# provider=azure
# provider=baremetal
# provider=gce
# provider=gke
# provider=kind
# provider=openstack
# provider=metallb
# provider=digitalocean
# provider=linode

$ helm install voyager oci://ghcr.io/appscode-charts/voyager \
  --version {{< param "info.version" >}} \
  --namespace voyager --create-namespace \
  --set cloudProvider=$provider \
  --set-file license=/path/to/the/license.txt \
  --wait --burst-limit=10000 --debug
```

To see the detailed configuration options, visit [here](https://github.com/voyagermesh/installer/tree/{{< param "info.version" >}}/charts/voyager).

</div>
<div class="tab-pane fade" id="script" role="tabpanel" aria-labelledby="script-tab">

## Using YAML

If you prefer to not use Helm, you can generate YAMLs from Voyager chart and deploy using `kubectl`. Here we are going to show the procedure using Helm 3.

```bash
# provider=acs
# provider=aks
# provider=aws
# provider=azure
# provider=baremetal
# provider=gce
# provider=gke
# provider=kind
# provider=openstack
# provider=metallb
# provider=digitalocean
# provider=linode

$ kubectl create ns voyager
$ helm template voyager oci://ghcr.io/appscode-charts/voyager \
  --version {{< param "info.version" >}} \
  --namespace voyager --create-namespace \
  --set cloudProvider=$provider \
  --set-file license=/path/to/the/license.txt \
  --set cleaner.skip=true | kubectl apply -f -
```

To see the detailed configuration options, visit [here](https://github.com/voyagermesh/installer/tree/{{< param "info.version" >}}/charts/voyager).

</div>
</div>

## Verify installation

To check if Voyager operator pods have started, run the following command:

```bash
$ kubectl get pods --all-namespaces -l app.kubernetes.io/name=voyager --watch

NAMESPACE   NAME                               READY   STATUS    RESTARTS   AGE
voyager     voyager-operator-84d575d55-5lphm   1/1     Running   0          6m42s
```

Once the operator pods are running, you can cancel the above command by typing `Ctrl+C`.

Now, to confirm CRD groups have been registered by the operator, run the following command:

```bash
$ kubectl get crd -l app.kubernetes.io/name=voyager
```

Now, you are ready to create your first ingress using Voyager.

## Configuring RBAC

Voyager creates an `Ingress` CRD. Voyager installer will create 2 user facing cluster roles:

| ClusterRole           | Aggregates To | Description                           |
|-----------------------|---------------|---------------------------------------|
| appscode:voyager:edit | admin, edit   | Allows edit access to Voyager CRDs, intended to be granted within a namespace using a RoleBinding. |
| appscode:voyager:view | view          | Allows read-only access to Voyager CRDs, intended to be granted within a namespace using a RoleBinding. |

These user facing roles supports [ClusterRole Aggregation](https://kubernetes.io/docs/admin/authorization/rbac/#aggregated-clusterroles) feature in Kubernetes 1.9 or later clusters.


## Using kubectl

Since Voyager uses its own TPR/CRD, you need to use full resource kind to find it with kubectl.

```bash
# List all voyager ingress
$ kubectl get ingress.voyager.appscode.com --all-namespaces

# List voyager ingress for a namespace
$ kubectl get ingress.voyager.appscode.com -n <namespace>

# Get Ingress YAML
$ kubectl get ingress.voyager.appscode.com -n <namespace> <ingress-name> -o yaml

# Describe Ingress. Very useful to debug problems.
$ kubectl describe ingress.voyager.appscode.com -n <namespace> <ingress-name>
```
