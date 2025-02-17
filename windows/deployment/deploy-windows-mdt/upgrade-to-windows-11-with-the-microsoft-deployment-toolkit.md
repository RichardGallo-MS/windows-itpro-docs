---
title: Perform an in-place upgrade to Windows 11 with MDT (Windows 11)
description: The simplest path to upgrade PCs that are currently running an earlier version of Windows client to Windows 11 is through an in-place upgrade.
ms.assetid: B8993151-3C1E-4F22-93F4-2C5F2771A460
ms.reviewer: 
manager: dougeby
ms.author: greglin
keywords: upgrade, update, task sequence, deploy
ms.prod: w10
ms.mktglfcycl: deploy
ms.localizationpriority: medium
ms.sitesec: library
ms.pagetype: mdt
audience: itpro
author: greg-lindsay
ms.topic: article
---

# Perform an in-place upgrade to Windows 11 with MDT

**Applies to**
- Windows 10
- Windows 11

The simplest path to upgrade PCs that are currently running an earlier version of Windows client to Windows 11 is through an in-place upgrade. 

> [!TIP]
> In-place upgrade is the preferred method to use when migrating to a newer version of the same OS, or upgrading to a new OS. This is especially true when you do not plan to significantly change the device's configuration or applications. MDT includes an in-place upgrade task sequence template that makes the process really simple. 

In-place upgrade differs from [computer refresh](refresh-a-windows-10-computer-with-windows-11.md) in that you cannot use a custom image to perform the in-place upgrade. In this article we will add a default Windows 11 image to the production deployment share specifically to perform an in-place upgrade.

> [!IMPORTANT]
> Windows 11 setup will block the upgrade process on devices that do not meet [Windows 11 hardware requirements](/windows/whats-new/windows-11-requirements). Be sure to verify that your device meets these requirements before attempting to upgrade to Windows 11.

Three computers are used in this topic: DC01, MDT01, and PC0002. 

- DC01 is a domain controller for the contoso.com domain
- MDT01 is a domain member server 
- PC0002 is a domain member computer running Windows 10, targeted for the Windows 11 upgrade

 ![computers.](../images/mdt-upgrade.png)

 The computers used in this topic.

> [!NOTE]
> For details about the setup for the procedures in this article, please see [Prepare for deployment with MDT](prepare-for-windows-deployment-with-mdt.md).

