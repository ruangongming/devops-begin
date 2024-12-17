# Enable KubeSphere Service Mesh After Installation
Log in to the console as admin. Click Platform in the upper-left corner and select Cluster Management.

Click `CRDs` and enter `clusterconfiguration` in the search bar. Click the result to view its detail page.

## Info

A Custom Resource Definition (CRD) allows users to create a new type of resources without adding another API server. They can use these resources like any other native Kubernetes objects.
In Custom Resources, click  on the right of ks-installer and select Edit YAML.

In this YAML file, navigate to servicemesh and change false to true for enabled. After you finish, click OK in the lower-right corner to save the configuration.
``` yaml
servicemesh:
enabled: true # Change “false” to “true”.
istio: # Customizing the istio installation configuration, refer to https://istio.io/latest/docs/setup/additional-setup/customize-installation/
  components:
    ingressGateways:
    - name: istio-ingressgateway # Used to expose a service outside of the service mesh using an Istio Gateway. The value is false by defalut.
      enabled: false
    cni:
      enabled: false # When the value is true, it identifies user application pods with sidecars requiring traffic redirection and sets this up in the Kubernetes pod lifecycle’s network setup phase.
```
Run the following command in kubectl to check the installation process:
```sh
kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l 'app in (ks-install, ks-installer)' -o jsonpath='{.items[0].metadata.name}') -f
Note
```
You can find the web kubectl tool by clicking  in the lower-right corner of the console.