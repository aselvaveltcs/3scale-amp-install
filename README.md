# 3scale 2.7 install  TODO create a new branch 

## on OCP3.11
```
oc new-project 3scale27
#need to create secret

oc create secret docker-registry threescale-registry-auth \
  --docker-server=registry.redhat.io \
  --docker-username=<serviceaccount> \
  --docker-password=<serviceaccountpass>

oc new-app --file https://raw.githubusercontent.com/3scale/3scale-amp-openshift-templates/2.7-stable/amp/amp.yml \
--param WILDCARD_DOMAIN=apps.cnr-test.redhatgov.io --param ADMIN_PASSWORD=admin --param TENANT_NAME=3scale27 \
--param MASTER_NAME=master27
```
## On 4.2
```
oc new-project 3scale27
#need to create secret

oc create secret docker-registry threescale-registry-auth \
  --docker-server=registry.redhat.io \
  --docker-username=<serviceaccount> \
  --docker-password=<serviceaccountpass>
```
Then follow instructions here 
https://github.com/3scale/3scale-operator/blob/master/doc/quickstart-guide.md

# 3scale-amp-install 2.3

https://access.redhat.com/documentation/en-us/red_hat_3scale/2.3/html-single/infrastructure/onpremises-installation#onpremises-installation

Pre-req
====================
install openshift. Make sure you have PVs
https://access.redhat.com/documentation/en-us/red_hat_3scale/2.3/html-single/infrastructure/onpremises-installation#configure_nodes_and_entitlements

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
subscription-manager repos --enable=rhel-7-server-3scale-amp-2.3-rpms
    
yum install 3scale-amp-template
````
saved to /opt/amp/templates
````
export AMP_TEMPLATE=/opt/amp/templates/amp.yml
````

OR just get template from

`````
wget https://raw.githubusercontent.com/3scale/3scale-amp-openshift-templates/2.3.0.GA/amp/amp.yml
export AMP_TEMPLATE=amp.yml
`````

OR via RPM page

AMP -https://access.redhat.com/downloads/content/3scale-amp-template/2.3.0-2.el7_5/x86_64/fd431d51/package

Template - https://access.redhat.com/downloads/content/3scale-amp-apicast-gateway-template/2.3.0-2.el7_5/x86_64/fd431d51/package

Install AMP
=================
````
oc new-project 3scale-amp

oc new-app --file $AMP_TEMPLATE --param WILDCARD_DOMAIN=apps.$EXTERNAL_IP.nip.io --param ADMIN_PASSWORD=admin --param TENANT_NAME=3scale-amp

#with wildcard router
oc new-app --file $AMP_TEMPLATE --param WILDCARD_DOMAIN=apps.$EXTERNAL_IP.nip.io --param WILDCARD_POLICY=Subdomain --param ADMIN_PASSWORD=admin --param TENANT_NAME=3scale-amp
````

If using wildcard router - 
https://docs.openshift.com/container-platform/3.11/install_config/router/default_haproxy_router.html#using-wildcard-routes

Alternatively: You can use select the template from the Service Catalog wizard on the OpenShift console

Take note of the passwords and tokens in the output. When the admin console is up and running, you will need the ADMIN_ACCESS_TOKEN .


Optional: Standalone api-cast on OpenShift
============================
https://access.redhat.com/documentation/en-us/red_hat_3scale/2.3/html-single/deployment_options/#apicast-openshift

  
```
oc new-project "3scalegateway" --display-name="gateway" --description="3scale gateway"
````

create a secret
https://access.redhat.com/documentation/en-us/red_hat_3scale/2.3/html-single/accounts/index#access_tokens

Get the Access Token from Settings Widget->Personal Details->Tokens. You can add a new one with the appropriate permissions

test
````
curl -k -v https://${ACCESS_TOKEN}@${TENANT_NAME}-admin.apps.${EXTERNAL_IP}.nip.io/admin/api/services.json | python -m json.tool

oc secret new-basicauth apicast-configuration-url-secret --password=https://${ACCESS_TOKEN}@${TENANT_NAME}-admin.apps.${EXTERNAL_IP}.nip.io


oc new-app -f https://raw.githubusercontent.com/3scale/3scale-amp-openshift-templates/2.3.0.GA/apicast-gateway/apicast.yml  -p LOG_LEVEL=debug
````

if you want to deploy a staging endpoint

````
oc new-app -f https://raw.githubusercontent.com/3scale/3scale-amp-openshift-templates/2.3.0.GA/apicast-gateway/apicast.yml  -p APICAST_NAME=apicast-staging  -p LOG_LEVEL=debug -p DEPLOYMENT_ENVIRONMENT=staging

````

create routes as needed for any services. If wildcard router enabled on the AMP having a different one would not be necessary