> If you have already completed all the steps in [Deploy a Windows 11 image using MDT](deploy-a-windows-11-image-using-mdt.md), then you already have a production deployment share and you can skip to [Add Windows 11 Enterprise x64 (full source)](#add-windows-11-enterprise-x64-full-source).

## Create the MDT production deployment share

On **MDT01**:

1. Ensure you are signed on as: contoso\administrator.
2. In the Deployment Workbench console, right-click **Deployment Shares** and select **New Deployment Share**.
3. On the **Path** page, in the **Deployment share path** text box, type **D:\\MDTProduction** and click **Next**.
4. On the **Share** page, in the **Share name** text box, type **MDTProduction$** and click **Next**.
5. On the **Descriptive Name** page, in the **Deployment share description** text box, type **MDT Production** and click **Next**.
6. On the **Options** page, accept the default settings and click **Next** twice, and then click **Finish**.
7. Using File Explorer, verify that you can access the **\\\\MDT01\\MDTProduction$** share.

## Add Windows 11 Enterprise x64 (full source)

> If you have already have a Windows 11 [reference image](create-a-windows-11-reference-image.md) in the **MDT Build Lab** deployment share, you can use the deployment workbench to copy and paste this image from the MDT Build Lab share to the MDT Production share and skip the steps in this section.

 ![copy reference image.](../images/mdt-copy-image.png)

 Copying the reference image to the production deployment share

 If you copy the reference image using the above process, you should verify that all the files on MDT01 in **D:\\MDTBuildLab\\Operating Systems\\W11EX64** were successfully copied to **D:\\MDTProduction\\Operating Systems\\W11EX64** and then skip to [Create a task sequence to upgrade to Windows 11 Enterprise](#create-a-task-sequence-to-upgrade-to-windows11-enterprise).

On **MDT01**:

1. Sign in as contoso\\administrator and copy the content of a Windows 11 Enterprise x64 DVD/ISO to the **D:\\Downloads\\Windows 11 Enterprise x64** folder on MDT01, or just insert the DVD or mount an ISO on MDT01.
2. Using the Deployment Workbench, expand the **Deployment Shares** node, and then expand **MDT Production**.
3. Right-click the **Operating Systems** node, and create a new folder named **Windows 11**.
4. Expand the **Operating Systems** node, right-click the **Windows 11** folder, and select **Import Operating System**. Use the following settings for the Import Operating System Wizard:
    - Full set of source files
    - Source directory: (location of your source files)
    - Destination directory name: <b>W11EX64</b>
5. After adding the operating system, in the **Operating Systems / Windows 11** folder, double-click it and change the name to: **Windows 11 Enterprise x64 Default Image**.

## Create a task sequence to upgrade to Windows 11 Enterprise

On **MDT01**:

1.  Using the Deployment Workbench, select **Task Sequences** in the **MDT Production** node, and create a folder named **Windows 11**.
2.  Right-click the new **Windows 11** folder and select **New Task Sequence**. Use the following settings for the New Task Sequence Wizard:
    -   Task sequence ID: W11-X64-UPG
    -   Task sequence name: Windows 11 Enterprise x64 Upgrade
    -   Template: Standard Client Upgrade Task Sequence
    -   Select OS: Windows 11 Enterprise x64 Default Image
    -   Specify Product Key: Do not specify a product key at this time
    -   Organization: Contoso
    -   Admin Password: Do not specify an Administrator password at this time

### Specify additional command line options

Before running the upgrade task sequence, an additional step is required if you are upgrading to Windows 11.  This step is not necessary if you are upgrading to Windows 10.

The **/EULA accept** command line option is required starting with Windows 11. For more information, see [Windows Setup command-line options](/windows-hardware/manufacture/desktop/windows-setup-command-line-options#eula). To add this command line option:

1. In the Windows 11 Enterprise x64 Upgrade task sequence that you just created, in the Preparation section, click **Add** > **General** > **Set Task Sequence Variable** and provide the following values:
   - Name: WindowsUpgradeAdditionalOptions
   - Task Sequence Variable: WindowsUpgradeAdditionalOptions
   - Value: /EULA accept
2. Make the Set Task Sequence Variable step the first step in the Preparation phase by moving it up above the other steps.  See the following example:

![Specify EULA](../images/windowsupgradeadditionaloptions.png)

Using the WindowsUpgradeAdditionalOptions variable to set command line options.

## Perform the Windows 11 upgrade

To initiate the in-place upgrade, perform the following steps on PC0002 (the device to be upgraded).

On **PC0002**:

1. Start the MDT deployment wizard by running the following command: **\\\\MDT01\\MDTProduction$\\Scripts\\LiteTouch.vbs**
2. Select the **Windows 11 Enterprise x64 Upgrade** task sequence, and then click **Next**. 
3. Select one or more applications to install (will appear if you use custom image): Install - Adobe Reader
4. On the **Ready** tab, click **Begin** to start the task sequence.
   When the task sequence begins, it automatically initiates the in-place upgrade process by invoking the Windows setup program (Setup.exe) with the necessary command-line parameters to perform an automated upgrade, which preserves all data, settings, apps, and drivers.

![upgrade1.](../images/upgrademdt-fig5-winupgrade.png)

<br>

After the task sequence completes, the computer will be fully upgraded to Windows 11.

## Related topics

[Windows 10 deployment scenarios](../windows-10-deployment-scenarios.md)<br>
[Microsoft Deployment Toolkit downloads and resources](/mem/configmgr/mdt/)