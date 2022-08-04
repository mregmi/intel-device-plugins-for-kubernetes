# Intel® Device Plugins Operator for Red Hat OpenShift Container Platform

## Introduction
The Intel Device Plugins Operator for OpenShift Container Platform is a collection of device plugins advertising Intel specific hardware resources to the kubelet. It provides a single point of control for Intel® Software Guard Extensions (Intel® SGX), Intel GPUs, Intel® QuickAccess Technology (Intel® QAT), Intel® Data Streaming Accelerator (Intel® DSA), and Intel® In-Memory Analytics Accelerator (Intel® IAA) devices to cluster administrators. 

**This release supports Intel Device Plugin Operator with QAT device plugin functionality and is intended for Intel internal teams only for testing purposes.** 

## Minimum Hardware Requirements
### Intel QAT Enabled Server
- Fourth Generation Intel® Xeon® Scalable Platform, code named "Sapphire Rapids" or later

# Installation - Intel Device Plugins Operator
## Prerequisites 
- Make sure Red Hat OpenShift Cluster is ready to use and the developing machine is RHEL and `oc` command is installed and configured properly. Please note that the following operation is verified on Red Hat OpenShift Cluster 4.10 and working machine RHEL-8.5
- Install the `oc` command to your development system
- Make sure to install operator-sdk
- Install **NFD operator** (if it's not already installed). Follow the [link](https://docs.openshift.com/container-platform/4.10/hardware_enablement/psap-node-feature-discovery-operator.html)  
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
Since the operator is not released externally, you can install the operator using operator-sdk. Run 
```
$ operator-sdk run bundle quay.io/ocpeng/intel-device-plugins-operator-bundle:0.24.1
```

## Verify Operator installation
1.  On the OpenShift console, go to **Operator** -> **Installed Operators**
2.  Verify the status of Intel Device Plugins Operator as **Succeeded**


# Deployment - Intel QAT Device Plugin 
## Prerequisites
In order to enable Intel QAT, there is a need for changing kernel parameters in the cluster node. This is done using [machine config](https://docs.openshift.com/container-platform/4.10/post_installation_configuration/machine-configuration-tasks.html). Please deploy the following machine config:
```
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  labels:
    machineconfiguration.openshift.io/role: worker 
  name: 100-worker-gpu 
spec:
  config:
    ignition:
      version: 3.2.0
  kernelArguments:
      - intel_iommu=on
```


## Deploying Intel QAT Device Plugin Custom Resource
1.	Go to **Operator** -> **Installed Operators**
2.  Open **Intel Device Plugins Operator**
3.  Navigate to the tab **Intel Quick Assist Technology Device Plugin**
4.  Click **Create QatDevicePlugin ->** set the correct parameters -> Click **Create** 
    OR for any customizations, please select `YAML view` and edit details. Once done, please click **Create**.
5.  Verify the CR by checking the status of the DaemonSet **`intel-qat-plugin`**
6.  Now `QatDevicePlugin` is ready to deploy the workloads
