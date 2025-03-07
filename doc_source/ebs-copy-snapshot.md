# Copy an Amazon EBS snapshot<a name="ebs-copy-snapshot"></a>

With Amazon EBS, you can create point\-in\-time snapshots of volumes, which we store for you in Amazon S3\. After you create a snapshot and it has finished copying to Amazon S3 \(when the snapshot status is `completed`\), you can copy it from one AWS Region to another, or within the same Region\. Amazon S3 server\-side encryption \(256\-bit AES\) protects a snapshot's data in transit during a copy operation\. The snapshot copy receives an ID that is different from the ID of the original snapshot\.

To copy multi\-volume snapshots to another AWS Region, retrieve the snapshots using the tag you applied to the multi\-volume snapshot set when you created it\. Then individually copy the snapshots to another Region\.

If you would like another account to be able to copy your snapshot, you must either modify the snapshot permissions to allow access to that account or make the snapshot public so that all AWS accounts can copy it\. For more information, see [Share an Amazon EBS snapshot](ebs-modifying-snapshot-permissions.md)\.

For information about copying an Amazon RDS snapshot, see [Copying a DB Snapshot](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_CopySnapshot.html) in the *Amazon RDS User Guide*\.

**Use cases**
+ Geographic expansion: Launch your applications in a new AWS Region\.
+ Migration: Move an application to a new Region, to enable better availability and to minimize cost\.
+ Disaster recovery: Back up your data and logs across different geographical locations at regular intervals\. In case of disaster, you can restore your applications using point\-in\-time backups stored in the secondary Region\. This minimizes data loss and recovery time\.
+ Encryption: Encrypt a previously unencrypted snapshot, change the key with which the snapshot is encrypted, or create a copy that you own in order to create a volume from it \(for encrypted snapshots that have been shared with you\)\.
+ Data retention and auditing requirements: Copy your encrypted EBS snapshots from one AWS account to another to preserve data logs or other files for auditing or data retention\. Using a different account helps prevent accidental snapshot deletions, and protects you if your main AWS account is compromised\.

**Prerequisites**
+ You can copy any accessible snapshots that have a `completed` status, including shared snapshots and snapshots that you have created\.
+ You can copy AWS Marketplace, VM Import/Export, and Storage Gateway snapshots, but you must verify that the snapshot is supported in the destination Region\.

