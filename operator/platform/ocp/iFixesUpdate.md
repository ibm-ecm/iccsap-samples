# Updating Content Collector for SAP ga-4.0.0.4 on Red Hat OpenShift

If you installed any of the Content Collector for SAP ga-4.0.0.4 components on a Kubernetes cluster, you can update them to a higher interim fix patch release level by using the updated operator and the relevant container iFixes. Required details like the image:tag of the interim fix patch Docker image can be found in the individual interim fix readmes.

- [Step 1: Get access to the interim fix container images](iFixesUpdate.md#step-1-get-access-to-the-interim-fix-container-images)
- [Step 2: Get access to the current version of the operator](iFixesUpdate.md#step-2-get-access-to-the-current-version-of-the-operator)
- [Step 3: Update the installed operator](iFixesUpdate.md#step-3-update-the-installed-operator)
- [Step 4: Update the custom resource YAML file for your Content Collector for SAP deployment](iFixesUpdate.md#step-4-update-the-custom-resource-yaml-file-for-your-ibm-content-collector-for-sap-deployment)
- [Step 5: Apply the updated custom resource YAML file](iFixesUpdate.md#step-5-apply-the-updated-custom-resource-yaml-file)
- [Step 6: Verify the updated automation containers](iFixesUpdate.md#step-6-verify-the-updated-automation-containers)

##  Step 1: Get access to the interim fix container images

You can access the container images in the IBM Docker registry with your IBMid (Option 1), or you can use the downloaded images from Fix Central (Option 2).

### Option 1: Create a pull secret for the IBM Cloud Entitled Registry

1. Log in to [MyIBM Container Software Library](https://myibm.ibm.com/products-services/containerlibrary) with the IBMid and password that are associated with the entitled software.

2. In the **Container software library** tile, click **View library** and then click **Copy key** to copy the entitlement key to the clipboard.

3. Log in to your Kubernetes cluster and set the context to the project for your existing deployment.

4. Create a pull secret by running a `kubectl create secret` command.
   ```bash
   $ kubectl create secret docker-registry admin.registrykey --docker-server=cp.icr.io --docker-username=iamapikey --docker-password="<API_KEY_GENERATED>" --docker-email=user@foo.com
   ```

   > **Note**: The `cp.icr.io` value for the **docker-server** parameter is the only registry domain name that contains the images.

5. Take a note of the secret and the server values so that you can set them to the **pullSecrets** and **repository** parameters when you run the operator for your containers.

### Option 2: Download the packages from the Fix Central download

For instructions on how to access and expand the interim fix patch, see the interim fix readme section “Download location” in the particular container’s readme file available in Fix Central.

1. Download the images per the instructions in your interim fix container readme. Make a note of the file name for your image.
2. Download the [`loadimages.sh`](../../scripts/loadimages.sh) script from GitHub.
3. Log in to your Kubernetes cluster and set the context to the project for your existing deployment.
4. Check that you can run a Docker command or podman command.
   For OpenShift 3.11:
   ```bash
   $ docker ps
   ```
   For OpenShift 4.2 and 4.3:
   ```bash
   $ podman ps
   ```   
5. Login to the Docker registry with a token.
   For OpenShift 3.11:
    ```bash
   $ docker login $(oc registry info) -u <ADMINISTRATOR> -p $(oc whoami -t)
   ```
   > **Note**: You can connect to a node in the cluster to resolve the `docker-registry.default.svc` parameter.

   You can also log in to an external Docker registry by using the following command:
   ```bash
   $ docker login <registry_url> -u <your_account>
   ```
   For OpenShift 4.2 and 4.3:
   ```bash
   $ podman login $(oc registry info) -u <ADMINISTRATOR> -p $(oc whoami -t) –tls-verify=false
   ```      
6. Run a `kubectl` command to make sure that you have access to Kubernetes.
   ```bash
   $ kubectl cluster-info
   ```
7. Run the `loadimages.sh` script to load the images into your Docker registry. Specify the two mandatory parameters in the command line.

   ```
   -p  Fix Central archive files location or archive filename
   -r  Target Docker registry and namespace
   -l  Optional: Target a local registry
   ```

   
   The following example shows the input values in the command line on OCP 4.2 and 4.3:

   ```
   # scripts/loadimages.sh -p <container interim fix>.tgz -r $(oc registry info)/my-project -tls-verify=false
   ```


   > **Note**: The project must have pull request privileges to the registry where the images are loaded. The project must also have pull request privileges to push the images into another namespace/project.
  
  Repeat this step for each container image from Fix Central that you want to update, including the operator image.

8. Check that the images are pushed correctly to the registry.
   ```bash
   $ oc get is
   ```

9. If you want to use an external Docker registry, create a Docker registry secret:
   ```bash
   $ oc create secret docker-registry admin.registrykey --docker-server=<registry_url> --docker-username=<your_account> --docker-password=<your_password> --docker-email=<your_email>
   ```
   Take a note of the secret and the server values so that you can set them to the **pullSecrets** and **repository** parameters when you run the operator for your containers.


## Step 2: Get access to the current version of the operator

If the operator in the project (namespace) of your deployment is already upgraded to the correct operator interim fix, skip to step 4.

1. Log in to your Kubernetes cluster and set the context to the project for your existing deployment.

2. Download or clone the repository on your local machine and change to the `operator` directory. 
   ```bash
   $ git clone git@github.com:ibm-ecm/iccsap-samples.git
   $ cd iccsap-samples/operator
   ```
   The repository contains the scripts and Kubernetes descriptors that are necessary to upgrade the Content Collector for SAP operator.


## Step 3: Update the installed operator

1. Log in to your Kubernetes cluster and set the context to the project for your existing deployment.

2. Go to the downloaded iccsap-samples.git for Content Collector for SAP Vga-4.0.0.4 and replace the files in the `/descriptors` directory with the files from the interim fix `/descriptors` folder.

   For example:
   ```bash
   $ cd iccsap-samples/operator/descriptors
   $ cp ./ga-4.0.0.4_iFix1/* . 
   ```   
3. Upgrade the iccsap-operator on your project.

   Use the interim fix [scripts/ga-4.0.0.4_iFix1/upgradeOperator.sh](../../scripts/upgradeOperator.sh) script to deploy the operator manifest descriptors.
   ```bash
   $ cd iccsap-samples/operator
   $ ./scripts/upgradeOperator.sh -i <registry_url>/iccsap-operator:ga-4.0.0.4-if001 -p 'admin.registrykey' -a accept
   ```

   Where *registry_url* is the value for your internal docker registry or `cp.icr.io/cp/cp4a` for the IBM Cloud Entitled Registry,  admin.registrykey is the secret created to access the registry, and *accept* means that you accept the [license](../../LICENSE).

   > **Note**: If you plan to use a non-admin user to install the operator, you must add the user to the `ibm-iccsap-operator` role. For example:
   ```bash
   $ oc adm policy add-role-to-user ibm-iccsap-operator <user_name>
   ```   
4. Monitor the pod until it shows a STATUS of Running:
   ```bash
   $ oc get pods -w
   ```   
     > **Note**: When started, you can monitor the operator logs with the following command:
   ```bash
   $ oc logs -f deployment/ibm-iccsap-operator -c operator
   ```    

## Step 4: Update the custom resource YAML file for your Content Collector for SAP deployment

Get the custom resource YAML file that you previously deployed and edit it by following the instructions for each component:

1. Verify that the release version is ga-4.0.0.4.1.

2. In the sections for each of the components that are included in your deployment, modify the configuration parameter ecm_configuration.<component>.image.tag to reflect the value for the image loaded, for example:
    ```bash
       repository: cp.icr.io/cp/cp4a/iccsap/cpe
       tag: ga-554-p8cpe-if001
   ```     
    > **Tip**: The values of the tags for a given interim fix can be found in the readme provided with that interim fix.
   
    > **Tip**: Verify that the secret named in the CR YAML file as the imagePullSecrets is valid. Note that the secret might be expired, in which case you must re-create the secret.

   Repeat this step for each component that you want to update to a new version.
   
3. In the ldap_configuration section, verify that the `lc_bind_secret parameter` is set.
   The value of the parameter is typically the same secret that is used in the `ecm_configuration.iccsap_secret_name parameter`. 
   
   Example:
   ```
     ldap_configuration:
    lc_bind_secret: ibm-iccsap-secret # secret is expected to have ldapUsername and ldapPassword keys
   ```    
    > **Tip**: The value that you provide for the lc_bind_secret must be the same value that you used previously for the iccsap-secret value.

## Step 5: Apply the updated custom resource YAML file

1. Check that all the components that you want to upgrade are configured with updated image tag values and so on.

   ```bash
   $ cat descriptors/my_iccsap_v1_iccsap_cr.yaml
   ```

2. Update the configured components by applying the custom resource.

   ```bash
   $ kubectl apply -f descriptors/my_iccsap_v1_iccsap_cr.yaml
   ```

## Step 6: Verify the updated automation containers

The operator reconciliation loop might take several minutes. When all of the pods are Running, you can access the status of your containers by using the following commands:
```bash
$ oc status
$ oc get pods -w
$ oc logs <operatorPodName> -f -c operator
```

Return to the interim fix readme for additional verification instructions for the particular component being updated.
