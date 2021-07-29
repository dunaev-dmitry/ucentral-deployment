# Helm deployment

Helm deployment is intended to be used for cloud deployments, but may also be used for on-premise deployments or even deployments in local development clusters like [MicroK8s](https://microk8s.io/).

This method requires some knowledge of Kubernetes and Helm, as well as general understanding on how Helm subchart values work. If you would like to simply tryout Open Wi-Fi Cloud SDK but don't have enough knowledge on these topics we highly recommend you condidering [Docker-compose deployment](docker-compose-deployment.md).

Helm deployment is done using assembly chart stored in [wlan-cloud-ucentral-deploy GitHub repository](https://github.com/Telecominfraproject/wlan-cloud-ucentral-deploy/tree/main/chart) which pulls Helm charts from other microservices repositories using [helm-git plugin](https://github.com/aslafy-z/helm-git), so you need to install it in order to install Open Wi-Fi Cloud SDK 2.0.

Currently only basic Helm values are supported that may be used for deployment to development clusters, but requires modifications in order to work correctly in other environments.

In future example values will be added into assembly chart for different environments.

In this documentation we will go through general deployment process leaving Helm values modification to you.

### Prerequisites

Following list of instruments should be installed on your machine in order to run Helm deployment \(we are considering you already have Kubernetes cluster and configured connection to it with Kubectl\):

* [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
* [Helm](https://helm.sh/docs/intro/install/)
* [Helm-git](https://github.com/aslafy-z/helm-git)

This may be done with the following commands on Mac OS X or GNU/Linux:

```text
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
helm plugin install https://github.com/aslafy-z/helm-git
```

Also in order for you application to work correctly, you need to have support for LoadBalancer service type as microservices must have static IP addesses that your Access Points can connect to. For development clusters [MetalLB](https://metallb.universe.tf/) may be used.

### Deploy TIP Controller

TIP Controller services use SSL certificates to ensure inter-service and Access Point connectivity security. These certificates may be generated, but as an entrypoint you may use TIP issued certificates from DigiCert.

For the deployment we will use prebundled chart with all required dependencies. To do that, you need to add public Helm repo first:

```text
helm repo add tip-wlan-cloud-ucentral-helm https://tip.jfrog.io/artifactory/tip-wlan-cloud-ucentral-helm/
```

Now he have almost everything ready for your deployment, since we already have all required TLS certificates in the same repository [that you may use](https://github.com/Telecominfraproject/wlan-cloud-ucentral-deploy/tree/main/docker-compose/certs) or use your own:

```text
helm upgrade --install --create-namespace --wait \
    --set-file ucentralgw.certs."restapi-cert\.pem"=restapi-cert.pem \
    --set-file ucentralgw.certs."restapi-key\.pem"=restapi-key.pem \
    --set-file ucentralgw.certs."websocket-cert\.pem"=websocket-cert.pem \
    --set-file ucentralgw.certs."websocket-key\.pem"=websocket-key.pem \
    --set-file rttys.certs."restapi-cert\.pem"=restapi-cert.pem \
    --set-file rttys.certs."restapi-key\.pem"=restapi-key.pem \
    --set-file ucentralsec.certs."restapi-cert\.pem"=restapi-cert.pem \
    --set-file ucentralsec.certs."restapi-key\.pem"=restapi-key.pem \
    -n tip \
    tip-ucentral tip-wlan-cloud-ucentral-helm/wlan-cloud-ucentral
```

This will deploy the uCentral Helm chart with its default configuration. The only required configuration is passing along the TLS certificates that will be used for various SSL endpoints. Further parameters can be found in the `values.yaml` file in the [parent](https://github.com/Telecominfraproject/wlan-cloud-ucentral-deploy/blob/main/chart/values.yaml) and in each of the subcharts \([ucentralgw](https://github.com/Telecominfraproject/wlan-cloud-ucentralgw/blob/master/helm/values.yaml), [ucentralsec](https://github.com/Telecominfraproject/wlan-cloud-ucentralsec/blob/main/helm/values.yaml), [rtty](https://github.com/Telecominfraproject/wlan-cloud-ucentralgw-rtty/blob/main/chart/values.yaml), [ui](https://github.com/Telecominfraproject/wlan-cloud-ucentralgw-ui/blob/main/helm/values.yaml)\). 

After deployment finished successfully you can check if everything works correctly by running `kubectl get pods -n tip`:

```text
NAME                                        READY   STATUS    RESTARTS   AGE
kafka-0                                     1/1     Running   0          15m
rttys-878df95d4-gn99x                       1/1     Running   0          15m
ucentralgw-6c955546c6-w2mw4                 1/1     Running   0          15m
ucentralgwui-fcbbbf84b-g7bcl                1/1     Running   0          15m
ucentralsec-74f447758d-ssdn5                1/1     Running   0          15m
zookeeper-0                                 1/1     Running   0          15m
```

After this you may access Web UI in your browser with default credentials:

* Login: `tip@ucentral.com`
* Password: `openwifi`

