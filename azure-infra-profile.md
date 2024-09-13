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

# 2 cases 

Check if there is a service principal that has the name of the resource group by executing 

```
az ad sp list --display-name $RESOURCE_GROUP
```

If the result of the output is `[]` then we are in **case 1** and we need to create it. Otherwise it already 
exist and we are in **case 2** 

## Case 1 Service principal with the name $RESOURCE_GROUP does not exist yet 

Let's create it 
```
az ad sp create-for-rbac --name $RESOURCE_GROUP --role Contributor --scopes /subscriptions/$SUBSCRIPTION/resourceGroups/$RESOURCE_GROUP
```

And you'll have a similar output than above
```
{
  "appId": "<redacted>",  
  "password": "<redacted>",
  "tenant": "<redacted>"
}
```

You will need also the subscription and the resource group name
```
echo $SUBSCRIPTION
echo $RESOURCE_GROUP
```

## Case 2 Service principal with the name $RESOURCE_GROUP already exist 

There is already a service principal that has exactly the same name than the resource group
and has already the contributor role. 

We have to add a supplementary secret for this service principal.
 
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

You will need also the subscription and the resource group name
```
echo $SUBSCRIPTION
echo $RESOURCE_GROUP
```

You must provide this 5 informations to complete the profile.

# Openshift on Azure

oc get ns default -ojsonpath='{.metadata.uid}'

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