**Considerations**
+ There is a limit of `20` concurrent snapshot copy requests per destination Region\. If you exceed this quota, you receive a `ResourceLimitExceeded` error\. If you receive this error, wait for one or more of the copy requests to complete before making a new snapshot copy request\.
+ User\-defined tags are not copied from the source snapshot to the new snapshot\. You can add user\-defined tags during or after the copy operation\. For more information, see [Tag your Amazon EC2 resources](Using_Tags.md)\.
+ Snapshots created by a snapshot copy operation have an arbitrary volume ID that should not be used for any purpose\.
+ Resource\-level permissions specified for the snapshot copy operation apply only to the new snapshot\. You cannot specify resource\-level permissions for the source snapshot\. For an example, see [Example: Copying snapshots](ExamplePolicies_EC2.md#iam-copy-snapshot)\.

**Pricing**
+ For pricing information about copying snapshots across AWS Regions and accounts, see [Amazon EBS Pricing](http://aws.amazon.com/ebs/pricing/)\.
+ Snapshot copy operations within a single account and Region do not copy any actual data and therefore are cost\-free as long as the encryption status of the snapshot copy does not change\.
+ If you copy a snapshot and encrypt it to a new KMS key, a complete \(non\-incremental\) copy is created\. This results in additional storage costs\.
+ If you copy a snapshot to a new Region, a complete \(non\-incremental\) copy is created\. This results in additional storage costs\. Subsequent copies of the same snapshot are incremental\.

## Incremental snapshot copying<a name="ebs-incremental-copy"></a>

Whether a snapshot copy is incremental is determined by the most recently completed snapshot copy\. When you copy a snapshot across Regions or accounts, the copy is an incremental copy if the following conditions are met:
+ The snapshot was copied to the destination Region or account previously\.
+ The most recent snapshot copy still exists in the destination Region or account\.
+ All copies of the snapshot in the destination Region or account are either unencrypted or were encrypted using the same KMS key\.

If the most recent snapshot copy was deleted, the next copy is a full copy, not an incremental copy\. If a copy is still pending when you start a another copy, the second copy starts only after the first copy finishes\.

Incremental snapshot copying reduces the time required to copy snapshots and saves on data transfer and storage costs by not duplicating data\.

We recommend that you tag your snapshots with the volume ID and creation time so that you can keep track of the most recent snapshot copy of a volume in the destination Region or account\.

To see whether your snapshot copies are incremental, check the [copySnapshot](ebs-cloud-watch-events.md#copy-snapshot-complete) CloudWatch event\.

## Encryption and snapshot copying<a name="creating-encrypted-snapshots"></a>

When you copy a snapshot, you can encrypt the copy or you can specify a KMS key that is different than the original, and the resulting copied snapshot uses the new KMS key\. However, changing the encryption status of a snapshot during a copy operation could result in a full \(not incremental\) copy, which might incur greater data transfer and storage charges\. For more information, see [Incremental snapshot copying](#ebs-incremental-copy)\. 

To copy an encrypted snapshot shared from another AWS account, you must have permissions to use the snapshot and the customer master key \(CMK\) that was used to encrypt the snapshot\. When using an encrypted snapshot that was shared with you, we recommend that you re\-encrypt the snapshot by copying it using a KMS key that you own\. This protects you if the original KMS key is compromised, or if the owner revokes it, which could cause you to lose access to any encrypted volumes that you created using the snapshot\. For more information, see [Share an Amazon EBS snapshot](ebs-modifying-snapshot-permissions.md)\.

You apply encryption to EBS snapshot copies by setting the `Encrypted` parameter to `true`\. \(The `Encrypted` parameter is optional if [encryption by default](EBSEncryption.md#encryption-by-default) is enabled\)\.

Optionally, you can use `KmsKeyId` to specify a custom key to use to encrypt the snapshot copy\. \(The `Encrypted` parameter must also be set to `true`, even if encryption by default is enabled\.\) If `KmsKeyId` is not specified, the key that is used for encryption depends on the encryption state of the source snapshot and its ownership\.

The following tables describe the encryption outcome for each possible combination of settings\.

**Topics**
+ [Encryption outcomes: Copying snapshots that you own](#own-snapshots)
+ [Encryption outcomes: Copying snapshots that are shared with you](#shared-snapshots)

### Encryption outcomes: Copying snapshots that you own<a name="own-snapshots"></a>

[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/ebs-copy-snapshot.html)

\*\* This is a customer managed key specified for the copy action\. This customer managed key is used instead of the default customer managed key for the AWS account and Region\.

### Encryption outcomes: Copying snapshots that are shared with you<a name="shared-snapshots"></a>

[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/ebs-copy-snapshot.html)

\*\* This is a customer managed key specified for the copy action\. This customer managed key is used instead of the default customer managed key for the AWS account and Region\.

## Copy a snapshot<a name="ebs-snapshot-copy"></a>

To copy a snapshot, use one of the following methods\.

------
#### [ Console ]

**To copy a snapshot using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Snapshots**\.

1. Select the snapshot to copy, and then choose **Actions**, **Copy snapshot**\.

1. For **Description**, enter a brief description for the snapshot copy\.

   By default, the description includes information about the source snapshot so that you can identify a copy from the original\. You can change this description as needed\.

1. For **Destination Region**, select the Region in which to create the snapshot copy\.

1. Specify the encryption status for the snapshot copy\.

   If the source snapshot is encrypted, or if your account is enabled for [encryption by default](EBSEncryption.md#encryption-by-default), then the snapshot copy is automatically encrypted and you can't change its encryption status\.

   If the source snapshot is unencrypted and your account is not enabled for encryption by default, encryption is optional\. To encrypt the snapshot copy, for **Encryption**, select **Encrypt this snapshot**\. Then, for **KMS key**, select the KMS key to use to encrypt the snapshot in the destination Region\.

1. Choose **Copy snapshot**\.

------
#### [ AWS CLI ]

**To copy a snapshot using the command line**

You can use one of the following commands\. For more information about these command line interfaces, see [Access Amazon EC2](concepts.md#access-ec2)\.
+ [copy\-snapshot](https://docs.aws.amazon.com/cli/latest/reference/ec2/copy-snapshot.html) \(AWS CLI\)
+ [Copy\-EC2Snapshot](https://docs.aws.amazon.com/powershell/latest/reference/items/Copy-EC2Snapshot.html) \(AWS Tools for Windows PowerShell\)

------

**To check for failure**  
If you attempt to copy an encrypted snapshot without having permissions to use the encryption key, the operation fails silently\. The error state is not displayed in the console until you refresh the page\. You can also check the state of the snapshot from the command line, as in the following example\.

```
aws ec2 describe-snapshots --snapshot-id snap-0123abcd
```

If the copy failed because of insufficient key permissions, you see the following message: "StateMessage": "Given key ID is not accessible"\.

When copying an encrypted snapshot, you must have `DescribeKey` permissions on the default CMK\. Explicitly denying these permissions results in copy failure\. For information about managing CMK keys, see [Controlling Access to Customer Master Keys](https://docs.aws.amazon.com/kms/latest/developerguide/control-access.html)\.