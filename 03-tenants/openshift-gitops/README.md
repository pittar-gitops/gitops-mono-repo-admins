# Tenant: openshift-gitops

This contains an extended `ClusterRole` for the default Argo CD instance that is installed by the OpenShift GitOps operator.

This works round a current limitation that keeps the default Argo CD instance from creating extra Argo CD instances for developers.  This issue is tracked by [GITOPS-1247](https://issues.redhat.com/browse/GITOPS-1247) and will hopefully be fixed in a future release.