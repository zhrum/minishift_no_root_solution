# minishift_no_root_solution
This file explains how to run a docker container built from a dockerfile into minishift...
* Without pushing the image to the official docker hub (everyone has his secrets!!)
* Without running the docker as root (as openshift prefers)

Login to minishift
```
oc login $(minishift ip):8443
```

Create a new project to test
```
oc new-project test
```

Login to projet
```
oc login -u system:admin -n test
```

Grab the cluster ip address provided for openshift internal registry
The user must have the system:registry role. To add this role:
```
oc adm policy add-role-to-user system:registry coucou
```
Have the admin role for the project associated with the Docker operation. For example, if accessing images in the global openshift project:
```
oc adm policy add-role-to-user admin coucou -n test
```

For writing or pushing images, for example when using the docker push command, the user must have the system:image-builder role. To add this role:
```
oc adm policy add-role-to-user system:image-builder coucou
```
Grab the cluster ip address provided for openshift internal registry
```
oc get svc -n default | grep registry #172.30.1.1
```
Login as user "coucou"
```
oc login -u coucou
```

Login to internal docker reg
```
docker login -p $(oc whoami -t) $(minishift openshift registry)
```
I don't konw why but this is necessary
```
eval $(minishift docker-env)
```
Create the docker file

```
mkdir smiletomcat
vim smiletomcat/Dockerfile
```
```console

FROM tomcat:8.0
RUN set -xe; \
    useradd -r -u 1001 -g 0 openshiftuser;
RUN set -xe; \
    chown -R openshiftuser /usr/local; \
    chgrp -R 0  /usr/local; \
    chmod -R g+rwX  /usr/local;
USER 1001
```

Build docker image from dockerfile
```
docker build smiletomcat --tag $(oc get svc -n default | grep registry):5000/test/smiletomcat
```
Push image to internal registry
```
docker push $(oc get svc -n default | grep registry):5000/test/smiletomcat
```
Create a new app 'myapp' using the pushed image
```
oc new-app test/smiletomcat --name=tomcatsmile
```


## References
* <https://github.com/debianmaster/Notes/wiki/How-to-push-docker-images-to-openshift-internal-registry-and-create-application-from-it.>
* <https://docs.openshift.org/latest/minishift/openshift/openshift-docker-registry.html>
* <https://docs.openshift.com/container-platform/3.3/install_config/registry/accessing_registry.html>
