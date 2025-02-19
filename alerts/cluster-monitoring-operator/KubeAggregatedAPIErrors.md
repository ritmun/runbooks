# KubeAggregatedAPIErrors

## Meaning

[This alert][KubeAggregatedAPIErrors] is triggered when multiple calls to the
aggregated API of OpenShift fail over a certain period.

## Impact

Errors on the aggregated API can result in the unavailability of some OpenShift
services.

## Diagnosis

The alert should contain information about the affected API and the scope of the
impact.

```text
 - alertname = KubeAggregatedAPIErrors
 - name = v1.packages.operators.coreos.com
 - namespace = default
...
 - message = Kubernetes aggregated API v1.packages.operators.coreos.com/default has reported errors. It has appeared unavailable 5 times averaged over the past 10m.
```

## Mitigation

### Check the APIs status checks are on True

Currently, there are at least four aggregated APIs in an OpenShift Cluster. The
API on the `openshift-apiserver` namespace, the prometheus-adapter on the
namespace `openshift-monitoring`, the package-server service in the
`openshift-operator-lifecycle-manager` namespace, and the API on the
`openshift-oauth-apiserver` namespace. However, it makes sense to check the
availability of all APIs.

To get a list of `APIServices` and their backing aggregated APIs, use the
following command:

```console
$ oc get apiservice
```

The `SERVICE` column notes here the aggregated API name. The availability status
for every listed API should be `True`. A `False` means that requests for that
API service, API server pods, or resources belonging to that apiGroup failed
many times during the last minutes.

Fetch the pods that serve the unavailable API. E.g.: for
`openshift-apiserver/api` use the following command:

```console
$ oc get pods -n openshift-apiserver
```

When their status is not `Running`, check the logs for more details. As these
pods are controlled by a deployment, they can be restart when they are not
answering to requests anymore.

### Check the authentication certificates of the aggregated API

Make sure the certificates are up to date and still valid. Use:

```console
$ oc get configmaps -n kube-system extension-apiserver-authentication
```

You can save those certificates into a file and use the following command to
check the end dates:

```console
$ openssl x509 -noout -enddate -in {myfile_with_certs.crt}
```

Those certificates are used by the aggregated APIs to validate requests. For the
case, they are expired check [here][cert] how to add a new one.

[cert]: https://docs.openshift.com/container-platform/latest/security/certificates/api-server.html
[KubeAggregatedAPIErrors]: https://github.com/openshift/cluster-monitoring-operator/blob/1824f9c9a39f54734298dd10e5d20d42c8247995/assets/control-plane/prometheus-rule.yaml#L399-L408
