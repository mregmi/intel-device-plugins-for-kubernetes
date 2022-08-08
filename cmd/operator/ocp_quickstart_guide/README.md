# Intel® Device Plugins Operator for Red Hat OpenShift Container Platform

## Introduction
The Intel Device Plugins Operator for OpenShift Container Platform is a collection of device plugins advertising Intel specific hardware resources to the kubelet. It provides a single point of control for Intel® Software Guard Extensions (Intel® SGX), Intel GPUs, Intel® QuickAccess Technology (Intel® QAT), Intel® Data Streaming Accelerator (Intel® DSA), and Intel® In-Memory Analytics Accelerator (Intel® IAA) devices to cluster administrators. 

**This release supports Intel Device Plugin Operator with QAT device plugin functionality and is intended for testing purposes only.** 

## Minimum Hardware Requirements
### Intel QAT Enabled Server
- Fourth Generation Intel® Xeon® Scalable Platform, code named "Sapphire Rapids" or later

# Installation - Intel Device Plugins Operator
## Prerequisites 
- Make sure Red Hat OpenShift Cluster is ready to use and the developing machine is RHEL and `oc` command is installed and configured properly. Please note that the following operation is verified on Red Hat OpenShift Cluster 4.11 and working machine RHEL-8.6
- Install the `oc` command to your development machine
- Development machine configurations tested on -  

    | Tool        | Version  |
    | ------------- |-------------|
    |go | go1.19 |
    |kubectl | v1.24.3 |
    |operator-sdk | v1.22.2 |
- Follow the [link](https://docs.openshift.com/container-platform/4.10/hardware_enablement/psap-node-feature-discovery-operator.html) to install **NFD operator** (if it's not already installed).   
    **Note:** Please only install the NFD operator and use steps below to create the NodeFeatureDiscovery instance.  
    - Create the NodeFeatureDiscovery instance  
    ```
    $ oc apply -f https://raw.githubusercontent.com/intel/intel-device-plugins-for-kubernetes/v0.24.0/deployments/nfd/overlays/node-feature-discovery/node-feature-discovery-openshift.yaml
    ```
    - Create the NodeFeatureRule instance  
    **Note:** By default, QAT device pci id supported are `["37c8", "4940"]`. If the pci id of the underlying QAT card is different, please add the entry to QAT device value in the file `/deployments/nfd/overlays/node-feature-rules/node-feature-rules-openshift.yaml`.
    ``` 
    $ oc apply -f https://raw.githubusercontent.com/intel/intel-device-plugins-for-kubernetes/v0.24.0/deployments/nfd/overlays/node-feature-rules/node-feature-rules-openshift.yaml
    ```

## Intel Device Plugins Operator Installation
Install the operator using below command - 
```
$ operator-sdk run bundle quay.io/ocpeng/intel-device-plugins-operator-bundle:0.24.1
```

## Verify Operator installation
1.  On the OpenShift console, go to **Operator** -> **Installed Operators**
2.  Verify the status of Intel Device Plugins Operator as **Succeeded**


# Deployment - Intel QAT Device Plugin 
## Prerequisites
In order to enable Intel QAT, there is a need for changing kernel parameters in the cluster node to enable `iommu`. This is done using [machine config](https://docs.openshift.com/container-platform/4.10/post_installation_configuration/machine-configuration-tasks.html). Please deploy the machine config using command -
```
$ oc apply -f https://raw.githubusercontent.com/mregmi/intel-device-plugins-for-kubernetes/qat_enable/cmd/operator/ocp_quickstart_guide/mco-iommu.yaml
```
**NOTE:** When the kernel parameter of the worker node is changed, the worker node will reboot. It is a one-time process. Please make sure that the worker node is up and running before deploying the QAT CR.


## Deploying Intel QAT Device Plugin Custom Resource
1.	Go to **Operator** -> **Installed Operators**
2.  Open **Intel Device Plugins Operator**
3.  Navigate to the tab **Intel Quick Assist Technology Device Plugin**
4.  Click **Create QatDevicePlugin ->** set the correct parameters -> Click **Create** 
    OR for any customizations, please select `YAML view` and edit details. Once done, please click **Create**.
5.  Verify the CR by checking the status of the DaemonSet **`intel-qat-plugin`**
6.  Now `QatDevicePlugin` is ready to deploy the workloads
