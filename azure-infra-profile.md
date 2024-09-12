# Goal 

Create a service principal that has the contributor role and can take snapshot of legacy azure storage 

# Discover the resource group your PVC belong to 

Get a pv from a pvc and display the resource group 
```
PV=$(oc get pvc -n kasten-io catalog-pv-claim -o jsonpath='{.spec.volumeName}')
VHANDLE=$(oc get pv $PV -o jsonpath='{.spec.csi.volumeHandle}' )
RESOURCE_GROUP=$(echo $VHANDLE | awk -F'/resourceGroups/' '{split($2, a, "/"); print a[1]}')
SUBSCRIPTION=$(echo $VHANDLE | awk -F'/subscriptions/' '{split($2, a, "/"); print a[1]}')
echo $RESOURCE_GROUP
echo $SUBSCRIPTION
```

And check on azure portal that this is the subscription and the resource group your pvc volume belong to. 

There is also a service principal that has exactly the same name than the resource group
which has already the contributor role. 


# Add a supplementary secret for this service principal.
 
``` 
SP_APP_ID=$(az ad sp list --display-name $RESOURCE_GROUP --query "[0].appId" --output tsv)
az ad app credential reset \
  --id $SP_APP_ID \
  --append \
  --display-name kasten
  --years 3
```

And you'll have a similar output than above
```
{
  "appId": "<redacted>",  
  "password": "<redacted>",
  "tenant": "<redacted>"
}
```

You will need also the subscription and tthe resource 
```
echo $SUBSCRIPTION
echo $RESOURCE_GROUP
```

You must provide this 5 information to complete the profile.

# Openshift on Azure

I you are using openshift you'll have to make sure that dashboard, 
executor and aggregatedapis can have access to the hostnetwork 
```
services:
  dashboardbff:
    hostNetwork: true
  executor:
    hostNetwork: true
  aggregatedapis:
    hostNetwork: true
```
