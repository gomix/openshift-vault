---
tags:
  - cli
  - oc
---
# oc describe

Show details of a specific resource or group of resources.

```
; oc describe TYPE NAME_PREFIX
```

## Usage examples

### Describe image stream tag

Perhaps you want to know the exposed ports of an image, of course you will get a lot more info!

```bash
%> oc describe istag httpd:latest -n openshift
Image Name:	sha256:686fea6c9e04ecb0ce2c5609e4dea4125a046dfdf451a164e5dd84293d7d6e30
Docker Image:	image-registry.openshift-image-registry.svc:5000/openshift/httpd@sha256:686fea6c9e04ecb0ce2c5609e4dea4125a046dfdf451a164e5dd84293d7d6e30
Name:		sha256:686fea6c9e04ecb0ce2c5609e4dea4125a046dfdf451a164e5dd84293d7d6e30
Created:	2 hours ago
Description:	Build and serve static content via Apache HTTP Server (httpd) 2.4 on UBI 8. For more information about using this builder image, including OpenShift considerations, see https://github.com/sclorg/httpd-container/blob/master/2.4/README.md.
		
		WARNING: By selecting this tag, your application will automatically update to use the latest version available on OpenShift, including major version updates.
		
Annotations:	iconClass=icon-httpd
		image.openshift.io/dockerLayersOrder=ascending
		openshift.io/display-name=Apache HTTP Server 2.4 (Latest)
		openshift.io/provider-display-name=Red Hat, Inc.
		sampleRepo=https://github.com/sclorg/httpd-ex.git
		tags=builder,httpd
		version=2.4
Image Size:	104.4MB in 3 layers
Layers:		77.95MB	sha256:e82314ec5008caa851098908a31d4380254e4e7c30cae3d8d8ca899bae86c5bc
		17.65MB	sha256:e8974e45da9467efb40118f00a135e6f68374f007c81e3e912b8401559539f67
		8.823MB	sha256:a4419ad58e77dc4cf4a114f3a55be05a062d66aa396e3b4814cc7032aab5e045
Image Created:	11 hours ago
Author:		<none>
Arch:		amd64
Entrypoint:	container-entrypoint
Command:	/usr/bin/run-httpd
Working Dir:	/opt/app-root/src
User:		1001
Exposes Ports:	8080/tcp, 8443/tcp
Docker Labels:	architecture=x86_64
		build-date=2026-09-14T00:16:59Z
		com.redhat.component=httpd-24-container
		com.redhat.license_terms=https://www.redhat.com/en/about/red-hat-end-user-license-agreements#UBI
		cpe=cpe:/a:redhat:enterprise_linux:8::appstream
		description=Apache httpd 2.4 available as container, is a powerful, efficient, and extensible web server. Apache supports a variety of features, many implemented as compiled modules which extend the core functionality. These can range from server-side programming language support to authentication schemes. Virtual hosting allows one Apache installation to serve many different Web sites.
		distribution-scope=public
		io.buildah.version=1.44.0
		io.k8s.description=Apache httpd 2.4 available as container, is a powerful, efficient, and extensible web server. Apache supports a variety of features, many implemented as compiled modules which extend the core functionality. These can range from server-side programming language support to authentication schemes. Virtual hosting allows one Apache installation to serve many different Web sites.
		io.k8s.display-name=Apache httpd 2.4
		io.openshift.expose-services=8080:http,8443:https
		io.openshift.s2i.scripts-url=image:///usr/libexec/s2i
		io.openshift.tags=builder,httpd,httpd-24
		io.s2i.scripts-url=image:///usr/libexec/s2i
		maintainer=SoftwareCollections.org <sclorg@redhat.com>
		name=rhel8/httpd-24
		org.opencontainers.image.created=2026-09-14T00:16:59Z
		org.opencontainers.image.revision=2e0d978279f37472a1722e7cee7d58d29a4d43f7
		release=1789344980
		summary=Platform for running Apache httpd 2.4 or building httpd-based application
		url=https://catalog.redhat.com/en/search?searchType=containers
		usage=s2i build https://github.com/sclorg/httpd-container.git --context-dir=examples/sample-test-app/ rhel8/httpd-24 sample-server
		vcs-ref=2e0d978279f37472a1722e7cee7d58d29a4d43f7
		vcs-type=git
		vendor=Red Hat, Inc.
		version=1
Environment:	container=oci
		STI_SCRIPTS_URL=image:///usr/libexec/s2i
		STI_SCRIPTS_PATH=/usr/libexec/s2i
		APP_ROOT=/opt/app-root
		HOME=/opt/app-root/src
		PATH=/opt/app-root/src/bin:/opt/app-root/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
		PLATFORM=el8
		HTTPD_VERSION=2.4
		SUMMARY=Platform for running Apache httpd 2.4 or building httpd-based application
		DESCRIPTION=Apache httpd 2.4 available as container, is a powerful, efficient, and extensible web server. Apache supports a variety of features, many implemented as compiled modules which extend the core functionality. These can range from server-side programming language support to authentication schemes. Virtual hosting allows one Apache installation to serve many different Web sites.
		HTTPD_CONTAINER_SCRIPTS_PATH=/usr/share/container-scripts/httpd/
		HTTPD_APP_ROOT=/opt/app-root
		HTTPD_CONFIGURATION_PATH=/opt/app-root/etc/httpd.d
		HTTPD_MAIN_CONF_PATH=/etc/httpd/conf
		HTTPD_MAIN_CONF_MODULES_D_PATH=/etc/httpd/conf.modules.d
		HTTPD_MAIN_CONF_D_PATH=/etc/httpd/conf.d
		HTTPD_TLS_CERT_PATH=/etc/httpd/tls
		HTTPD_VAR_RUN=/var/run/httpd
		HTTPD_DATA_PATH=/var/www
		HTTPD_DATA_ORIG_PATH=/var/www
		HTTPD_LOG_PATH=/var/log/httpd

```

