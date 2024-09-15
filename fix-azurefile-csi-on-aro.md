# Goal

When deploying [ARO cluster](https://learn.microsoft.com/en-us/azure/openshift/intro-openshift) 
by default the Azure file csi storage class does not work. We show how to fix that.

# Set the default driver to `unManaged`

As explained in [the openshift guide](https://docs.openshift.com/container-platform/4.14/storage/container_storage_interface/persistent-storage-csi-sc-manage.html#persistent-storage-csi-sc-manage) you need to have the default storage class unmanaged.

To accomplish these objectives, you change the setting for the spec.storageClassState field in the ClusterCSIDriver.

```
oc edit ClusterCSIDriver file.csi.azure.com
```

to the value "Unmanaged".

# Follow the microsoft guide first 

Follow the [microsoft guide](/https://learn.microsoft.com/en-us/azure/openshift/howto-create-a-storageclass) first 
but instead of creating a new storage class replace the default storage class by the storage class you created 
following the tutorial.