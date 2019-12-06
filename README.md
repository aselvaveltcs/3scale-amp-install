# 3scale 2.7 AMP install  

## on OCP3.11
```
oc new-project 3scale27
#need to create secret

oc create secret docker-registry threescale-registry-auth \
  --docker-server=registry.redhat.io \
  --docker-username=<serviceaccount> \
  --docker-password=<serviceaccountpass>

# modify wild card domain for the OCP cluster
export WILDCARD_DOMAIN=apps.example.com

oc new-app \
--file https://raw.githubusercontent.com/3scale/3scale-amp-openshift-templates/2.7-stable/amp/amp.yml \
--param WILDCARD_DOMAIN=${WILDCARD_DOMAIN}--param ADMIN_PASSWORD=admin \
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


Standalone 2.7 api-cast on OpenShift
============================
https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.7/html/installing_3scale/installing-apicast

  
```
oc new-project "3scalegateway" --display-name="gateway" --description="3scale gateway"
````

create a secret using the instructions here
https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.7/html/admin_portal_guide/tokens#access_tokens

Click on the gear icon in the navigation bar.
Navigate to Personal > Tokens.
Click Add Access Token.
Specify a name, select one or more scopes, and choose the permission for the token.
To save the new token, click Create Access token.

test
````

# modify wild card domain for the OCP cluster
export WILDCARD_DOMAIN=apps.example.com

curl -k -v https://${ACCESS_TOKEN}@${TENANT_NAME}-admin.${WILDCARD_DOMAIN}/admin/api/services.json | python -m json.tool

oc secret new-basicauth apicast-configuration-url-secret --password=https://<APICAST_ACCESS_TOKEN>@<TENANT_NAME>-admin.<WILDCARD_DOMAIN>


oc new-app -f https://raw.githubusercontent.com/3scale/3scale-amp-openshift-templates/2.7.0.GA/apicast-gateway/apicast.yml  -p LOG_LEVEL=debug
````

if you want to deploy a staging endpoint

````
oc new-app -f https://raw.githubusercontent.com/3scale/3scale-amp-openshift-templates/2.7.0.GA/apicast-gateway/apicast.yml  -p APICAST_NAME=apicast-staging  -p LOG_LEVEL=debug -p DEPLOYMENT_ENVIRONMENT=staging

````

create routes as needed for any services. 
