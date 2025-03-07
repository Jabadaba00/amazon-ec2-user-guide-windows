# Install NVIDIA drivers on Windows instances<a name="install-nvidia-driver"></a>

An instance with an attached NVIDIA GPU, such as a P3 or G4dn instance, must have the appropriate NVIDIA driver installed\. Depending on the instance type, you can either download a public NVIDIA driver, download a driver from Amazon S3 that is available only to AWS customers, or use an AMI with the driver pre\-installed\.

To install AMD drivers on a Linux instance with an attached AMD GPU, such as a G4ad instance, see [Install AMD drivers](install-amd-driver.md) instead\.  To install NVIDIA drivers on a Linux instance, see [Install NVIDIA drivers on a Linux instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/install-nvidia-driver.html)\.

**Contents**
+ [Types of NVIDIA drivers](#nvidia-driver-types)
+ [Available drivers by instance type](#nvidia-driver-instance-type)
+ [Installation options](#nvidia-installation-options)
+ [Install an additional version of CUDA](#gpu-instance-install-cuda)

## Types of NVIDIA drivers<a name="nvidia-driver-types"></a>

The following are the main types of NVIDIA drivers that can be used with GPU\-based instances\.

Tesla drivers  
These drivers are intended primarily for compute workloads, which use GPUs for computational tasks such as parallelized floating\-point calculations for machine learning and fast Fourier transforms for high performance computing applications\.

GRID drivers  
These drivers are certified to provide optimal performance for professional visualization applications that render content such as 3D models or high\-resolution videos\. You can configure GRID drivers to support two modes\. Quadro Virtual Workstations provide access to four 4K displays per GPU\. GRID vApps provide RDSH App hosting capabilities\.

Gaming drivers  
These drivers contain optimizations for gaming and are updated frequently to provide performance enhancements\. They support a single 4K display per GPU\.

**Configured mode**  
On Windows, the Tesla drivers are configured to run in Tesla Compute Cluster \(TCC\) mode\. The GRID and gaming drivers are configured to run in Windows Display Driver Model \(WDDM\) mode\. In TCC mode, the card is dedicated to compute workloads\. In WDDM mode, the card supports both compute and graphics workloads\.

**NVIDIA control panel**  
The NVIDIA control panel is supported with GRID and Gaming drivers\. It is not supported with Tesla drivers\.

**Supported APIs for Tesla drivers**
+ OpenCL
+ NVIDIA CUDA and related libraries \(for example, cuDNN, TensorRT, nvJPEG, and cuBLAS\)
+ NVENC for video encoding and NVDEC for video decoding

**Supported APIs for GRID and gaming drivers**
+ DirectX, Direct2D, DirectX Video Acceleration, DirectX Raytracing
+ OpenCL, OpenGL, and Vulkan
+ NVIDIA CUDA and related libraries \(for example, cuDNN, TensorRT, nvJPEG, and cuBLAS\)
+ NVENC for video encoding and NVDEC for video decoding

## Available drivers by instance type<a name="nvidia-driver-instance-type"></a>

The following table summarizes the supported NVIDIA drivers for each GPU instance type\.


| Instance type | Tesla driver | GRID driver | Gaming driver | 
| --- | --- | --- | --- | 
| G2 | Yes | No | No | 
| G3 | Yes | Yes | No | 
| G4dn | Yes | Yes | Yes | 
| G5 | Yes | Yes | Yes | 
| P2 | Yes | No | No | 
| P3 | Yes | Yes ² | No | 

¹ This Tesla driver also supports optimized graphics applications specific to the ARM64 platform

## Installation options<a name="nvidia-installation-options"></a>

Use one of the following options to get the NVIDIA drivers required for your GPU instance\.

### Option 1: AMIs with the NVIDIA drivers installed<a name="preinstalled-nvidia-driver"></a>

AWS and NVIDIA offer different Amazon Machine Images \(AMI\) that come with the NVIDIA drivers installed\.
+ [Marketplace offerings with the Tesla driver](http://aws.amazon.com/marketplace/search/results?page=1&filters=VendorId&VendorId=e6a5002c-6dd0-4d1e-8196-0a1d1857229b%2Cc568fe05-e33b-411c-b0ab-047218431da9&searchTerms=tesla+driver)
+ [Marketplace offerings with the GRID driver](http://aws.amazon.com/marketplace/search/results?&searchTerms=NVIDIA+quadro)
+ [Marketplace offerings with the Gaming driver](http://aws.amazon.com/marketplace/search/results?searchTerms=NVIDIA+gaming)

If you create a custom Windows AMI using one of the AWS Marketplace offerings, the AMI must be a standardized image created [using Sysprep](Creating_EBSbacked_WinAMI.md#ami-create-standard) to ensure that the GRID driver works\.

### Option 2: Public NVIDIA drivers<a name="public-nvidia-driver"></a>

The options offered by AWS come with the necessary license for the driver\. Alternatively, you can install the public drivers and bring your own license\. To install a public driver, download it from the NVIDIA site as described here\.

Alternatively, you can use the options offered by AWS instead of the public drivers\. To use a GRID driver on a P3 instance, use the AWS Marketplace AMIs as described in [Option 1](#preinstalled-nvidia-driver)\. To use a GRID driver on a G5, G4dn, or G3 instance, use the AWS Marketplace AMIs, as described in Option 1 or install the NVIDIA drivers provided by AWS as described in [Option 3](#nvidia-GRID-driver)\.

**To download a public NVIDIA driver**  
Log on to your Windows instance and download the 64\-bit NVIDIA driver appropriate for the instance type from [http://www\.nvidia\.com/Download/Find\.aspx](http://www.nvidia.com/Download/Find.aspx)\. For **Product Type**, **Product Series**, and **Product**, use the options in the following table\.


| Instance | Product Type | Product Series | Product | 
| --- | --- | --- | --- | 
| G2 | GRID | GRID Series | GRID K520 | 
| G3 | Tesla | M\-Class | M60 | 
| G4dn | Tesla | T\-Series | T4 | 
| G5 ¹ | Tesla | A\-Series | A10 | 
| P2 | Tesla | K\-Series | K80 | 
| P3 | Tesla | V\-Series | V100 | 

¹ G5 instances require driver version 470\.00 or later

**To install the NVIDIA driver on Windows**

1. Open the folder where you downloaded the driver and launch the installation file\. Follow the instructions to install the driver and reboot your instance as required\.

1. Disable the built\-in display adapter using Device Manager\. Install these Windows features: **Media Foundation** and **Quality Windows Audio Video Experience**\.

1. Check Device Manager to verify that the GPU is working correctly\.

1. To achieve the best performance from your GPU, complete the optimization steps in [Optimize GPU settings](optimize_gpu.md)\.

### Option 3: GRID drivers \(G5, G4dn, and G3 instances\)<a name="nvidia-GRID-driver"></a>

These downloads are available to AWS customers only\. By downloading, you agree to use the downloaded software only to develop AMIs for use with the NVIDIA A10G, NVIDIA Tesla T4, or NVIDIA Tesla M60 hardware\. Upon installation of the software, you are bound by the terms of the [NVIDIA GRID Cloud End User License Agreement](https://aws-nvidia-license-agreement.s3.amazonaws.com/NvidiaGridAWSUserLicenseAgreement.DOCX)\.

**Prerequisites**
+ If you launch your Windows instance using a custom Windows AMI, the AMI must be a standardized image created [using Sysprep](Creating_EBSbacked_WinAMI.md#ami-create-standard) to ensure that the GRID driver works\.
+ Configure default credentials for the AWS Tools for Windows PowerShell on your Windows instance\. For more information, see [Getting Started with the AWS Tools for Windows PowerShell](https://docs.aws.amazon.com/powershell/latest/userguide/pstools-getting-started.html) in the *AWS Tools for Windows PowerShell User Guide*\.
+ Your user or role must have the permissions granted that contains the **AmazonS3ReadOnlyAccess** policy\. For more information, see [AWS managed policy: AmazonS3ReadOnlyAccess](https://docs.aws.amazon.com/AmazonS3/latest/userguide/security-iam-awsmanpol.html#security-iam-awsmanpol-amazons3readonlyaccess) in the *Amazon Simple Storage Service User Guide*\.
+ G3 instances require AWS provided DNS resolution for GRID licensing to work\.
+ [IMDSv2](configuring-instance-metadata-service.md) is only supported with NVIDIA driver version 14\.0 or greater\. 

**To install the NVIDIA GRID driver on your Windows instance**

1. Connect to your Windows instance and open a PowerShell window\.

1. Download the drivers and the [NVIDIA GRID Cloud End User License Agreement](https://aws-nvidia-license-agreement.s3.amazonaws.com/NvidiaGridAWSUserLicenseAgreement.DOCX) from Amazon S3 to your desktop using the following PowerShell commands\.

   ```
   $Bucket = "ec2-windows-nvidia-drivers"
   $KeyPrefix = "latest"
   $LocalPath = "$home\Desktop\NVIDIA"
   $Objects = Get-S3Object -BucketName $Bucket -KeyPrefix $KeyPrefix -Region us-east-1
   foreach ($Object in $Objects) {
       $LocalFileName = $Object.Key
       if ($LocalFileName -ne '' -and $Object.Size -ne 0) {
           $LocalFilePath = Join-Path $LocalPath $LocalFileName
           Copy-S3Object -BucketName $Bucket -Key $Object.Key -LocalFile $LocalFilePath -Region us-east-1
       }
   }
   ```

   Multiple versions of the NVIDIA GRID driver are stored in this bucket\. You can download all of the available Windows versions in the bucket by removing the `-KeyPrefix $KeyPrefix` option\.

   Starting with GRID version 11\.0, you can use the drivers under `latest` for both G3 and G4dn instances\. We will not add versions later than 11\.0 to `g4/latest`, but will keep version 11\.0 and the earlier versions specific to G4dn under `g4/latest`\.

   G5 instances require GRID 13\.1 or later \(or GRID 12\.4 or later\)\.

1. Navigate to the desktop and double\-click the installation file to launch it \(choose the driver version that corresponds to your instance OS version\)\. Follow the instructions to install the driver and reboot your instance as required\. To verify that the GPU is working properly, check Device Manager\.

1. \(Optional\) Use the following command to disable the licensing page in the control panel to prevent users from accidentally changing the product type \(NVIDIA GRID Virtual Workstation is enabled by default\)\. For more information, see the [GRID Licensing User Guide](http://docs.nvidia.com/grid/4.6/grid-licensing-user-guide/index.html)\.

   ```
   New-ItemProperty -Path "HKLM:\SOFTWARE\NVIDIA Corporation\Global\GridLicensing" -Name "NvCplDisableManageLicensePage" -PropertyType "DWord" -Value "1"
   ```

1. \(Optional\) Depending on your use case, you might complete the following optional steps\. If you do not require this functionality, do not complete these steps\.

   1. To help take advantage of the four displays of up to 4K resolution, set up the high\-performance display protocol, [NICE DCV](https://docs.aws.amazon.com/dcv/)\.

   1. NVIDIA Quadro Virtual Workstation mode is enabled by default\. To activate GRID Virtual Applications for RDSH Application hosting capabilities, complete the GRID Virtual Application activation steps in [Activate NVIDIA GRID Virtual Applications](activate_grid.md)\.

### Option 4: NVIDIA gaming drivers \(G5 and G4dn instances\)<a name="nvidia-gaming-driver"></a>

These drivers are available to AWS customers only\. By downloading them, you agree to use the downloaded software only to develop AMIs for use with the NVIDIA A10G and NVIDIA Tesla T4 hardware\. Upon installation of the software, you are bound by the terms of the [NVIDIA GRID Cloud End User License Agreement](https://aws-nvidia-license-agreement.s3.amazonaws.com/NvidiaGridAWSUserLicenseAgreement.DOCX)\.

**Prerequisites**
+ If you launch your Windows instance using a custom Windows AMI, the AMI must be a standardized image created [using Sysprep](Creating_EBSbacked_WinAMI.md#ami-create-standard) to ensure that the gaming driver works\.
+ Configure default credentials for the AWS Tools for Windows PowerShell on your Windows instance\. For more information, see [Getting Started with the AWS Tools for Windows PowerShell](https://docs.aws.amazon.com/powershell/latest/userguide/pstools-getting-started.html) in the *AWS Tools for Windows PowerShell User Guide*\.
+ Your users or role must have the permissions granted that contains the **AmazonS3ReadOnlyAccess** policy\. For more information, see [AWS managed policy: AmazonS3ReadOnlyAccess](https://docs.aws.amazon.com/AmazonS3/latest/userguide/security-iam-awsmanpol.html#security-iam-awsmanpol-amazons3readonlyaccess) in the *Amazon Simple Storage Service User Guide*\.
+ G3 instances require AWS provided DNS resolution for GRID licensing to work\.
+ [IMDSv2](configuring-instance-metadata-service.md) is only supported with NVIDIA driver version 495\.x or greater\. 

**To install the NVIDIA gaming driver on your Windows instance**
**Note**  


1. Connect to your Windows instance and open a PowerShell window\.

1. Download and install the gaming driver using the following PowerShell commands\.

   ```
   $Bucket = "nvidia-gaming"
   $KeyPrefix = "windows/latest"
   $LocalPath = "$home\Desktop\NVIDIA"
   $Objects = Get-S3Object -BucketName $Bucket -KeyPrefix $KeyPrefix -Region us-east-1
   foreach ($Object in $Objects) {
       $LocalFileName = $Object.Key
       if ($LocalFileName -ne '' -and $Object.Size -ne 0) {
           $LocalFilePath = Join-Path $LocalPath $LocalFileName
           Copy-S3Object -BucketName $Bucket -Key $Object.Key -LocalFile $LocalFilePath -Region us-east-1
       }
   }
   ```

   Multiple versions of the NVIDIA GRID driver are stored in this S3 bucket\. You can download all of the available versions in the bucket if you change the value of the `$KeyPrefix` variable from *"windows/latest"* to *"windows"*\.

1. Navigate to the desktop and double\-click the installation file to launch it \(choose the driver version that corresponds to your instance OS version\)\. Follow the instructions to install the driver and reboot your instance as required\. To verify that the GPU is working properly, check Device Manager\.

1. Use one of the following methods to register the driver\.

------
#### [ Version 527\.27 or above ]

   Create the following registry key with the 64\-bit version of PowerShell, or the Command Prompt window\.

   *key*: `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\nvlddmkm\Global`

   *name*: vGamingMarketplace

   *type*: DWord

   *value*: 2

**PowerShell**  
Run the following PowerShell command to create this registry value\. The AWS Tools for PowerShell in AWS Windows AMIs defaults to the 32\-bit version and this command fails\. Instead, use the 64\-bit version of PowerShell included with the operating system\.

   ```
   New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\nvlddmkm\Global" -Name "vGamingMarketplace" -PropertyType "DWord" -Value "2"
   ```

**Command Prompt**  
Run the following registry command to create this registry value\. You can run it using the Command Prompt window or a 64\-bit version of PowerShell\.

   ```
   reg add "HKLM\SYSTEM\CurrentControlSet\Services\nvlddmkm\Global" /v vGamingMarketplace /t REG_DWORD /d 2
   ```

------
#### [ Earlier versions ]

   Create the following registry key with the 64\-bit version of PowerShell, or the Command Prompt window\.

   *key*: `HKEY_LOCAL_MACHINE\SOFTWARE\NVIDIA Corporation\Global`

   *name*: vGamingMarketplace

   *type*: DWord

   *value*: 2

**PowerShell**  
Run the following PowerShell command to create this registry value\. The AWS Tools for PowerShell in AWS Windows AMIs defaults to the 32\-bit version and this command fails\. Instead, use the 64\-bit version of PowerShell included with the operating system\.

   ```
   New-ItemProperty -Path "HKLM:\SOFTWARE\NVIDIA Corporation\Global" -Name "vGamingMarketplace" -PropertyType "DWord" -Value "2"
   ```

**Command Prompt**  
Run the following registry command to create this registry key with the Command Prompt window\. You can also use this command in the 64\-bit version of PowerShell\.

   ```
   reg add "HKLM\SOFTWARE\NVIDIA Corporation\Global" /v vGamingMarketplace /t REG_DWORD /d 2
   ```

------

1. Run the following command in PowerShell\. This downloads the certification file, renames the file `GridSwCert.txt`, and moves the file to the Public Documents folder on your system drive\. Typically, the folder path is `C:\Users\Public\Documents`\.
   + For version 461\.40 or later:

     ```
     Invoke-WebRequest -Uri "https://nvidia-gaming.s3.amazonaws.com/GridSwCert-Archive/GridSwCertWindows_2021_10_2.cert" -OutFile "$Env:PUBLIC\Documents\GridSwCert.txt"
     ```
   + For version 445\.87:

     ```
     Invoke-WebRequest -Uri "https://nvidia-gaming.s3.amazonaws.com/GridSwCert-Archive/GridSwCert-Windows_2020_04.cert" -OutFile "$Env:PUBLIC\Documents\GridSwCert.txt"
     ```
   + For earlier versions:

     ```
     Invoke-WebRequest -Uri "https://nvidia-gaming.s3.amazonaws.com/GridSwCert-Archive/GridSwCert-Windows_2019_09.cert" -OutFile "$Env:PUBLIC\Documents\GridSwCert.txt"
     ```

1. Reboot your instance\.

1. Verify the NVIDIA Gaming license using the following command\.

   ```
   "C:\Windows\System32\DriverStore\FileRepository\nvgrid*\\nvidia-smi.exe" -q
   ```

   The output should be similar to the following\.

   ```
   vGPU Software Licensed Product
       Product Name              : NVIDIA Cloud Gaming
       License Status            : Licensed (Expiry: N/A)
   ```

1. \(Optional\) To help take advantage of the single display of up to 4K resolution, set up the high\-performance display protocol [NICE DCV](https://docs.aws.amazon.com/dcv)\. If you do not require this functionality, do not complete this step\.

## Install an additional version of CUDA<a name="gpu-instance-install-cuda"></a>

After you install an NVIDIA graphics driver on your instance, you can install a version of CUDA other than the version that is bundled with the graphics driver\. The following procedure demonstrates how to configure multiple versions of CUDA on the instance\.

**To install the CUDA toolkit**

1. Connect to your Windows instance\.

1. Open the [NVIDIA website](https://developer.nvidia.com/cuda-downloads?target_os=Windows&target_arch=x86_64) and select the version of CUDA that you need\.

1. For **Installer Type**, select **exe \(local\)** and then choose **Download**\.

1. Using your browser, run the downloaded install file\. Follow the instructions to install the CUDA toolkit\. You might be required to reboot the instance\.