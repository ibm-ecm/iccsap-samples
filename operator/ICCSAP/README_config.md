# Configuring IBM Content Collector for SAP 4.0.0.4

IBM Content Collector for SAP provides numerous containerized components for use in your container environment. The configuration settings for the components are recorded and stored in the shared YAML file for operator deployment. After you prepare your environment, you add the values for your configuration settings to the YAML so that the operator can deploy your containers to match your environment. 

## Requirements and prerequisites

Confirm that you have completed the following tasks to prepare to deploy your Content Collector for SAP images:

- Prepare your Content Collector for SAP environment.You must complete all of the [preparation steps for Content Collector for SAP](https://www.ibm.com/support/knowledgecenter/SSRW2R_4.0.0/doc/container/tsk_preparing.html) before you are ready to deploy the container images. Collect the values for these environment components; you use them to configure your Content Collector for SAP container deployment.

- Prepare your container environment. See [Preparing to install automation containers on Kubernetes](https://www.ibm.com/support/knowledgecenter/SSRW2R_4.0.0/doc/container/tsk_preparing.html)

## Customize the YAML file for your deployment

All of the configuration values for the components that you want to deploy are included in the [iccsap_v1_iccsap_cr_template.yaml](../descriptors/iccsap_v1_iccsap_cr_template.yaml) file. Create a copy of this file on the system that you prepared for your container environment, for example `my_iccsap_v1_iccsap_cr_template.yaml`. 

The custom YAML file includes the following sections that apply for all of the components:
- shared_configuration - Specify your deployment and your overall security information.
- monitoring_configuration - Optional for deployments where you want to enable monitoring.
- logging_configuration - Optional for deployments where you want to enable logging.

After the shared section, the YAML includes a section of parameters for each of the available components. If you plan to include a component in your deployment, you un-comment the parameters for that component and update the values. For some parameters, the default values are sufficient. For other parameters, you must supply values that correspond to your specific environment or deployment needs. 

The optional initialize_configuration and verify_configuration section includes values for a set of automatic set up steps for your FileNet P8 domain and IBM Content Navigator deployment. 

If you want to exclude any components from your deployment, leave the section for that component and all related parameters commented out in the YAML file. 

All components require that you deploy the Content Platform Engine container. For that reason, you must complete the values for that section in all deployment use cases.

A description of the configuration parameters is available in [Configuration reference for operators](https://www.ibm.com/support/knowledgecenter/SSRW2R_4.0.0/doc/container/tsk_install_icp4a_operator.html)

For a more focused YAML file that contains the default value for each Content Collector for SAP parameter, see the [iccsap_sample_cr.yaml](../descriptors/iccsap_sample_cr.yaml). You can use this shorter sample resource file to compile all the values you need for your Content Collector for SAP environment, then copy the sections into the [iccsap_v1_iccsap_cr_template.yaml](../descriptors/iccsap_v1_iccsap_cr_template.yaml) file before you deploy.

Use the information in the following sections to record the configuration settings for the components that you want to deploy.


## Complete the installation

After you have set all of the parameters for the relevant components, return to to the install or update page for your platform to configure other components and complete the deployment with the operator.

Install pages:
   - [Installing on Red Hat OpenShift](../platform/ocp/install.md)
   - [Installing on Certified Kubernetes](../platform/k8s/install.md)

Update pages:
   - [Updating on Red Hat OpenShift](../platform/ocp/update.md)
   - [Updating on Certified Kubernetes](../platform/k8s/update.md)
