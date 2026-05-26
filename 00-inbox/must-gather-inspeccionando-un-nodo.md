Me interesa ver cómo está configurado...

## Case 1

```bash title="Inspeccionando Nodo"
$ omc get node cgeocpworkxxs01 -o wide
NAME              STATUS   ROLES    AGE   VERSION   INTERNAL-IP    EXTERNAL-IP    OS-IMAGE                                                KERNEL-VERSION                 CONTAINER-RUNTIME
cgeocpworkxxs01   Ready    worker   3y    v1.33.6   172.24.160.1   172.24.160.1   Red Hat Enterprise Linux CoreOS 9.6.20260128-1 (Plow)   5.14.0-570.83.1.el9_6.x86_64   cri-o://1.33.8-3.rhaos4.20.git9658698.el9

$ omc get node cgeocpworkxxs01 -o json | jq '.metadata.annotations["machineconfiguration.openshift.io/currentConfig"]' -r
rendered-worker-595cfac0373a308a0899a812c38406e2

$ omc get node cgeocpworkxxs01 -o json | jq '.metadata.annotations["machineconfiguration.openshift.io/desiredConfig"]' -r
rendered-worker-595cfac0373a308a0899a812c38406e2

$ omc get mcp worker
NAME     CONFIG                                             UPDATED   UPDATING   DEGRADED   MACHINECOUNT   READYMACHINECOUNT   UPDATEDMACHINECOUNT   DEGRADEDMACHINECOUNT   AGE
worker   rendered-worker-595cfac0373a308a0899a812c38406e2   True      False      False      17             17                  17                    0                      6y

$ omc get mcp worker -o json | jq .spec.configuration.name -r
rendered-worker-595cfac0373a308a0899a812c38406e2

$ omc get mcp worker -o json | jq -r .spec.configuration.source[].name 
00-override-worker-generated-crio-default-container-runtime
00-override-worker-generated-crio-default-ulimits
00-worker
01-worker-container-runtime
01-worker-kubelet
50-worker-auto-sizing-disabled
50-worker-chrony-configuration
50-worker-custom-ca
97-worker-generated-kubelet
98-worker-generated-kubelet
99-worker-generated-crio-add-inheritable-capabilities
99-worker-generated-crio-capabilities
99-worker-generated-crio-seccomp-use-default
99-worker-generated-kubelet
99-worker-generated-registries
99-worker-ssh
99-worker-udev-vmware-configuration
crio-pull-timeout-custom

$ omc get mc 50-worker-chrony-configuration
NAME                             GENERATEDBYCONTROLLER   IGNITIONVERSION   AGE
50-worker-chrony-configuration                           2.2.0             6y

$ omc get mc 50-worker-chrony-configuration -o json | jq .spec
{
  "config": {
    "ignition": {
      "config": {},
      "security": {
        "tls": {}
      },
      "timeouts": {},
      "version": "2.2.0"
    },
    "networkd": {},
    "passwd": {},
    "storage": {
      "files": [
        {
          "contents": {
            "source": "data:text/plain;charset=utf-8;base64,c2VydmVyIG50cGNwZC5jYWl4YWdhbGljaWEuY2cgaWJ1cnN0IG1heHBvbGwgMTAKc2VydmVyIG50cGNyLmNhaXhhZ2FsaWNpYS5jZyBpYnVyc3QgbWF4cG9sbCAxMApzZXJ2ZXIgZXhhY3RpdmVkaXMxLmNhaXhhZ2FsaWNpYS5jZyBpYnVyc3QgbWF4cG9sbCAxMApzZXJ2ZXIgZXhhY3RpdmVkaXM0LmNhaXhhZ2FsaWNpYS5jZyBpYnVyc3QgbWF4cG9sbCAxMApkcmlmdGZpbGUgL3Zhci9saWIvY2hyb255L2RyaWZ0Cm1ha2VzdGVwIDEuMCAzCnJ0Y3N5bmMKbG9nZGlyIC92YXIvbG9nL2Nocm9ueQoK",
            "verification": {}
          },
          "filesystem": "root",
          "mode": 420,
          "path": "/etc/chrony.conf"
        }
      ]
    }
  },
  "osImageURL": ""
}

$ omc get mc 50-worker-chrony-configuration -o json | jq -r .spec.config.storage.files[].contents.source | sed -E 's/^data:text\/plain;charset=utf-8;base64,//' | base64 -d
server ntpcpd.caixagalicia.cg iburst maxpoll 10
server ntpcr.caixagalicia.cg iburst maxpoll 10
server exactivedis1.caixagalicia.cg iburst maxpoll 10
server exactivedis4.caixagalicia.cg iburst maxpoll 10
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
logdir /var/log/chrony

```