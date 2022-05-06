WSL CLI
cd -
go back to latest path in WSL2

Helm CLI

helm ls -A
All name space

k get namespaces
Get namespaces with kubernetes

k create namespace { NAMESPACE-NAME }
Create namespace as NAMESPACE-NAME with kubectl

helm repo add { REPOSITORY-NAME } { REPOSITORY-NAME }/{ CHART-NAME }
Add new repository by url of repository

helm repo ls
Get list of helm repository

helm repo update
Update the version information of the helm repository

helm show values { CHART-NAME } > { FILENAME }.yaml
Show values of helm

helm ls --namespace { NAMESPACE-NAME }
Get list of release with namespace

k get ingress -n { NAMESPACE-NAME  }
get Ingress with namespace by kubectl

k get secrets -n { NAMESPACE-NAME }
get secret of namespace with kubectl

*Then access to phpldapadmin with host of ingress :
e.g. ldapadmin.selab.cloud

access LoginDn :
type to bindDn

*Access password to with lens > secrets. it encoded with Base64.
*Could get the decoded (raw) password in Lens.
