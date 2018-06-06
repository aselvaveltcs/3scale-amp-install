# 3scale-amp-install 2.2

https://access.redhat.com/documentation/en-us/red_hat_3scale/2.2/html-single/infrastructure/#onpremises-installation

Pre-req
install openshift. Make sure you have PVs
https://access.redhat.com/documentation/en-us/red_hat_3scale/2.2/html-single/infrastructure/#configure_nodes_and_entitlements

subscription-manager register --username=$RHSM_USER --password=$RHSM_PASS
 


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

saved to /opt/amp/templates

OR just get template from

https://raw.githubusercontent.com/3scale/3scale-amp-openshift-templates/2.2.0.GA/amp/amp.yml

oc new-app --file amp.yml --param WILDCARD_DOMAIN=apps.$EXTERNAL_IP.nip.io --param ADMIN_PASSWORD=3scaleUser

