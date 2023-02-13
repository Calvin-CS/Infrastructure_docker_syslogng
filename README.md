# Infrastructure_docker_syslogng

Note: I could not get syslogng working, but my custom rsyslog works wonderfully.  Only use that one.

Couple of extra setup steps here:
1. Make the PVs for the config and /var/log/ persistent storage before in the production namespace.
  (kubernetes/syslogngpv-dynamic.yaml)
2. Deploy the kubernetes manifests.
3. Make a new private end point using Azure CLI (Bash):

AKS_MC_RG=$(az aks show -g CS_Infrastructure_AKS --name Infrastructure --query nodeResourceGroup -o tsv)

To confirm: az network private-link-service list -g $AKS_MC_RG --query "[].{Name:name,Alias:alias}" -o table

AKS_PLS_ID=$(az network private-link-service list -g $AKS_MC_RG --query "[].id" -o tsv)

az network private-endpoint create -g CS_Infrastructure_USNorthCentral --name aksSyslogPE --vnet-name Subnet_CS_Cloud_USNorthCentral --subnet Server_AKS_syslog --private-connection-resource-id $AKS_PLS_ID --connection-name connectToAKSSyslog


NOTE: if you remove the service from kubernetes (not upgrading, removal) then you need to make sure remove a few things:
- aksSyslogPE private-endpoint
- associated vNic in the Server_AKS_syslog vnet

4. Register DNS to the pviate-endpoint
  syslog.cs.calvin.edu -> 10.230.64.28
