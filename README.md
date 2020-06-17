# mikroways.m7s_k8s_helm

[![Ansible Galaxy][galaxy_image]][galaxy_link]
[![Latest tag][tag_image]][tag_url]

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
  - name: dex-authenticator
    git_repo: https://github.com/mintel/dex-k8s-authenticator.git
    chart_base_path: charts/dex-k8s-authenticator
    upgrade: true
    namespace: dex
    config_values:
      global:
        deployEnv: prod
      dexK8sAuthenticator:
        logoUrl: https://mikroways.net/images/mikroways.png
```

> The last example installs a chart from git, so `git_repo` and `chart_base_path`
> must be specified
> It also shows how to allow upgrade of charts. 

By default charts will not be upgraded

### Notes

* By default the stable repo used by the example above is already set.
* When a chart version is not set, the latest will be installed
* When namespace is specified, a namespace will be created
* When state is not specified, it is assumed to be present
* When state is absent, chart will be uninstalled but namespace will not be
  deleted
* When config_values are modified, helm chart will not be updated. 
  * To upgrade a chart the **upgrade: true** option must be set
 
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

[galaxy_image]:         http://img.shields.io/badge/galaxy-mikroways.m7s__k8s__helm-660198.svg?style=flat
[galaxy_link]:          https://galaxy.ansible.com/Mikroways/m7s_k8s_helm
[tag_image]:            https://img.shields.io/github/v/tag/Mikroways/ansible_role_m7s_k8s_helm.svg
[tag_url]:              https://github.com/Mikroways/ansible_role_m7s_k8s_helm/tags
