# Helm Chart for Openshift ZTP on Openshift Virtualization
This project is for a Helm Chart that deploys the necessary resources for running VMs in Openshift Virtualization for Openshift ZTP.  

This Chart is based on @v1k0d3n 's Gist [here](https://gist.github.com/v1k0d3n/c5da49ec0cca4d8517af8ffee9248308)

The Chart will create the VMs in your OCP cluster, and also deployments, services, and routes to expose those VMs via fakefish.  This allows for control over those VMs via Redfish, including powering on and off, mounting ISO files, and many other tasks.  

Once deployed, you can configure ACM and ZTP as normal and create `BareMetalHost` (BMH) objects like they were a physical server with out-of-band management.  

NOTE: This type of deployment is only suitable for lab/non-production/demonstration use.  

# Usage
## Install
1.  Clone the project and navigate to the chart directory

``` 
git clone git@github.com:ttracey-rh/ocp-cnv-ztp-helm.git
cd cnv-ztp
```

2.  Copy the `values.yaml` file and modify the variables in the new file to match your OCP cluster.  See below for an explanation of the variables.
```
cp values.yaml my-values.yaml
```
3.  Copy your cluster's kubeconfig into the `secrets/` directory.  This is needed for the fakefish pods to interact with the VMs within OCP.  The admin kubeconfig will work, of course, but another user with the necessary RBAC in place will work as well.

4.  Install the chart

``` 
helm install ztp-cnv . -f my-values.yaml
```

5.  Retrieve the routes and MAC addresses for the fakefish pods and newly created VMs for creating your BMH resources
```
oc get vm -ocustom-columns=NAME:.metadata.name,MAC:.spec.template.spec.domain.devices.interfaces[0].macAddress -n <vm namespace>
oc get route -n <vm namespace>
```

## Uninstall
1. Uninstall the helm chart
```
helm uninstall ztp-cnv
```
2.  Verify the resources have been removed
```
oc get ns <vm namespace>
```

# Variable Description
| Variable | Description | Example |
|---|---|---|
|vm_name | VM Name Prefix | ztp-node |
|vm_namespace | Namespace for the VMs and fakefish resources | ztp-vms |
|vm_number | Number of VMs to create | 3 |
| storageclass | StorageClass for VM Disks | nfs-csi|
|size_disk1 | Size of the first virtual hard disk for the OS  | 120Gi |
|size_disk2 | Optional.  Size of a second virtual hard disk for use with ODF.  | 100Gi |
| size_mem | Memory resources for VMs | 48Gi |
|size_cpu | CPU Core count for VMs | 10 |
|img_fakefish| URL and Tag for the kubevirt-fakefish image| quay.io/bjozsa-redhat/fakefish-kubevirt:v4.17.6 |
|network | Namespace and Name of a `NetworkAttachmentDefinition` to connect the VMs to the network| default/br1-vlan110 |
| metallb | Optional.  Boolean.  Creates a LoadBalancer type service to expose the fakefish pods via an IP address. | false
|metallb_pool| Required if `metallb` is true.  Name of the IP Pool for MetalLB to use. | vlan110-pool |
| odf | Optional.  Boolean.  Creates the second disk resources for ODF | false | 
|kubeconfig| Name of the kubeconfig file placed in the secrets/ directory| kubeconfig-ocp-virt-lab | 

Note: Variables for VM resources must be specified in `Mi` or `Gi`.  The OCP virtualization mutating webhook will reject any other values.

# To Do
- There's probably a better way to get the kubeconfig into the fakefish deployments.
- Add the `NetworkAttachmentDefinition` resources to the chart so they are deployed automatically.  

