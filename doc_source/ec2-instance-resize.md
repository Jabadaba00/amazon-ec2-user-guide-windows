# Change the instance type<a name="ec2-instance-resize"></a>

As your needs change, you might find that your instance is over\-utilized \(the instance type is too small\) or under\-utilized \(the instance type is too large\)\. If this is the case, you can resize your instance by changing its instance type\. For example, if your `t2.micro` instance is too small for its workload, you can increase its size by changing it to a bigger T2 instance type, such as `t2.large`\. Or you can change it to another instance type, such as `m5.large`\. You might also want to change from a previous generation to a current generation instance type to take advantage of some features, such as support for IPv6\.

If you want a recommendation for an instance type that is best able to handle your existing workload, you can use AWS Compute Optimizer\. For more information, see [Get recommendations for an instance type](ec2-instance-recommendations.md)\.

## Which instructions to follow?<a name="choose-instance-resize-instructions"></a>

There are different instructions for changing the instance type\. The instructions to use depend on whether the instance type is compatible with the instance's current configuration\. For information about how compatibility is determined, see [Compatibility for changing the instance type](resize-limitations.md)\.

Use the following table to determine which instructions to follow\.


| Compatibility | Use these instructions | 
| --- | --- | 
| Compatible | [Change the instance type of an existing instance](#change-instance-type-of-ebs-backed-instance) | 
| Not compatible | [Change the instance type by launching a new instance](#migrate-instance-configuration) | 

## Considerations for compatible instance types<a name="resize-ebs-backed-instance-considerations"></a>

Consider the following when changing the instance type of an existing instance:
+ We recommend that you update the AWS PV driver package before changing the instance type\. For more information, see [Upgrade PV drivers on Windows instances](Upgrading_PV_drivers.md)\.
+ You must stop your Amazon EBS\-backed instance before you can change its instance type\. Ensure that you plan for downtime while your instance is stopped\. Stopping the instance and changing its instance type might take a few minutes, and restarting your instance might take a variable amount of time depending on your application's startup scripts\. For more information, see [Stop and start your instance](Stop_Start.md)\.
+ When you stop and start an instance, we move the instance to new hardware\. If your instance has a public IPv4 address, we release the address and give your instance a new public IPv4 address\. If you require a public IPv4 address that does not change, use an [Elastic IP address](elastic-ip-addresses-eip.md)\.
+ You can't change the instance type if [hibernation](Hibernate.md) is enabled for the instance\.
+ You can't change the instance type of a [Spot Instance](spot-requests.md#stopping-a-spot-instance)\.
+ If your instance is in an Auto Scaling group, the Amazon EC2 Auto Scaling service marks the stopped instance as unhealthy, and may terminate it and launch a replacement instance\. To prevent this, you can suspend the scaling processes for the group while you're changing the instance type\. For more information, see [Suspending and resuming a process for an Auto Scaling group](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-suspend-resume-processes.html) in the *Amazon EC2 Auto Scaling User Guide*\.
+ When you change the instance type of an instance with NVMe instance store volumes, the updated instance might have additional instance store volumes, because all NVMe instance store volumes are available even if they are not specified in the AMI or instance block device mapping\. Otherwise, the updated instance has the same number of instance store volumes that you specified when you launched the original instance\.

## Change the instance type of an existing instance<a name="change-instance-type-of-ebs-backed-instance"></a>

Use the following instructions to change the instance type of an instance if the instance type that you need is compatible with the instance's current configuration\.

**To change the instance type of an Amazon EBS\-backed instance**

1. \(Optional\) If the new instance type requires drivers that are not installed on the existing instance, you must connect to your instance and install the drivers first\. For more information, see [Compatibility for changing the instance type](resize-limitations.md)\.

1. \(Optional\) If you configured your Windows instance to use [static IP addressing](config-windows-multiple-ip.md#step1) and you change from an instance type that doesn't support enhanced networking to an instance type that does support enhanced networking, you might get a warning about a potential IP address conflict when you reconfigure static IP addressing\. To prevent this, enable DHCP on the network interface for your instance before you change the instance type\. From your instance, open the **Network and Sharing Center**, open **Internet Protocol Version 4 \(TCP/IPv4\) Properties** for the network interface, and choose **Obtain an IP address automatically**\. Change the instance type and reconfigure static IP addressing on the network interface\.

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Instances**\.

1. Select the instance and choose **Instance state**, **Stop instance**\. When prompted for confirmation, choose **Stop**\. It can take a few minutes for the instance to stop\.

1. With the instance still selected, choose **Actions**, **Instance settings**, **Change instance type**\. This option is grayed out if the instance state is not `stopped`\.

1. On the **Change instance type** page, do the following:

   1. For **Instance type**, select the instance type that you want\.

      If the instance type is not in the list, then it's not compatible with the configuration of your instance\. Instead, use the following instructions: [Change the instance type by launching a new instance](#migrate-instance-configuration)\.

   1. \(Optional\) If the instance type that you selected supports EBS optimization, select **EBS\-optimized** to enable EBS optimization, or deselect **EBS\-optimized** to disable EBS optimization\. If the instance type that you selected is EBS optimized by default, **EBS\-optimized** is selected and you can't deselect it\.

   1. Choose **Apply** to accept the new settings\.

1. To start the instance, select the instance and choose **Instance state**, **Start instance**\. It can take a few minutes for the instance to enter the `running` state\. If your instance won't start, see [Troubleshoot changing the instance type](troubleshoot-change-instance-type.md)\.

1. \[Windows Server 2016 and later\] Connect to your Windows instance and run the following EC2Launch PowerShell script to configure the instance after the instance type is changed\.
**Important**  
The administrator password will reset when you enable the initialize instance EC2 Launch script\. You can modify the configuration file to disable the administrator password reset by specifying it in the settings for the initialization tasks\. For steps on how to disable password reset, see [Configure initialization tasks](ec2launch.md#ec2launch-inittasks)\.

   ```
   PS C:\> C:\ProgramData\Amazon\EC2-Windows\Launch\Scripts\InitializeInstance.ps1 -Schedule
   ```

## Change the instance type by launching a new instance<a name="migrate-instance-configuration"></a>

If the current configuration of your EBS\-backed instance is incompatible with the new instance type that you want, then you can't change the instance type of the original instance\. Instead, you must launch a new instance with a configuration that is compatible with the new instance type that you want, and then migrate your application to the new instance\. For information about how compatibility is determined, see [Compatibility for changing the instance type](resize-limitations.md)\.

To migrate your application to a new instance, do the following:
+ Back up the data on your original instance\.
+ Launch a new instance with a configuration that is compatible with the new instance type that you want, and attach any EBS volumes that were attached to your original instance\.
+ Install your application and any software on your new instance\.
+ Restore any data\.
+ If your original instance has an Elastic IP address, and you want to ensure that your users can continue uninterrupted to use the applications on your new instance, you must associate the Elastic IP address with your new instance\. For more information, see [Elastic IP address](elastic-ip-addresses-eip.md)\.

**To change the instance type for a new instance configuration**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. Back up data that you need to keep, as follows:
   + For data on your instance store volumes, back up the data to persistent storage\.
   + For data on your EBS volumes, [take a snapshot of the volumes](ebs-creating-snapshot.md) or [detach the volumes from the instance](ebs-detaching-volume.md) so that you can attach them to the new instance later\.

1. In the navigation pane, choose **Instances**\.

1. Choose **Launch instances**\. When you configure the instance, do the following:

   1. Select an AMI that will support the instance type that you want\. 

   1. Select the new instance type that you want\. If the instance type that you want isn't available, then it's not compatible with the configuration of the AMI that you selected\.

   1. If you're using an Elastic IP address, select the VPC that the original instance is currently running in\.

   1. If you want to allow the same traffic to reach the new instance, select the security group that is associated with the original instance\.

   1. When you're done configuring your new instance, complete the steps to select a key pair and launch your instance\. It can take a few minutes for the instance to enter the `running` state\.

1. If required, [attach any new EBS volumes](ebs-attaching-volume.md) based on the snapshots that you created, or any EBS volumes that you detached from the original instance, to the new instance\.

1. Install your application and any required software on the new instance\.

1. Restore any data that you backed up from the instance store volumes of the original instance\.

1. If you are using an Elastic IP address, assign it to the new instance as follows:

   1. In the navigation pane, choose **Elastic IPs**\.

   1. Select the Elastic IP address that is associated with the original instance and choose **Actions**, **Disassociate Elastic IP address**\. When prompted for confirmation, choose **Disassociate**\.

   1. With the Elastic IP address still selected, choose **Actions**, **Associate Elastic IP address**\.

   1. For **Resource type**, choose **Instance**\. 

   1. For **Instance**, choose the new instance with which to associate the Elastic IP address\.

   1. \(Optional\) For **Private IP address**, specify a private IP address with which to associate the Elastic IP address\.

   1. Choose **Associate**\.

1. \(Optional\) You can terminate the original instance if it's no longer needed\. Select the instance, verify that you are about to terminate the original instance and not the new instance \(for example, check the name or launch time\), and then choose **Instance state**, **Terminate instance**\.