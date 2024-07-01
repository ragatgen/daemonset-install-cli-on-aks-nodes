Issue, after cluster node image upgrade, or cluster upgrade, customer using blobCSI driver will face the above error, WHY???

ObjectID (managed identity auth) depends on azure cli, and since az cli is not installed in the aks nodes, the auth fails, resulting the above error, this is a very uncommon situation where there is no reference about and if you face a case like this, may be ending up on an ICM for sure, well, here is the tip on how to fix this issue:

Tip

Option 1

Change the pv configuration 

Instead of:
azureStorageAuthType=MSI
azureStorageIdentityObjectID=<guid>
 
Use 
 
azureStorageAuthType=MSI
azureStorageIdentityResourceID=<msi-res-id>


The above will make the configuration to point to the MSI resource ID instead of the object ID, this way, the pv will not try to use az cli to make the auth on this new version release of the blobCSI driver

Option 2

Install az cli in all the nodes

With the following daemon set you should be able to make the installation with no issues


apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: install-azure-cli
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: install-azure-cli
  template:
    metadata:
      labels:
        name: install-azure-cli
    spec:
      hostNetwork: true
      containers:
      - name: install-azure-cli
        image: ubuntu:22.04
        securityContext:
          privileged: true
        volumeMounts:
        - name: rootfs
          mountPath: /host
        command:
        - /bin/bash
        - -c
        - |
          set -e
          
          # Perform chroot to access host filesystem
          chroot /host /bin/bash -c '
            set -e
            apt-get update
            apt-get install -y curl

            # Install Azure CLI
            curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
          '
          
          sleep 3600
      hostPID: true
      volumes:
      - name: rootfs
        hostPath:
          path: /
![image](https://github.com/ragatgen/daemonset-install-cli-on-aks-nodes/assets/68605966/bb21c197-2f81-4996-8f05-52ae65bcfca6)
