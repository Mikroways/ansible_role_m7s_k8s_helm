Explicar que crea los namespace, no los elimina
Explicar que no actualiza, y tampoco cambia los valores


# mikroways.m7s_k8s_helm

This role installs helm based on role `andrewrothstein.kubernetes-helm`. Then it
allows defining which charts to install using variables.

## Role Variables

This role provides with the following variables:

* `m7s_k8s_helm_repos:` array of helm repositories. Each member must define a
  name and a url.
* `m7s_k8s_helm_charts:` arrayof charts to install. Each member must define a
  name, a chart name a version of chart a namespace and chart values.

For example:

```yaml
m7s_k8s_helm_repos:
  - name: stable
    url: https://kubernetes-charts.storage.googleapis.com/
    state: present

m7s_k8s_helm_charts:
  - name: nginx-ingress
    chart: stable/nginx-ingress
    version: 1.26.2
    namespace: nginx-ingress
    state: present
    config_values:
      controller:
        hostNetwork: true
        dnsPolicy: ClusterFirstWithHostNet
        reportNodeInternalIp: true
        kind: DaemonSet
        service:
          type: NodePort
```

### Notes

* By default the stable repo used by the example above is already set.
* When a chart version is not set, the latest will be installed
* When namespace is specified, a namespace will be created
* When state is not specified, it is assumed to be present
* When state is absent, chart will be uninstalled but namespace will not be
  deleted
* When config_values are modified, helm chart will not be updated. Do it manual
  or uninstall and then install to apply changes.

## Dependencies

It only depends on `andrewrothstein.kubernetes-helm`

## Example Playbook

```yaml
- hosts: kube-master
  tasks:
    - name: install helm
      include_role:
        name: mikroways.m7s_k8s_helm
        tasks_from: install-helm.yml
      tags: [ helm ]

- hosts: kube-master[0]
  roles:
    - { role: mikroways.m7s_k8s_helm, tags: [ helm ] }

```
> Note that the example show how to install helm binary on all master nodes, but
> only run (un)install helm charts from one master node


## License

GPLv2

