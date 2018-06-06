# 3scale-amp-install 2.2

https://access.redhat.com/documentation/en-us/red_hat_3scale/2.2/html-single/infrastructure/#onpremises-installation

Pre-req
====================
install openshift. Make sure you have PVs
https://access.redhat.com/documentation/en-us/red_hat_3scale/2.2/html-single/infrastructure/#configure_nodes_and_entitlements

Get the template
====================
subscription-manager register --username=$RHSM_USER --password=$RHSM_PASS
 

````
if [ -z "$EMP_POOL_ID" ]
then
    EMP_POOL_ID=$(subscription-manager list --available | \
        grep 'Subscription Name\|Pool ID' | \
        grep -A1 'Employee SKU' | \
        grep 'Pool ID' | awk '{print $NF}')
fi


subscription-manager attach --pool=$EMP_POOL_ID
subscription-manager repos --enable=rhel-7-server-3scale-amp-2.2-rpms
    
yum install 3scale-amp-template
````
saved to /opt/amp/templates

OR just get template from

`````
wget https://raw.githubusercontent.com/3scale/3scale-amp-openshift-templates/2.2.0.GA/amp/amp.yml
`````

Install AMP
=================
````
oc new-project 3scale-amp

oc new-app --file amp.yml --param WILDCARD_DOMAIN=apps.$EXTERNAL_IP.nip.io --param ADMIN_PASSWORD=admin

#with wildcard router
oc new-app --file amp.yml --param WILDCARD_DOMAIN=apps.$EXTERNAL_IP.nip.io --param WILDCARD_POLICY=Subdomain
````
Optional: Standalone api-cast on OpenShift
============================
https://access.redhat.com/documentation/en-us/red_hat_3scale/2.2/html-single/deployment_options/#apicast-openshift

  
```
oc new-project "3scalegateway" --display-name="gateway" --description="3scale gateway"

#create a secret
https://access.redhat.com/documentation/en-us/red_hat_3scale/2.2/html-single/accounts#tokens#creating-access-tokens
Get the Access Token from Settings Widget->Personal Details->Tokens. You can add a new one with the appropriate permissions

oc secret new-basicauth apicast-configuration-url-secret --password=https://$ACCESSTOKEN@3scale-admin.apps.$EXERNAL_IP.nip.io


oc new-app -f https://raw.githubusercontent.com/3scale/3scale-amp-openshift-templates/2.2.0.GA/apicast-gateway/apicast.yml  -p LOG_LEVEL=debug

#if you want to deploy a staging endpoint
oc new-app -f https://raw.githubusercontent.com/3scale/3scale-amp-openshift-templates/2.2.0.GA/apicast-gateway/apicast.yml  -p APICAST_NAME=apicast-staging  -p LOG_LEVEL=debug -p DEPLOYMENT_ENVIRONMENT=staging
```
create routes as needed for any services. If wildcard router enabled on the AMP having a different one would not be necessary
