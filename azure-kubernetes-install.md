Azure CLI
---------

We need the Azure CLI to wire up the connection between Azure Kubernetes Service and the `kubectl` Kubernetes command line, and to authorize communication between Azure Kubernetes Service and Azure Container Registry.

1. See https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest for the steps to download and install the Azure CLI on your computer.

2. With the Azure CLI installed, run:

   ```
   az login
   ```

   It'll direct you to open a URL, paste in a code, and login to your Microsoft account.

3. If necessary, switch subscriptions with `az account list --output table` and [`az account set ...`](https://docs.microsoft.com/en-us/cli/azure/account?view=azure-cli-latest#az-account-set)

4. Run this command to login to kubectl:

   ```
   az aks get-credentials --resource-group kubernetes --name YOUR_CLUSTER_NAME
   ```

   Because I named my cluster `CLUSTER_NAME-` at the top of this exercise, I'll run `az aks get-credentials --resource-group kubernetes --name CLUSTER_NAME`

5. The certificate details end up in `~/.kube/config`.  Open this file in a text editor and look around.

   Notice the `docker-for-desktop` configuration together with the Azure Kubernetes cluster.

Now let's get the Kubernetes cluster logged into the registry.  (See also https://docs.microsoft.com/en-us/azure/container-registry/container-registry-auth-aks)

6. Run this command:

   ```
   az aks show --resource-group kubernetes --name YOUR_CLUSTER_NAME --query "servicePrincipalProfile.clientId" --output tsv
   ```

   This shows the service principal Azure created for the Kubernetes cluster.

7. Run this command to get the ACR id:

   ```
   az acr show --resource-group kubernetes --name YOUR_REGISTRY_NAME --query "id" --output tsv
   ```

   This shows the resource id for the Azure Container Registry.  It's very long.

8. Now let's use the two ids we harvested in the previous commands to assign permissions from Kubernetes to reach into the container registry.

   ```
   az role assignment create --assignee AKS_PRINCIPAL --role Reader --scope ACR_ID
   ```

   The `AKS_PRINCIPAL` is the result from step 6, and the `ACR_ID` is the result from step 7.

   Switch clusters
---------------

1. The Azure cli switched us from `docker-for-desktop` to the Azure cluster.  In a command prompt, run:

   ```
   kubectl get all
   ```

   Notice the `frontend` and `backend` resources we created in previous sections aren't present in this server.

2. List all your contexts:

   ```
   kubectl config get-contexts
   ```

3. To switch to Docker's Kubernetes cluster, run:

   ```
   kubectl config use-context docker-for-desktop
   ```

   Now run `kubectl get all` and see the content we created previously.

4. Switch back to the Azure Kubernetes cluster:

   ```
   kubectl config use-context YOUR_NAME
   ```

   I typed `kubectl config use-context <name>`.