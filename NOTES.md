# ODH Operator Testing

## Summary

The ODH operator at it's core is a re-branded `kubeflow` [operator](https://operatorhub.io/operator/kubeflow).

The primary differentiators are:

- A default `kfDef` custom resource
- The [odh manifests](https://github.com/opendatahub-io/odh-manifests/)
- The [odh contrib manifests](https://github.com/opendatahub-io-contrib/odh-contrib-manifests)

### Cluster State: Before

```
oc api-resources > api-before.txt
oc get crd > crd-before.txt
oc get sub -A > sub-before.txt
oc get csv -A > csv-before.txt
```

### ODH Install

```
# install odh operator
oc apply -k https://github.com/redhat-cop/gitops-catalog/opendatahub-operator/operator/overlays/stable

# new project
oc new-project odh-testing

# default kfdef
cat << YAML | oc apply -f -
kind: KfDef
apiVersion: kfdef.apps.kubeflow.org/v1
metadata:
  name: opendatahub
spec:
  applications:
    - kustomizeConfig:
        repoRef:
          name: manifests
          path: odh-common
      name: odh-common
    - kustomizeConfig:
        repoRef:
          name: manifests
          path: odh-dashboard
      name: odh-dashboard
    - kustomizeConfig:
        repoRef:
          name: manifests
          path: prometheus/cluster
      name: prometheus-cluster
    - kustomizeConfig:
        repoRef:
          name: manifests
          path: prometheus/operator
      name: prometheus-operator
    - kustomizeConfig:
        repoRef:
          name: manifests
          path: grafana/cluster
      name: grafana-cluster
    - kustomizeConfig:
        repoRef:
          name: manifests
          path: grafana/grafana
      name: grafana-instance
    - kustomizeConfig:
        repoRef:
          name: manifests
          path: odh-notebook-controller
      name: odh-notebook-controller
    - kustomizeConfig:
        repoRef:
          name: manifests
          path: notebook-images
      name: notebook-images
    - kustomizeConfig:
        overlays:
          - odh-model-controller
        repoRef:
          name: manifests
          path: model-mesh
      name: model-mesh
    - kustomizeConfig:
        overlays:
          - metadata-store-mariadb
          - ds-pipeline-ui
          - object-store-minio
          - default-configs
        repoRef:
          name: manifests
          path: data-science-pipelines
      name: data-science-pipelines
  repos:
    - name: manifests
      uri: 'https://github.com/opendatahub-io/odh-manifests/tarball/v1.4.1'
YAML

# NOTE: many accidentally install the kfdef in openshift-operators
# NOTE: the default ns in the console changes to openshift-operators

# NOTE: kfdef opendatahub does not display anything in the `resources` tab indicating a lack of labeling

# BUG: operator installs `grafana` and `prometheus` operators with kfDef
# BUG: removes sub, not csv

# NOTE: `The group odh-admins no longer exists in OpenShift and has been removed from the selected group list.`
# BUG: PVCs created for notebook are set to `20` (bytes) not `20Gi` (gigabytes)
```

### Cluster State: After

```
# see what crds got installed
oc api-resources > api-after.txt
oc get crd > crd-after.txt

diff -u api-before.txt api-after.txt > api-diff.txt
diff -u crd-before.txt crd-after.txt > crd-diff.txt

grep '^+' api-diff.txt
grep '^+' crd-diff.txt
```

### ODH Uninstall

```
# source for functions
source odh-helper.sh

# choose function
install_odh
destroy_odh
```
