# Enable DevOps After Installation kubesphere
Log in to the console as admin. Click Platform in the upper-left corner and select Cluster Management.

Click `CRDs` and enter `clusterconfiguration` in the search bar. Click the result to view its detail page.

## Info

A Custom Resource Definition (CRD) allows users to create a new type of resources without adding another API server. They can use these resources like any other native Kubernetes objects.
In Custom Resources, click  on the right of ks-installer and select Edit YAML.

In this YAML file, search for devops and change false to true for enabled. After you finish, click OK in the lower-right corner to save the configuration.
```yaml
devops:
  enabled: true # Change "false" to "true".
```
Use the web kubectl to check the installation process by running the following command:
```sh
kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l 'app in (ks-install, ks-installer)' -o jsonpath='{.items[0].metadata.name}') -f
```
### Note

You can find the web kubectl tool by clicking  in the lower-right corner of the console.