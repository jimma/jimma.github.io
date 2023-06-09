---
layout:     page
title:      "ServiceAccount And Security Context Constraint In OpenShift"
subtitle:   ""
date:       2023-06-02
author:     "Jim Ma"
header-img: "img/post-bg-06.jpg"
---
Security is always the important features for every project or product. It isn't 
exceptional for OpenShift. OpenShift not only leverages the security features from Kubernetes 
like Service Account and RBAC, but also created other security feature, for example Security
Context Constraint(SCC) to strength the pod security. In this post, I am going to discuss these
two security objects in OpenShift and explain this usage of these two features. 

## Service Account
As other server system does, the Kubernetes API server(kube-apiserver) always need to authenticate 
and authorize the request before really do something like create pod, delete deployment etc.
A service account is an identity associated with a specific resource running within an Openshift cluster. 
Service accounts are primarily used for authentication and authorization purposes, it is the entity of the RBAC(Role based Access Control)
which for processes that run in a Pod to do authenticate and authorize.
By binding different Roles to a specific service account, it associates with different API permissions to manipulate the resource in OpenShift.
With Service Accounts associated with the roles and permissions, they can be used by applications and processes to authenticate when they talk to the kube-apiserver.
There are two different Roles in Openshift or Kubernetes. They are Role and ClusterRole.
Roles are namespace-specific, while ClusterRoles are cluster-wide. 
The following example is the role which has the permission lists for a pod resource. 
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-role
  namespace: your-namespace
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

The apiGroups is empty default represents the core API group. The resources with the API group specify the exact resource
of this Role's target. The verbs are one of these which define the permission list for one of these operations:

| Verbs | Description |
| ----------- | ----------- |
| get | Allows reading/viewing a resource or its details |
|list| Allows listing multiple resources of a kind.|
|watch| Allows watching for changes to a resource.|
|create| Allows creating a new resource.|
|update| Allows updating/modifying an existing resource.|
|patch| Allows making changes to specific fields of a resource.|
|delete| Allows deleting/removing a resource.|
|deletecollection| Allows deleting multiple resources of a kind.|

A resource can only have one of these above verbs. `kubectl api-resources -o wide` can list the verbs for the resource.

Like the Role and ClusterRole, there are RoleBind and ClusterRoleBind. RoleBind are namespace-specific, while ClusterRoleBind are cluster-wide. 
They are created to associate roles with users, groups, or service accounts.
```
- apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    name: my-rolebinding
    namespace: mynamespace
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: Role
    name: pod-role
  subjects:
  - kind: ServiceAccount
    name: myserviceaccount
    namespace: mynamespace
```
This RoleBinding associates `pod-role` to ServiceAccount `myserviceaccount`. Now the subject "myserviceaccount"
will have all the permissions that `pod-role` already associated.

To use this service account, simply add the `serviceAccountName` to the pod deployment, the accesses from process or application 
running in the containers will be either allowed or denied. 
```
apiVersion: v1
kind: Pod
metadata:
  name: sa-pod
  namespace: mynamespace
spec:
  containers:
  - image: myimage:latest
    name: sa-pod-container
  serviceAccountName: myserviceaccount
```
### Security Context Constraint
Openshift Security Context Constraint is intended to limit a pod's access permission to the host.It 
mainly controls these things:
#### Container Privilege 
SCCs grants the level of privileges a pod can have. It defines if a pod can run as root user, or run privileged containers etc.

#### Resource Access
SCC defines the types of resources a pod can access. For example,  the host directory, network resources can all be controlled
by SSC. 

#### Capabilities
SCCs can be defined to allow or drop Linux capabilities in the containers. For the complete linux capability list
please check this [page](https://man7.org/linux/man-pages/man7/capabilities.7.html).

#### SELinux Policies
It defines access controls for the applications, processes, and files in a container with SeLinux Policy. 
These policies tell the pod/container what can or can’t be accessed.

There are 7 default SSCs in openshift: 
```
NAME               PRIV    CAPS   SELINUX     RUNASUSER          FSGROUP     SUPGROUP    PRIORITY   READONLYROOTFS   VOLUMES
anyuid             false   []     MustRunAs   RunAsAny           RunAsAny    RunAsAny    10         false            [configMap downwardAPI emptyDir persistentVolumeClaim projected secret]
hostaccess         false   []     MustRunAs   MustRunAsRange     MustRunAs   RunAsAny    <none>     false            [configMap downwardAPI emptyDir hostPath persistentVolumeClaim projected secret]
hostmount-anyuid   false   []     MustRunAs   RunAsAny           RunAsAny    RunAsAny    <none>     false            [configMap downwardAPI emptyDir hostPath nfs persistentVolumeClaim projected secret]
hostnetwork        false   []     MustRunAs   MustRunAsRange     MustRunAs   MustRunAs   <none>     false            [configMap downwardAPI emptyDir persistentVolumeClaim projected secret]
node-exporter      false   []     RunAsAny    RunAsAny           RunAsAny    RunAsAny    <none>     false            [*]
nonroot            false   []     MustRunAs   MustRunAsNonRoot   RunAsAny    RunAsAny    <none>     false            [configMap downwardAPI emptyDir persistentVolumeClaim projected secret]
privileged         true    [*]    RunAsAny    RunAsAny           RunAsAny    RunAsAny    <none>     false            [*]
restricted         false   []     MustRunAs   MustRunAsRange     MustRunAs   RunAsAny    <none>     false            [configMap downwardAPI emptyDir persistentVolumeClaim projected secret]
```

The last one `restricted` is the default SSC for a pod in OpenShift. This SSC :
```
Ensures that pods cannot run as privileged

Ensures that pods cannot mount host directory volumes

Requires that a pod is run as a user in a pre-allocated range of UIDs

Requires that a pod is run with a pre-allocated MCS label

Allows pods to use any FSGroup
```

Your customized SSC can be created with a yaml file like:
```
---
kind: SecurityContextConstraints
apiVersion: security.openshift.io/v1
metadata:
  name: scc-demo
allowPrivilegedContainer: true
runAsUser:
  type: RunAsAny
seLinuxContext:
  type: RunAsAny
fsGroup:
  type: RunAsAny
supplementalGroups:
  type: RunAsAny
requiredDropCapabilities:
  - SYS_CHROOT
```

SCCs can be associated with specific service accounts.When the pod is defined the service account 
name, it will automatically have the associated SSC :
```
oc adm policy add-scc-to-user <scc-name> -z <service-account-name>
```

### Summary
OpenShift is the enterprise Kubernets distribution. It is enriched with
many security and other development features. Whenever you paly with OpenShift 
and run into some security related issues, please look at the Service Account 
and Security Context Constraints settings for the default or the deployment. 
Hope this post can explain some basic thing about Service Account and Security Context
Constraints and provide a little help when you deploy the application to the OpenShift Cluster.


### References
https://kubernetes.io/docs/reference/access-authn-authz/rbac/
https://docs.openshift.com/container-platform/4.13/authentication/managing-security-context-constraints.html
https://daein.medium.com/how-to-work-the-security-context-constraints-scc-on-ocp4-607175e3da34
https://cloud.redhat.com/blog/managing-sccs-in-openshift















