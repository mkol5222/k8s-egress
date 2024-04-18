# Kubernetes egress

Follow `MULTIPASS.md` networking section to setup new Hyper-V switch called `multipass` with dedicated subnet.
This is done in Administrator's PowerShell.

```shell
# verify vSwitch multipass exists and is assigned with 10.38.0.0/24 subnet
Get-NetIPAddress -InterfaceAlias "vEthernet (multipass)" | Select-Object IPAddress, PrefixLength

# you may check cloud-init-node1.yml to undertand MicroK8s setup and new eth1 interface with IP 10.38.0.101
cat ./cloud-init-node1.yml
code ./cloud-init-node1.yml

# launch first cluster node
multipass launch -v -n node1 --cloud-init cloud-init-node1.yml -m 6G -d 20G -c 4 --network name=multipass,mode=manual,mac=52:54:00:f1:94:fa

# once you got back to the prompt, check the status of the node
# some checks 
multipass exec node1 -- ip a show dev eth1 # fixed IP address
multipass exec node1 -- microk8s status -w # will wait in case cluster is not ready
multipass exec node1 -- sudo tail -f /var/log/cloud-init-output.log # did cloud-init work well?
multipass exec node1 -- microk8s.kubectl get po -A --watch # monitors pods in all ns


# now we have cluster node1 running
multipass shell node1

# continue on node1
k get no -o wide
k get po -A -o wide

# container image build process
# https://github.com/mkol5222/squid-openssl-docker/blob/main/.github/workflows/container-package.yml
# results to image e.g.  ghcr.io/mkol5222/mkol5222/squid-openssl-docker:2024-04-18

microk8s ctr i pull ghcr.io/mkol5222/squid-openssl-docker:2024-04-18

# create a namespace
k create ns squid
# execute squid add-hoc
k run squid --image=ghcr.io/mkol5222/squid-openssl-docker:2024-04-18 -n squid --restart=Never --port=3128 --expose
# check the pod
k get po -n squid
k describe po squid -n squid
k describe svc squid -n squid

# try connect via squid proxy
k run client --image=nginx -it --rm --restart=Never -- curl --proxy http://squid.squid.svc.cluster.local:3128 -k -vvv https://httpbin.org/headers

# proxy env variables
k run client --image=nginx -it --rm --restart=Never --env https_proxy=http://squid.squid.svc.cluster.local:3128 -- curl  -k -vvv https://httpbin.org/headers


# output should contain
#     "X-Own-Forwarded-For": "10.1.166.139"
# which is real IP of the client Pod
#   and certificate issue CN=Forward Proxy
# because HTTPS inspection is happeing by the proxy

# remove
k -n squid delete pod squid
k -n squid delete svc squid
k delete ns squid
```