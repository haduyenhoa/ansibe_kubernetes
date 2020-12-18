# ansibe_kubernetes
Use Ansibe to setup kubernetes cluster

- Prepare Linux machine, can connect via SSH, using .pem file. The pem file must have the permission right = 600: `chmod 600 ...`
- Update inbound network settings as described in: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#before-you-begin
- do not specify "ip" in hosts.yaml to avoid issue with ipv6 address

Apply the playbook:

```
Activate the venv:
$ source ./vietsearch/env/bin/activate

```

declare -a IPS=(18.207.129.157 3.235.143.75 3.226.244.135 3.236.114.214)

to avod host checking:

`export ANSIBLE_HOST_KEY_CHECKING=false`


```
# Deploy Kubespray with Ansible Playbook - run the playbook as root
# The option `--become` is required, as for example writing SSL keys in /etc/,
# installing packages and interacting with various systemd daemons.
# Without --become the playbook will fail to run!
$ ansible-playbook -i vietsearch-cluster/inventory/vietsearch/hosts.yaml  --become --become-user=root ./kubespray/cluster.yml
```

Scaling, upgradeing nodes:
https://github.com/kubernetes-sigs/kubespray/blob/master/docs/nodes.md


- Dashboard token:
> kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')

Dashboard address: 
https://3.237.4.177:6443/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/cluster?namespace=default


1. To get admin.conf

ssh $USERNAME@$IP_CONTROLLER_0
USERNAME=$(whoami)
sudo chown -R $USERNAME:$USERNAME /etc/kubernetes/admin.conf
exit

scp $USERNAME@$IP_CONTROLLER_0:/etc/kubernetes/admin.conf kubespray-do.conf


then:
`export KUBECONFIG=$PWD/kubespray-do.conf`


`kubectl get nodes`


2. To start dashboard
- add service account & apply cluster role binding
`$ kubectl -n vs-infra apply -f inventory/vietsearch/app/service-account.yaml`
`$ kubectl -n vs-infra apply -f inventory/vietsearch/app/cluster-role-binding`
- get token:

`$ kubectl -n vs-infra describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')`

- start proxy
`$ kubectl -n vs-infra proxy`
Dashboard:
http://127.0.0.1:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login

-> need to change to namespace vs-infra once logged in
