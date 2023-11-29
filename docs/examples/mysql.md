# Expose MySQL Database Server with Voyager Gateway
Voyager Gateway supports protocol specific traffic routing for MySQL database servers. This project introduces `MySQLRoute` which is used to expose MySQL servers to the gateway including the mysql specific envoy filters. The envoy filter is also extended to support TLS secured upstream and downstream connections. Features like TLS termination is also implemented to decouple dependencies.
In this article we will learn how to expose a MySQL database server with Voyager Gateway and explore all the potentials step by step. We will learn how to :
- expose MySQL database server on plain TCP 
- secure client-to-gateway connection with TLS
- secure gateway-to-backend connection with TLS

### Environment Setup 
- install voyager gateway 
- deploy mysql server with KubeDB

### Deploy Gateway Class
GatewayClass is cluster-scoped resource defined by the infrastructure provider. This resource represents a class of Gateways that can be instantiated. See official k8s (documentation)[https://gateway-api.sigs.k8s.io/api-types/gatewayclass/] for details. 

As Voyager Gateway uses Envoyproxy under the hood, we need to set `gateway.envoyproxy.io/gatewayclass-controller` as the controller name. Also we will use EnvoyProxy crd to customize deployment images or service type. So let's pass a EnvoyProxy reference in the parameterRef for now, later we will discuss how to create one.  

```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: GatewayClass
metadata:
  name: eg
spec:
  controllerName: gateway.envoyproxy.io/gatewayclass-controller
  parametersRef:
    group: gateway.envoyproxy.io
    kind: EnvoyProxy
    name: custom-proxy-config
    namespace: envoy-gateway-system
```
Apply this yaml to the cluster, 
```bash
$ kubectl apply -f /home/office/go/src/voyagermesh.dev/gateway-docs/docs/yamls/gatewayclass.yaml
gatewayclass.gateway.networking.k8s.io/eg created
```

### Configure Envoyproxy
```yaml
apiVersion: gateway.envoyproxy.io/v1alpha1
kind: EnvoyProxy
metadata:
  name: custom-proxy-config
  namespace: gateway-system
spec:
  provider:
    type: Kubernetes
    kubernetes:
      envoyDeployment:
        container:
          image: docker-opc-ecv-group-cicd-nexus.rp-ocn.apps.ocn.infra.ftgroup/voyagermesh/envoy:v1.28.1 #docker.io/shiponcs/envoy-contrib:v1.27
          securityContext: 
            runAsUser: 1000
      envoyService:
        type: NodePort
```

## Configure the Gateway for Plain TCP connection 
```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: Gateway
metadata:
  name: mysql-tcp-gw  
  namespace: gateway-ns
spec:
  gatewayClassName: eg
  listeners:
  - name: mysql-tcp-lis 
    protocol: TCP
    port: 10001
    allowedRoutes:
      kinds:
        - group: gateway.voyagermesh.com
          kind: MySQLRoute
```

## Configure the backend 
```yaml
apiVersion: kubedb.com/v1alpha2
kind: MySQL
metadata:
  name: mysql-tcp
  namespace: db-ns
spec:
  version: "8.0.31"
  storageType: Durable
  storage:
    storageClassName: "standard"
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 1Gi
  terminationPolicy: WipeOut
```

```yaml
apiVersion: gateway.voyagermesh.com/v1alpha1
kind: MySQLRoute
metadata:
  name: mysql-tcp-route
  namespace: db-ns
spec:
  parentRefs:
  - name: mysql-tcp-gw
    sectionName: mysql-tcp-lis
  rules:
  - backendRefs:
    - name: mysql-tcp
      port: 3306
```




## Configure the Gateway for client-to-gateway TLS secure connection
```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: Gateway
metadata:
  name: mysql-tls-gw 
  namespace: gateway-ns
spec:
  gatewayClassName: eg
  listeners:
  - name: mysql-tls-lis
    protocol: TLS
    tls:
      mode: Terminate
      certificateRefs:
        - name: mysql-tls-server-cert
          namespace: db-ns
    port: 10002
    allowedRoutes:
      kinds:
        - group: gateway.voyagermesh.com
          kind: MySQLRoute
```

## Configure the Upstream TLS
```yaml 
apiVersion: gateway.networking.k8s.io/v1alpha2
kind: BackendTLSPolicy
metadata:
  name: mysql-backendtls
  namespace: default
spec:
  targetRef: 
    group: ''
    kind: Service
    name: mysql
    namespace: default
  tls: 
    caCertRefs:
    - name: mysql-client-cert
      group: ''
      kind: Secret
    hostname: mysql
```


## Expose MySQL on plain TCP

mariadb mysql walg => ship those
mysql codebase overview => 
singlestore takeover => code review