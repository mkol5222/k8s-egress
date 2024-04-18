# Kubernetes visualization

```shell
# node1 ready, use it
multipass shell node1
# install kexp
curl -Ls https://github.com/iximiuz/kexp/releases/latest/download/kexp_linux_amd64.tar.gz | tar xvz
sudo mv kexp /usr/local/bin

# grab config
mkdir ~/.kube; microk8s config > ~/.kube/config
kexp --host 0.0.0.0

# now reachable from your browser
# http://node1.mshome.net:5173/ui/

# try something like
k create ns app1
k create deployment web --image=nginx -n app1 --replicas 3
k expose deployment web --port=80 --target-port=80 -n app1
# look at it in the browser inside kexps
```