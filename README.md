<div align="center">

# homelab-infra

![Ansible](https://img.shields.io/badge/Ansible-Automation-red?style=for-the-badge&logo=ansible&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-Cluster-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Containerd](https://img.shields.io/badge/Containerd-Runtime-575757?style=for-the-badge&logo=docker&logoColor=white)
![Argo CD](https://img.shields.io/badge/ArgoCD-GitOps-EF7B4D?style=for-the-badge&logo=argo&logoColor=white)

</div>

Infraestrutura de homelab baseada em Ansible para preparar hosts Linux, instalar containerd, configurar Kubernetes e aplicar addons no cluster.

## Visao Geral

Este repositório organiza a automação da infraestrutura em torno de um fluxo simples:

1. Preparar os hosts com dependências comuns.
2. Instalar e configurar o runtime de containers.
3. Instalar os componentes do Kubernetes.
4. Inicializar o primeiro nó de control-plane.
5. Adicionar os demais nós de control-plane e workers.
6. Aplicar addons e serviços do cluster.

O objetivo é manter a criação e manutenção do cluster reproduzíveis, com inventário versionado e playbooks separados por responsabilidade.

## Estrutura

O repositório está dividido nas seguintes áreas:

- [inventory/hosts.yml](inventory/hosts.yml): define os grupos de hosts usados pelos playbooks.
- [inventory/group_vars/all.yml](inventory/group_vars/all.yml): variáveis globais do ambiente.
- [inventory/host_vars](inventory/host_vars): variáveis por host, mantidas fora da documentação para não expor detalhes do ambiente.
- [playbooks](playbooks): ponto de entrada para bootstrap, formação do cluster e instalação de addons.
- [roles](roles): implementação reutilizável de cada etapa de configuração.

## Playbooks Principais

- [playbooks/bootstrap.yml](playbooks/bootstrap.yml): prepara os hosts de control-plane com common, containerd e kubernetes.
- [playbooks/init_cluster.yml](playbooks/init_cluster.yml): inicializa o nó de bootstrap do control-plane.
- [playbooks/join.yml](playbooks/join.yml): adiciona nós extras de control-plane e workers ao cluster.
- [playbooks/addons.yml](playbooks/addons.yml): aplica addons no cluster por meio do kubectl.
- [playbooks/argocd.yml](playbooks/argocd.yml): instala o Argo CD e expõe o serviço por NodePort.

## Roles

- [roles/common/tasks/main.yml](roles/common/tasks/main.yml): ajustes básicos do sistema.
- [roles/containerd/tasks/main.yml](roles/containerd/tasks/main.yml): instalação e configuração do containerd.
- [roles/kubernetes/tasks/main.yml](roles/kubernetes/tasks/main.yml): instalação das ferramentas do Kubernetes, sysctl e módulos de kernel.
- [roles/controlplane/tasks/main.yml](roles/controlplane/tasks/main.yml): inicialização do primeiro control-plane e geração dos comandos de join.
- [roles/controlplane_join/tasks/main.yml](roles/controlplane_join/tasks/main.yml): entrada dos demais nós de control-plane.
- [roles/worker/tasks/main.yml](roles/worker/tasks/main.yml): entrada dos workers no cluster.

## Fluxo Sugerido

Uma sequência típica de execução é:

1. Rodar o bootstrap dos nós base.
2. Inicializar o primeiro nó de control-plane.
3. Fazer o join dos demais nós.
4. Aplicar os addons desejados.

Exemplos:

```bash
ansible-playbook -i inventory/hosts.yml playbooks/bootstrap.yml
ansible-playbook -i inventory/hosts.yml playbooks/init_cluster.yml
ansible-playbook -i inventory/hosts.yml playbooks/join.yml
ansible-playbook -i inventory/hosts.yml playbooks/argocd.yml
```

## Observacoes

- O inventário contém dados específicos do ambiente e deve ser mantido fora de compartilhamentos públicos.
- Alguns addons dependem de um cluster já funcional e de conectividade entre os nós.
- O uso de playbooks dedicados por serviço facilita manutenção, troubleshooting e reaplicação idempotente.

