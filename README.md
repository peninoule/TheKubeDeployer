Role Name
=========

Ansible Role used to deploy a Kubernetes (K8S) cluster on Virtual Machines
It uses `Cri-O` as the container runtime and `Calico` as the Container Network Plugin

Requirements
------------

# NODES
At least 2 nodes must be present on the inventory (See inventory example below) (1 Controlplane, 1 Worker Node).
/!\: Having multiple Master nodes is not supported yet

## Operating systems
The clusters nodes must run a `yum` like distribution. (RHEL, Centos, Rocky...)
The role has been tested on Rocky 9 but should work on any yum like distrib

## Connectivity
- Nodes inside the cluster must have full dns resolution (forward and reverse) and network access
- Nodes must have Internet access to download updates and setup kubernetes repositories

---

## Ansible controller requirements:
The following packages must be installed on the Ansible Controller
- `helm`: The role installs [Calico](https://www.tigera.io/project-calico/) CNI via helm, ** on the Ansible Control node (The one used to launch the playbook) **
- [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl)

Role Variables
--------------

1.`vars/main.yml`
You can leave this variables as they are
| Name | Type | Default | 
| ---- | ---- | ------- |
| packages_prerequisites | list | ["curl", "chrony"] |
| mandatory_sysctl_params | map | {net.ipv4.ip_forward: 1} | 
| mandatory_kernel_modules | list | ["br_netfilter", "overlay"] |

2.defaults/main.yml
You can adjust to your needs

| Name | Type | Required | Default | Description |
| ---- | ---- | -------- | ------- | ----------- |
| os | str | yes | "CentOS_8" | Name of the Os used to craft kube repo URL. See [this link for an example url](https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/1.21:/1.21.7/) |
| additional_kernel_modules | list | No | None | List of additional sysctl params | 
| additional_sysctl_params | map | No | Key / Value Pair of additional_sysctl_params |
| chrony_servers | list | yes | ["1.1.1.1"] | List of Chrony servers to set | 
| timezone | str | yes | "Europe/Paris" | The Timezone to set |
| kube_container_runtime | str | yes | "crio" | CRI to use (Role will only work with Crio anyway) |
| crio_major_version | str | yes | "1.24" | Major Version of Crio to use |
| crio_minor_version | str | yes | "4" | Minor Version of Crio to use |
| pod_network_cidr | str | yes | "10.244.0.0/16" | CIDR for pods. If unsure, leave it as it is |
| control_plane_endpoint | str | yes | None | Endpoint for your cluster. Either a DNS name or an ip address. If unsure, set dns name bound to your control plane. You will be able to change the DNS binding afterwards |
| calico_operator_namespace | str | yes | "tigera-operator" | Namespace to install the calico operator |

Dependencies
------------

None

Inventory Setup
---------------

Your inventory must contain the following groups:
- bootstrap: The bootstrap group should container 1 node (Typically your master node). The `kubeadm init` command will launched from the bootstrap
- masters: list of masters nodes (Only 1 master supported yet)
- workers: list of workers (Multiple workers allowed)

Example:

```yaml
all:
  hosts:
    master1.example.com:
    worker1.example.com:
  children:
    kubernetes:
      hosts:
        master1.example.com:
        worker1.example.com:
      children:
        bootstrap:
          hosts:
            master1.example.com:
        masters:
          hosts:
            master1.example.com:
        workers:
          hosts:
            worker1.example.com:
```

Example Playbook
----------------

    - hosts: kubernetes
      roles:
         - { role: TheKubeDeployer, control_plane_endpoint: "kubernetes.example.com" }

Author Information
------------------

Les Tanches
