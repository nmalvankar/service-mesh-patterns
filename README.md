# Service Mesh Patterns

This project provides examples for widely practiced Service Mesh configurations.

> NOTE: These examples assume the control plane is already deployed.
> To deploy the mesh, follow the instructions in the [trevorbox/service-mesh](https://github.com/trevorbox/service-mesh) project.

## Setup

Export Default vars

```sh
source default-vars.txt && export $(cut -d= -f1 default-vars.txt)
```

*Or*, Export Custom vars

```sh
export deploy_namespace=bookinfo
export control_plane_namespace=istio-system
export control_plane_name=basic-install
export control_plane_route_name=api
```

## Basic Gateway Configuration

This example demonstrates a basic confiuration using:

- A single Gateway deployed in the *"<control_plane_namespace>"*.
- A VirtualService deployed in the member namespace referencing the Gateway in *"<control_plane_namespace>/<gateway_name>"*.

Deploy

```sh
./deploy-basic-gateway-configuration.sh
```

Test the bookinfo application

```sh
# Open the following url in a web browser
echo "https://$(oc get route ${control_plane_route_name} -n ${control_plane_namespace} -o jsonpath={'.spec.host'})/productpage"
```

## Multiple Ingress Gateways with MongoDB

This example shows how to deploy MongoDB behind Service Mesh on Openshift and open a NodePort on the mongo ingress gateway for external communication. With this configuration we can present a certificate in the mongo-ingressgateway proxy and test TLS connections from outside the mesh to MongoDB. A normal Openshift route does not support the mongo protocol.

### Install the Service Mesh

Follow the README.md within the service-mesh folder to deploy the control-plane-mongodb

### Install mongodb app

```sh
export deploy_namespace=mongodb

oc new-project ${deploy_namespace}

helm install mongodb -n ${deploy_namespace} mongodb/
```

### Install Mongo Gateway Configuration

> TODO: refactor bookinfo + mongo into the same chart since there is a new reviews-v2 deployment

```sh
export deploy_namespace=mongodb

helm install mongo-gateway-configuration -n ${deploy_namespace} mongo-gateway-configuration/
```

1. Login

   ```sh
   oc login <mycluster>
   ```

2. Update playbook with appropriate k8s_namespace variable
3. Run playbook on cluster

   ```sh
   ansible-playbook playbook.yml
   ```

4. Test normal connectivity to LoadBalancer

   ```sh
   ./scripts/ingress-mongodb-setup.sh
   ```

5. Test TLS connectivity to LoadBalancer

   ```sh
   ./scripts/ingress-mongodb-setup-tls.sh
   ```
