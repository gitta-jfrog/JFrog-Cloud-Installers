# JFrog Unified Platform On Openshift 

JFrog Unified Platform on Openshift official support is for the operator deployment only through Openshift's Operatorhub.

## 2024 Update

JFrog now provides full support for OpenShift Platform by using [JFrog Helm Charts](https://github.com/jfrog/charts/tree/master/stable). 
Using Helm Charts is the recommended method for new deployments on the OpenShift Platforms.

For more information, please visit the [JFrog Installation and Setup Documentation](https://jfrog.com/help/r/jfrog-installation-setup-documentation/ha-installation-on-openshift-using-artifactory-chart).

## Repo Layout

| Folder                          | Purpose                                                 |
|---------------------------------|---------------------------------------------------------|
| helm                            | Contains the Openshift Helm charts used by the Operator |
| helm/openshift-artifactory-ha   | Openshift Artifactory HA helm chart                     |
| helm/openshift-xray             | Openshift Xray helm chart                               |
| helm/openshift-pipelines        | Opneshift Pipelines helm chart                          | 
| operator                        | Contains the Openshift certified operators code base    |
| operator/artifactory-ha-operator| Artifactory Enterprise Operator                         |
| operator/xray-operator          | Xray Enterprise Operator                                |
| operator/pipeline-operator      | Pipelines Operator (Beta)                               |

## How to install?

You can find the Redhat certified Operators in the Operatorhub in your Openshift web console.

You will need to be an administrator of your Openshift cluster to install our operator.

Additional steps can be found at [JFrog Partner support wiki](https://www.jfrog.com/confluence/display/JFROG/JFrog+Partner+Integrations#JFrogPartnerIntegrations-redhatopenshift]).

## Security Context Constraints

The `restricted` security context constraint will prevent the helm or operator from deploying into Openshift on most namespaces.

To enable either the helm chart or operator to deploy into your Openshift cluster access to the `anyuid` security context constraint will need to be apply to the relevant service account in the associated namespace.

Below is an example of applying the `anyuid` scc to the service account `openshiftartifactoryha-artifactory-ha` in the namespace `artifactory`:

`oc adm policy add-scc-to-user anyuid -z openshiftartifactoryha-artifactory-ha -n artifactory`

Once the `anyuid` scc has been applied to the correct service accounts the helm charts or operators will deploy into your Openshift cluster.

## Custom User or Group Ids

The images uploaded to `registry.redhat.connect.com` that the helm charts and operators use have been modified from the standard docker images available at `docker.bintray.io`

These images have been customized to run in the Openshift user id and group id range of `1000720000/10000`

If you need to use another custom user id and/or group id range you can change the `uid` and `gid` values in `values.yaml` of the relevant helm chart or operator yaml deployment.

## No Root Environments

Some environments do not allow root. In these scenarios users can remove the `customInitContainersBegin` from the example values.yaml below:

````text
    customInitContainersBegin: |
      - name: "prepare-uid-persistent-volume"
        image: "{{ .Values.initContainerImage }}"
        imagePullPolicy: "{{ .Values.artifactory.image.pullPolicy }}"
        command:
          - 'sh'
          - '-c'
          - >
            chown -Rv {{ .Values.artifactory.uid }}:{{ .Values.artifactory.uid }} {{ .Values.artifactory.persistence.mountPath }}
        securityContext:
            runAsUser: 0
        volumeMounts:
          - mountPath: "{{ .Values.artifactory.persistence.mountPath }}"
            name: volume
````

Once this has been removed there is no other root user permissions are required to deploy into Openshift.

## Why are there different helm charts?

The charts in the helm folder are used specifically to create the helm based operator for the certification process to enable it into the Openshift Operatorhub as a certified operator.

The `values.yaml` contained in those relevant charts have been modified to work in Redhat Openshift. The base chart however has not been changed only made a sub-chart.

Helm users can reference the `values.yaml` to modify their own deployments to work with Openshift.

## Contributing
Please read [CONTRIBUTING.md](JFrog-Cloud-Installers/Openshift4/CONTRIBUTING.md) for details on our code of conduct, and the process for submitting pull requests to us.

## Versioning
We use [SemVer](http://semver.org/) for versioning. For the versions available, see the [tags on this repository](https://github.com/jfrog/JFrog-Cloud-Installers/tags).

## Contact

Github issues are the preferred way to communicate with the team. The team is notified via Slack when a new issue is created.
