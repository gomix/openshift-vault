RHCOS has no direct support for sosreport ,  you must use a suporting container using `toolbox`.

## sosreport on OCP node with RHEL 9.6

```
%> oc debug node/crc
Starting pod/crc-debug-jrb2b ...
To use host binaries, run `chroot /host`. Instead, if you need to access host namespaces, run `nsenter -a -t 1`.
Pod IP: 192.168.126.11
If you don't see a command prompt, try pressing enter.
sh-5.1# chroot /host   
sh-5.1# cat /etc/redhat-release 
Red Hat Enterprise Linux release 9.6 (Plow)
sh-5.1# toolbox
Trying to pull registry.redhat.io/rhel9/support-tools:latest...
Getting image source signatures
Checking if image destination supports signatures
Copying blob 8669339e0390 done   | 
Copying blob 57759c315739 done   | 
Copying config 2d2fd8e640 done   | 
Writing manifest to image destination
Storing signatures
2d2fd8e640ebfcb0a0834c043576bc8d6f2386fd70f155d64666399efb8a30ed
Spawning a container 'toolbox-root' with image 'registry.redhat.io/rhel9/support-tools'
Detected RUN label in the container image. Using that as the default...
ba69cfec97b947d4fc6c0931ff33a828a1dbe94abc93a0c0b7896482f5b4c389
toolbox-root
Container started successfully. To exit, type 'exit'.
[root@crc /]# 
[root@crc /]# sos report

sos report (version 4.10.2)

This command will collect diagnostic and configuration information from
this Red Hat Enterprise Linux system and installed applications.

An archive containing the collected information will be generated in
/host/var/tmp/sos.hpcxnygd and may be provided to a Red Hat support
representative.

Any information provided to Red Hat will be treated in accordance with
the published support policies at:

        Distribution Website : https://www.redhat.com/
        Commercial Support   : https://access.redhat.com/

The generated archive may contain data considered sensitive and its
content should be reviewed by the originating organization before being
passed to any third party.

No changes will be made to system configuration.

Press ENTER to continue, or CTRL-C to quit.
Optionally, please enter the case id that you are generating this report for: 04453169                                                   
 Setting up archive ...                                                                                                                                             
 Setting up plugins ...                                                 
 ...
 [plugin:systemd] skipped command 'systemd-resolve --status': required services missing: systemd-resolved.  
[plugin:systemd] skipped command 'systemd-resolve --statistics': required services missing: systemd-resolved.  
 Running plugins. Please wait ...

  Finishing plugins              [Running: networking]                                    ]]
 Plugin networking timed out


Creating compressed archive...

Your sos report has been generated and saved in:
        /host/var/tmp/sosreport-crc-04453169-2026-05-29-hrhzthm.tar.xz

 Size   16.58MiB
 Owner  root
 sha256 53f4d975071dee3cc7d49fb82eec74d80266a928b759d88857ff1b353e7451fa

Please send this file to your support representative.

[root@crc /]# 
```

Para OpenShift

```
sos report \
-e openshift \
-e openshift_ovn \
-e openvswitch \
-e podman \
-e crio \
-k crio.all=on \
-k crio.logs=on \
-k podman.all=on \
-k podman.logs=on \
-k networking.ethtool-namespaces=off \
--all-logs \
--plugin-timeout=600
```