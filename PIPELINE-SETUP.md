# Pipeline Setup

Add the helm slave:

```
git clone https://github.com/redhat-cop/containers-quickstarts.git
cd containers-quickstarts/jenkins-slaves/jenkins-slave-helm
oc process -f ../../.openshift/templates/jenkins-slave-generic-template.yml \
    -p NAME=jenkins-slave-helm \
    -p SOURCE_CONTEXT_DIR=jenkins-slaves/jenkins-slave-helm \
    -p DOCKERFILE_PATH=Dockerfile \
    | oc create -n openshift -f -
```