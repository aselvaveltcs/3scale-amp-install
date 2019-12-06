# 3scale 2.7 install  

## on OCP3.11
```
oc new-project 3scale27
#need to create secret

oc create secret docker-registry threescale-registry-auth \
  --docker-server=registry.redhat.io \
  --docker-username=<serviceaccount> \
  --docker-password=<serviceaccountpass>

oc new-app \
--file https://raw.githubusercontent.com/3scale/3scale-amp-openshift-templates/2.7-stable/amp/amp.yml \
--param WILDCARD_DOMAIN=apps.cnr-test.redhatgov.io --param ADMIN_PASSWORD=admin \
--param TENANT_NAME=3scale27 --param MASTER_NAME=master27
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


TODO - fix for 2.7 Optional: Standalone 2.3 api-cast on OpenShift
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
