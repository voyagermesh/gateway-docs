---
title: Using Custom HAProxy Templates | Kubernetes Ingress
menu:
  docs_{{ .version }}:
    identifier: custom-tpl-config
    name: Custom Templates
    parent: config-ingress
    weight: 10
product_name: voyager
menu_name: docs_{{ .version }}
section_menu_id: guides
---
> New to Voyager? Please start [here](/docs/concepts/overview.md).

# Using Custom HAProxy Templates

Since 3.2.0, Voyager can use custom templates provided by users to render HAProxy configuration. Voyager comes with a set of GO [text/templates](https://golang.org/pkg/text/template/) found [here](https://github.com/voyagermesh/gateway-docs/tree/{{< param "info.version" >}}/hack/docker/voyager/templates). These templates are mounted at `/srv/voyager/templates`. You can mount a ConfigMap with matching template names when installing Voyager operator to a different location and pass that to Voyager operator using `--custom-templates` flag. Voyager will [load](https://github.com/voyagermesh/gateway-docs/blob/3ae30cd023ff8fa6301d2656bf9fbc5765529691/pkg/haproxy/template.go#L40) the built-in templates first and then load any custom templates if provided. As long as the custom templates have [same name](https://golang.org/pkg/text/template/#Template.ParseGlob) as the built-in templates, custom templates will be used render HAProxy config. You can overwrite any number of templates as you wish. Also note that templates are loaded when Voyager operator starts. So, if you want to reload custom templates, you need to restart the running Voyager operator pod (not HAProxy pods).

In this example, we are going to overwrite the [defaults.cfg](https://raw.githubusercontent.com/voyagermesh/voyager/{{< param "info.version" >}}/hack/docker/voyager/templates/defaults.cfg) template which is used to render the [`defaults`](https://github.com/voyagermesh/gateway-docs/blob/3ae30cd023ff8fa6301d2656bf9fbc5765529691/hack/docker/voyager/templates/haproxy.cfg#L6) section of HAProxy config.

```bash
$ cat /tmp/defaults.cfg

defaults
	log global

	# my custom template
	# https://cbonte.github.io/haproxy-dconv/1.7/configuration.html#4.2-option%20abortonclose
	option dontlog-normal
	log /dev/log local0 notice alert
	option dontlognull
	option http-server-close

	# Timeout values
	timeout client 5s
	timeout client-fin 5s
	timeout connect 5s
	timeout server 5s
	timeout tunnel 5s

	# default traffic mode is http
	# mode is overwritten in case of tcp services
	mode http
```

Now create a ConfigMap using the defaults.cfg as key and the file content as the value.

```bash
$ kubectl create configmap -n voyager voyager-templates --from-file=/tmp/defaults.cfg
```

Now, the ConfigMap `voyager-templates` has to be mounted in the voyager operator pod and `--custom-templates` flag has to be set. To do this, set `templates.cfgmap` value to Voyager operator chart.

```bash
$ helm install voyager oci://ghcr.io/appscode-charts/voyager \
  --version {{< param "info.version" >}} \
  --namespace voyager --create-namespace \
  --set cloudProvider=minikube \
  --set templates.cfgmap=voyager-templates \
  --wait --burst-limit=10000 --debug
```

![installer](/docs/images/ingress/configuration/custom-template/installer.png)

This will restart the Voyager operator pods. After start, Voyager pod will update any existing HAProxy configuration to the new templates.
