# minishift_no_root_solution

### login to minishift
```
oc login $(minishift ip):8443
```

###Create a new project to test
```
oc new-project test
```

#login to projet
```
oc login -u system:admin -n test
```

#Grab the cluster ip address provided for openshift internal registry
#The user must have the system:registry role. To add this role:
```
oc adm policy add-role-to-user system:registry admin
```
#Have the admin role for the project associated with the Docker operation. For example, if accessing images in the global openshift project:
```
oc adm policy add-role-to-user admin admin -n test
```

#For writing or pushing images, for example when using the docker push command, the user must have the system:image-builder role. To add this role:
```
oc adm policy add-role-to-user system:image-builder admin
```
#Grab the cluster ip address provided for openshift internal registry
```
oc get svc -n default | grep registry #172.30.1.1
```

#login to internal docker reg
```
docker login -p $(oc whoami -t) $(minishift openshift registry)
```
#I don't konw why but this is necessary
```
eval $(minishift docker-env)
```
#Create the docker file

```
mkdir smiletomcat
vim smiletomcat/Dockerfile
```
```console

FROM tomcat:8.0
RUN set -xe; \
    useradd -r -u 1001 -g 0 openshiftroot;
RUN set -xe; \
    chown -R openshiftroot /usr/local; \
    chgrp -R 0  /usr/local; \
    chmod -R g+rwX  /usr/local;
USER 1001
```

#build docker image
```
docker build smiletomcat --tag 172.30.1.1:5000/test/smiletomcat
```
#Push image to internal registry
```
docker push 172.30.1.1:5000/test/smiletomcat
```
#Create a new app 'myapp' using the pushed image
```
oc new-app test/smiletomcat --name=tomcatsmile
```


##References : 
<https://github.com/debianmaster/Notes/wiki/How-to-push-docker-images-to-openshift-internal-registry-and-create-application-from-it.>
<https://docs.openshift.org/latest/minishift/openshift/openshift-docker-registry.html>
<https://docs.openshift.com/container-platform/3.3/install_config/registry/accessing_registry.html>
