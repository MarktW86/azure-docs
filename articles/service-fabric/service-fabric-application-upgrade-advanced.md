---
title: Advanced Application Upgrade Topics | Microsoft Docs
description: This article covers some advanced topics pertaining to upgrading a Service Fabric application.
services: service-fabric
documentationcenter: .net
author: mani-ramaswamy
manager: timlt
editor: ''

ms.assetid: e29585ff-e96f-46f4-a07f-6682bbe63281
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 7/27/2017
ms.author: subramar;chackdan

---
# Service Fabric application upgrade: advanced topics
## Adding or removing services during an application upgrade
If a new service is added to an application that is already deployed, and published as an upgrade, the new service is added to the deployed application.  Such an upgrade does not affect any of the services that were already part of the application. However, an instance of the service that was added must be started for the new service to be active (using the `New-ServiceFabricService` cmdlet).

Services can also be removed from an application as part of an upgrade. However, all current instances of the to-be-deleted service must be stopped before proceeding with the upgrade (using the `Remove-ServiceFabricService` cmdlet).

## Manual upgrade mode
> [!NOTE]
> The unmonitored manual mode should be considered only for a failed or suspended upgrade. The monitored mode is the recommended upgrade mode for Service Fabric applications.
>
>

Azure Service Fabric provides multiple upgrade modes to support development and production clusters. Deployment options chosen may be different for different environments.

The monitored rolling application upgrade is the most typical upgrade to use in the production environment. When the upgrade policy is specified, Service Fabric ensures that the application is healthy before the upgrade proceeds.

 The application administrator can use the manual rolling application upgrade mode to have total control over the upgrade progress through the various upgrade domains. This mode is useful when a customized or complex health evaluation policy is required, or a non-conventional upgrade is happening (for example, the application is already in data loss).

Finally, the automated rolling application upgrade is useful for development or testing environments to provide a fast iteration cycle during service development.

## Change to manual upgrade mode
**Manual**--Stop the application upgrade at the current UD and change the upgrade mode to Unmonitored Manual. The administrator needs to manually call **MoveNextApplicationUpgradeDomainAsync** to proceed with the upgrade or trigger a rollback by initiating a new upgrade. Once the upgrade enters into the Manual mode, it stays in the Manual mode until a new upgrade is initiated. The **GetApplicationUpgradeProgressAsync** command returns FABRIC\_APPLICATION\_UPGRADE\_STATE\_ROLLING\_FORWARD\_PENDING.

## Upgrade with a diff package
A Service Fabric application can be upgraded by provisioning with a full, self-contained application package. An application can also be upgraded by using a diff package that contains only the updated application files, the updated application manifest, and the service manifest files.

A full application package contains all the files necessary to start and run a Service Fabric application. A diff package contains only the files that changed between the last provision and the current upgrade, plus the full application manifest and the service manifest files. Any reference in the application manifest or service manifest that can't be found in the build layout is searched for in the image store.

Full application packages are required for the first installation of an application to the cluster. Subsequent updates can be either a full application package or a diff package.

Occasions when using a diff package would be a good choice:

* A diff package is preferred when you have a large application package that references several service manifest files and/or several code packages, config packages, or data packages.
* A diff package is preferred when you have a deployment system that generates the build layout directly from your application build process. In this case, even though the code hasn't changed, newly built assemblies get a different checksum. Using a full application package would require you to update the version on all code packages. Using a diff package, you only provide the files that changed and the manifest files where the version has changed.

When an application is upgraded using Visual Studio, the diff package is published automatically. To create a diff package manually, the application manifest, and the service manifests must be updated, but only the changed packages should be included in the final application package.

For example, let's start with the following application (version numbers provided for ease of understanding):

```text
app1           1.0.0
  service1     1.0.0
    code       1.0.0
    config     1.0.0
  service2     1.0.0
    code       1.0.0
    config     1.0.0
```

Now, let's assume you wanted to update only the code package of service1 using a diff package using PowerShell. Now, your updated application has the following folder structure:

```text
app1           2.0.0      <-- new version
  service1     2.0.0      <-- new version
    code       2.0.0      <-- new version
    config     1.0.0
  service2     1.0.0
    code       1.0.0
    config     1.0.0
```

In this case, you update the application manifest to 2.0.0, and the service manifest for service1 to reflect the code package update. The folder for your application package would have the following structure:

```text
app1/
  service1/
    code/
```

## Next steps
[Upgrading your Application Using Visual Studio](service-fabric-application-upgrade-tutorial.md) walks you through an application upgrade using Visual Studio.

[Upgrading your Application Using Powershell](service-fabric-application-upgrade-tutorial-powershell.md) walks you through an application upgrade using PowerShell.

Control how your application upgrades by using [Upgrade Parameters](service-fabric-application-upgrade-parameters.md).

Make your application upgrades compatible by learning how to use [Data Serialization](service-fabric-application-upgrade-data-serialization.md).

Fix common problems in application upgrades by referring to the steps in [Troubleshooting Application Upgrades](service-fabric-application-upgrade-troubleshooting.md).
