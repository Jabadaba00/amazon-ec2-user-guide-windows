# Retrieve instance metadata<a name="instancedata-data-retrieval"></a>

Because your instance metadata is available from your running instance, you do not need to use the Amazon EC2 console or the AWS CLI\. This can be helpful when you're writing scripts to run from your instance\. For example, you can access the local IP address of your instance from instance metadata to manage a connection to an external application\.

Instance metadata is divided into categories\. For a description of each instance metadata category, see [Instance metadata categories](instancedata-data-categories.md)\.

To view all categories of instance metadata from within a running instance, use the following IPv4 or IPv6 URIs\.

**IPv4**

```
http://169.254.169.254/latest/meta-data/
```

**IPv6**

```
http://[fd00:ec2::254]/latest/meta-data/
```

The IP addresses are link\-local addresses and are valid only from the instance\. For more information, see [Link\-local address](https://en.wikipedia.org/wiki/Link-local_address) on Wikipedia\.

**Note**  
The examples in this section use the IPv4 address of the instance metadata service: `169.254.169.254`\. If you are retrieving instance metadata for EC2 instances over the IPv6 address, ensure that you enable and use the IPv6 address instead: `fd00:ec2::254`\. The IPv6 address of the instance metadata service is compatible with IMDSv2 commands\. The IPv6 address is only accessible on [Instances built on the Nitro System](instance-types.md#ec2-nitro-instances)\.

The command format is different, depending on whether you use IMDSv1 or IMDSv2\. By default, you can use both instance metadata services\. To require the use of IMDSv2, see [Use IMDSv2](configuring-instance-metadata-service.md)\.

You can use PowerShell cmdlets to retrieve the URI\. For example, if you are running version 3\.0 or later of PowerShell, use the following cmdlet\.

------
#### [ IMDSv2 ]

```
PS C:\> [string]$token = Invoke-RestMethod -Headers @{"X-aws-ec2-metadata-token-ttl-seconds" = "21600"} -Method PUT -Uri http://169.254.169.254/latest/api/token
```

```
PS C:\> Invoke-RestMethod -Headers @{"X-aws-ec2-metadata-token" = $token} -Method GET -Uri http://169.254.169.254/latest/meta-data/
```

------
#### [ IMDSv1 ]

```
PS C:\> Invoke-RestMethod -uri http://169.254.169.254/latest/meta-data/
```

------

If you don't want to use PowerShell, you can install a third\-party tool such as GNU Wget or cURL\.

**Important**  
If you install a third\-party tool on a Windows instance, ensure that you read the accompanying documentation carefully, as the method of calling the HTTP and the output format might be different from what is documented here\.

For the command to retrieve instance metadata from a Linux instance, see [Retrieve instance metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html) in the *Amazon EC2 User Guide for Windows Instances*\.

## Costs<a name="imds-costs"></a>

You are not billed for HTTP requests used to retrieve instance metadata and user data\.

## Considerations<a name="imds-considerations"></a>

To avoid problems with instance metadata retrieval, consider the following:
+ The AWS SDKs use IMDSv2 calls by default\. If the IMDSv2 call receives no response, the SDK retries the call and, if still unsuccessful, uses IMDSv1\. This can result in a delay\. In a container environment, if the hop limit is 1, the IMDSv2 response does not return because going to the container is considered an additional network hop\. To avoid the process of falling back to IMDSv1 and the resultant delay, in a container environment we recommend that you set the hop limit to 2\. For more information, see [Configure the instance metadata options](configuring-instance-metadata-options.md)\.
+ If you launch a Windows instance using a custom Windows AMI, to ensure that the instance metadata service works on the instance, the AMI must be a standardized image created [using Sysprep](Creating_EBSbacked_WinAMI.md#ami-create-standard)\. Otherwise, the instance metadata service won't work\.
+ For IMDSv2, you must use `/latest/api/token` when retrieving the token\. Issuing `PUT` requests to any version\-specific path, for example `/2021-03-23/api/token`, will result in the metadata service returning 403 Forbidden errors\. This behavior is intended\. 

  

## Responses and error messages<a name="instance-metadata-returns"></a>

All instance metadata is returned as text \(HTTP content type `text/plain`\)\.

A request for a specific metadata resource returns the appropriate value, or a `404 - Not Found` HTTP error code if the resource is not available\. 

A request for a general metadata resource \(the URI ends with a /\) returns a list of available resources, or a `404 - Not Found` HTTP error code if there is no such resource\. The list items are on separate lines, terminated by line feeds \(ASCII 10\)\.

For requests made using Instance Metadata Service Version 2, the following HTTP error codes can be returned:
+ `400 - Missing or Invalid Parameters` – The `PUT` request is not valid\.
+ `401 - Unauthorized` – The `GET` request uses an invalid token\. The recommended action is to generate a new token\.
+ `403 - Forbidden` – The request is not allowed or the instance metadata service is turned off\.

## Examples of retrieving instance metadata<a name="instancedata-meta-data-retrieval-examples"></a>

The following examples provide commands that you can use on a Windows instance\. For the commands to retrieve instance metadata from a Linux instance, see [Retrieve instance metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html) in the *Amazon EC2 User Guide for Windows Instances*\.

**Topics**
+ [Get the available versions of the instance metadata](#instance-metadata-ex-1)
+ [Get the top\-level metadata items](#instance-metadata-ex-2)
+ [Get the list of available public keys](#instance-metadata-ex-3)
+ [Show the formats in which public key 0 is available](#instance-metadata-ex-4)
+ [Get public key 0 \(in the OpenSSH key format\)](#instance-metadata-ex-5)
+ [Get the subnet ID for an instance](#instance-metadata-ex-6)
+ [Get the instance tags for an instance](#instance-metadata-ex-7)

### Get the available versions of the instance metadata<a name="instance-metadata-ex-1"></a>

This example gets the available versions of the instance metadata\. Each version refers to an instance metadata build when new instance metadata categories were released\. The instance metadata build versions do not correlate with the Amazon EC2 API versions\. The earlier versions are available to you in case you have scripts that rely on the structure and information present in a previous version\.

**Note**  
To avoid having to update your code every time Amazon EC2 releases a new instance metadata build, we recommend that you use `latest` in the path, and not the version number\. For example, use `latest` as follows:  
`curl http://169.254.169.254/latest/meta-data/ami-id`

------
#### [ IMDSv2 ]

```
PS C:\> [string]$token = Invoke-RestMethod -Headers @{"X-aws-ec2-metadata-token-ttl-seconds" = "21600"} -Method PUT -Uri http://169.254.169.254/latest/api/token
```

```
PS C:\> Invoke-RestMethod -Headers @{"X-aws-ec2-metadata-token" = $token} -Method GET -Uri http://169.254.169.254/
1.0
2007-01-19
2007-03-01
2007-08-29
2007-10-10
2007-12-15
2008-02-01
2008-09-01
2009-04-04
2011-01-01
2011-05-01
2012-01-12
2014-02-25
2014-11-05
2015-10-20
2016-04-19
...
latest
```

------
#### [ IMDSv1 ]

```
PS C:\> Invoke-RestMethod -uri http://169.254.169.254/
1.0
2007-01-19
2007-03-01
2007-08-29
2007-10-10
2007-12-15
2008-02-01
2008-09-01
2009-04-04
2011-01-01
2011-05-01
2012-01-12
2014-02-25
2014-11-05
2015-10-20
2016-04-19
...
latest
```

------

### Get the top\-level metadata items<a name="instance-metadata-ex-2"></a>

This example gets the top\-level metadata items\. For more information, see [Instance metadata categories](instancedata-data-categories.md)\.

------
#### [ IMDSv2 ]

```
PS C:\> [string]$token = Invoke-RestMethod -Headers @{"X-aws-ec2-metadata-token-ttl-seconds" = "21600"} -Method PUT -Uri http://169.254.169.254/latest/api/token
```

```
PS C:\> Invoke-RestMethod -Headers @{"X-aws-ec2-metadata-token" = $token} -Method GET -Uri http://169.254.169.254/latest/meta-data/
ami-id
ami-launch-index
ami-manifest-path
block-device-mapping/
hostname
iam/
instance-action
instance-id
instance-life-cycle
instance-type
local-hostname
local-ipv4
mac
metrics/
network/
placement/
profile
public-hostname
public-ipv4
public-keys/
reservation-id
security-groups
services/
```

------
#### [ IMDSv1 ]

```
PS C:\> Invoke-RestMethod -uri http://169.254.169.254/latest/meta-data/    
ami-id
ami-launch-index
ami-manifest-path
block-device-mapping/
hostname
iam/
instance-action
instance-id
instance-type
local-hostname
local-ipv4
mac
metrics/
network/
placement/
profile
public-hostname
public-ipv4
public-keys/
reservation-id
security-groups
services/
```

------

The following examples get the values of some of the top\-level metadata items that were obtained in the preceding example\. The IMDSv2 requests use the stored token that was created in the preceding example command, assuming it has not expired\.

------
#### [ IMDSv2 ]

```
PS C:\> Invoke-RestMethod -Headers @{"X-aws-ec2-metadata-token" = $token} -Method GET -Uri http://169.254.169.254/latest/meta-data/ami-id
ami-0abcdef1234567890
```

------
#### [ IMDSv1 ]

```
PS C:\> Invoke-RestMethod -uri http://169.254.169.254/latest/meta-data/ami-id
ami-0abcdef1234567890
```

------

 

------
#### [ IMDSv2 ]

```
PS C:\> Invoke-RestMethod -Headers @{"X-aws-ec2-metadata-token" = $token} -Method GET -Uri http://169.254.169.254/latest/meta-data/reservation-id
r-0efghijk987654321
```

------
#### [ IMDSv1 ]

```
PS C:\> Invoke-RestMethod -uri http://169.254.169.254/latest/meta-data/reservation-id
r-0efghijk987654321
```

------

 

------
#### [ IMDSv2 ]

```
PS C:\> Invoke-RestMethod -Headers @{"X-aws-ec2-metadata-token" = $token} -Method GET -Uri http://169.254.169.254/latest/meta-data/local-hostname
ip-10-251-50-12.ec2.internal
```

------
#### [ IMDSv1 ]

```
PS C:\> Invoke-RestMethod -uri http://169.254.169.254/latest/meta-data/local-hostname
ip-10-251-50-12.ec2.internal
```

------

 

------
#### [ IMDSv2 ]

```
PS C:\> Invoke-RestMethod -Headers @{"X-aws-ec2-metadata-token" = $token} -Method GET -Uri http://169.254.169.254/latest/meta-data/public-hostname
ec2-203-0-113-25.compute-1.amazonaws.com
```

------
#### [ IMDSv1 ]

```
PS C:\> Invoke-RestMethod -uri http://169.254.169.254/latest/meta-data/public-hostname
ec2-203-0-113-25.compute-1.amazonaws.com
```

------

### Get the list of available public keys<a name="instance-metadata-ex-3"></a>

This example gets the list of available public keys\.

------
#### [ IMDSv2 ]

```
PS C:\> [string]$token = Invoke-RestMethod -Headers @{"X-aws-ec2-metadata-token-ttl-seconds" = "21600"} -Method PUT -Uri http://169.254.169.254/latest/api/token
```

```
PS C:\> Invoke-RestMethod -Headers @{"X-aws-ec2-metadata-token" = $token} -Method GET -Uri http://169.254.169.254/latest/meta-data/public-keys/
0=my-public-key
```

------
#### [ IMDSv1 ]

```
PS C:\> Invoke-RestMethod -uri http://169.254.169.254/latest/meta-data/public-keys/ 0=my-public-key
```

------

### Show the formats in which public key 0 is available<a name="instance-metadata-ex-4"></a>

This example shows the formats in which public key 0 is available\.

------
#### [ IMDSv2 ]

```
PS C:\> [string]$token = Invoke-RestMethod -Headers @{"X-aws-ec2-metadata-token-ttl-seconds" = "21600"} -Method PUT -Uri http://169.254.169.254/latest/api/token
```

```
PS C:\> Invoke-RestMethod -Headers @{"X-aws-ec2-metadata-token" = $token} -Method GET -Uri http://169.254.169.254/latest/meta-data/public-keys/0/openssh-key
openssh-key
```

------
#### [ IMDSv1 ]

```
PS C:\> Invoke-RestMethod -uri http://169.254.169.254/latest/meta-data/public-keys/0/openssh-key
openssh-key
```

------

### Get public key 0 \(in the OpenSSH key format\)<a name="instance-metadata-ex-5"></a>

This example gets public key 0 \(in the OpenSSH key format\)\.

------
#### [ IMDSv2 ]

```
PS C:\> [string]$token = Invoke-RestMethod -Headers @{"X-aws-ec2-metadata-token-ttl-seconds" = "21600"} -Method PUT -Uri http://169.254.169.254/latest/api/token
```

```
PS C:\> Invoke-RestMethod -Headers @{"X-aws-ec2-metadata-token" = $token} -Method GET -Uri http://169.254.169.254/latest/meta-data/public-keys/0/openssh-key
ssh-rsa MIICiTCCAfICCQD6m7oRw0uXOjANBgkqhkiG9w0BAQUFADCBiDELMAkGA1UEBhMC
VVMxCzAJBgNVBAgTAldBMRAwDgYDVQQHEwdTZWF0dGxlMQ8wDQYDVQQKEwZBbWF6
b24xFDASBgNVBAsTC0lBTSBDb25zb2xlMRIwEAYDVQQDEwlUZXN0Q2lsYWMxHzAd
BgkqhkiG9w0BCQEWEG5vb25lQGFtYXpvbi5jb20wHhcNMTEwNDI1MjA0NTIxWhcN
MTIwNDI0MjA0NTIxWjCBiDELMAkGA1UEBhMCVVMxCzAJBgNVBAgTAldBMRAwDgYD
VQQHEwdTZWF0dGxlMQ8wDQYDVQQKEwZBbWF6b24xFDASBgNVBAsTC0lBTSBDb25z
b2xlMRIwEAYDVQQDEwlUZXN0Q2lsYWMxHzAdBgkqhkiG9w0BCQEWEG5vb25lQGFt
YXpvbi5jb20wgZ8wDQYJKoZIhvcNAQEBBQADgY0AMIGJAoGBAMaK0dn+a4GmWIWJ
21uUSfwfEvySWtC2XADZ4nB+BLYgVIk60CpiwsZ3G93vUEIO3IyNoH/f0wYK8m9T
rDHudUZg3qX4waLG5M43q7Wgc/MbQITxOUSQv7c7ugFFDzQGBzZswY6786m86gpE
Ibb3OhjZnzcvQAaRHhdlQWIMm2nrAgMBAAEwDQYJKoZIhvcNAQEFBQADgYEAtCu4
nUhVVxYUntneD9+h8Mg9q6q+auNKyExzyLwaxlAoo7TJHidbtS4J5iNmZgXL0Fkb
FFBjvSfpJIlJ00zbhNYS5f6GuoEDmFJl0ZxBHjJnyp378OD8uTs7fLvjx79LjSTb
NYiytVbZPQUQ5Yaxu2jXnimvw3rrszlaEXAMPLE my-public-key
```

------
#### [ IMDSv1 ]

```
PS C:\> Invoke-RestMethod -uri http://169.254.169.254/latest/meta-data/public-keys/0/openssh-key
ssh-rsa MIICiTCCAfICCQD6m7oRw0uXOjANBgkqhkiG9w0BAQUFADCBiDELMAkGA1UEBhMC
VVMxCzAJBgNVBAgTAldBMRAwDgYDVQQHEwdTZWF0dGxlMQ8wDQYDVQQKEwZBbWF6
b24xFDASBgNVBAsTC0lBTSBDb25zb2xlMRIwEAYDVQQDEwlUZXN0Q2lsYWMxHzAd
BgkqhkiG9w0BCQEWEG5vb25lQGFtYXpvbi5jb20wHhcNMTEwNDI1MjA0NTIxWhcN
MTIwNDI0MjA0NTIxWjCBiDELMAkGA1UEBhMCVVMxCzAJBgNVBAgTAldBMRAwDgYD
VQQHEwdTZWF0dGxlMQ8wDQYDVQQKEwZBbWF6b24xFDASBgNVBAsTC0lBTSBDb25z
b2xlMRIwEAYDVQQDEwlUZXN0Q2lsYWMxHzAdBgkqhkiG9w0BCQEWEG5vb25lQGFt
YXpvbi5jb20wgZ8wDQYJKoZIhvcNAQEBBQADgY0AMIGJAoGBAMaK0dn+a4GmWIWJ
21uUSfwfEvySWtC2XADZ4nB+BLYgVIk60CpiwsZ3G93vUEIO3IyNoH/f0wYK8m9T
rDHudUZg3qX4waLG5M43q7Wgc/MbQITxOUSQv7c7ugFFDzQGBzZswY6786m86gpE
Ibb3OhjZnzcvQAaRHhdlQWIMm2nrAgMBAAEwDQYJKoZIhvcNAQEFBQADgYEAtCu4
nUhVVxYUntneD9+h8Mg9q6q+auNKyExzyLwaxlAoo7TJHidbtS4J5iNmZgXL0Fkb
FFBjvSfpJIlJ00zbhNYS5f6GuoEDmFJl0ZxBHjJnyp378OD8uTs7fLvjx79LjSTb
NYiytVbZPQUQ5Yaxu2jXnimvw3rrszlaEXAMPLE my-public-key
```

------

### Get the subnet ID for an instance<a name="instance-metadata-ex-6"></a>

This example gets the subnet ID for an instance\.

------
#### [ IMDSv2 ]

```
PS C:\> [string]$token = Invoke-RestMethod -Headers @{"X-aws-ec2-metadata-token-ttl-seconds" = "21600"} -Method PUT -Uri http://169.254.169.254/latest/api/token
```

```
PS C:\> Invoke-RestMethod -Headers @{"X-aws-ec2-metadata-token" = $token} -Method GET -Uri http://169.254.169.254/latest/meta-data/network/interfaces/macs/02:29:96:8f:6a:2d/subnet-id
subnet-be9b61d7
```

------
#### [ IMDSv1 ]

```
PS C:\> Invoke-RestMethod -uri http://169.254.169.254/latest/meta-data/network/interfaces/macs/02:29:96:8f:6a:2d/subnet-id
subnet-be9b61d7
```

------

### Get the instance tags for an instance<a name="instance-metadata-ex-7"></a>

In the following examples, the sample instance has [tags on instance metadata enabled](Using_Tags.md#allow-access-to-tags-in-IMDS) and the instance tags `Name=MyInstance` and `Environment=Dev`\.

This example gets all the instance tag keys for an instance\.

------
#### [ IMDSv2 ]

```
PS C:\> $token = Invoke-RestMethod -Headers @{"X-aws-ec2-metadata-token-ttl-seconds" = "21600"} -Method PUT -Uri http://169.254.169.254/latest/api/token
```

```
PS C:\> Invoke-RestMethod -Headers @{"X-aws-ec2-metadata-token" = $token} -Method GET -Uri http://169.254.169.254/latest/meta-data/tags/instance
Name
Environment
```

------
#### [ IMDSv1 ]

```
PS C:\> Invoke-RestMethod -uri http://169.254.169.254/latest/meta-data/tags/instance
Name
Environment
```

------

The following example gets the value of the `Name` key that was obtained in the preceding example\. The IMDSv2 request uses the stored token that was created in the preceding example command, assuming it has not expired\.

------
#### [ IMDSv2 ]

```
PS C:\> Invoke-RestMethod -Headers @{"X-aws-ec2-metadata-token" = $token} -Method GET -Uri http://169.254.169.254/latest/meta-data/tags/instance/Name
MyInstance
```

------
#### [ IMDSv1 ]

```
PS C:\> Invoke-RestMethod -uri http://169.254.169.254/latest/meta-data/tags/instance/Name
MyInstance
```

------

## Query throttling<a name="instancedata-throttling"></a>

We throttle queries to the instance metadata service on a per\-instance basis, and we place limits on the number of simultaneous connections from an instance to the instance metadata service\. 

If you're using the instance metadata service to retrieve AWS security credentials, avoid querying for credentials during every transaction or concurrently from a high number of threads or processes, as this might lead to throttling\. Instead, we recommend that you cache the credentials until they start approaching their expiry time\. For more information about IAM role and security credentials associated with the role, see [Retrieve security credentials from instance metadata](iam-roles-for-amazon-ec2.md#instance-metadata-security-credentials)\.

If you are throttled while accessing the instance metadata service, retry your query with an exponential backoff strategy\.

## Limit instance metadata service access<a name="instance-metadata-limiting-access"></a>

You can consider using local firewall rules to disable access from some or all processes to the instance metadata service\.

**Note**  
For [Instances built on the Nitro System](instance-types.md#ec2-nitro-instances), IMDS can be reached from your own network when a network appliance within your VPC, such as a virtual router, forwards packets to the IMDS address, and the default [source/destination check](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_NAT_Instance.html#EIP_Disable_SrcDestCheck) on the instance is disabled\. To prevent a source from outside your VPC reaching IMDS, we recommend that you modify the configuration of the network appliance to drop packets with the destination IPv4 address of IMDS 169\.254\.169\.254 and, if you enabled the IPv6 endpoint, the IPv6 address of IMDS fd00:ec2::254\.

**Using Windows firewall to limit access**

The following PowerShell example uses the built\-in Windows firewall to prevent the Internet Information Server webserver \(based on its default installation user ID of `NT AUTHORITY\IUSR`\) from accessing 169\.254\.169\.254\. It uses a *deny rule* to reject all instance metadata requests \(whether IMDSv1 or IMDSv2\) from any process running as that user\.

```
PS C:\> $blockPrincipal = New-Object -TypeName System.Security.Principal.NTAccount ("NT AUTHORITY\IUSR")
PS C:\> $BlockPrincipalSID = $blockPrincipal.Translate([System.Security.Principal.SecurityIdentifier]).Value
PS C:\> $BlockPrincipalSDDL = "D:(A;;CC;;;$BlockPrincipalSID)"
PS C:\> New-NetFirewallRule -DisplayName "Block metadata service from IIS" -Action block -Direction out `
-Protocol TCP -RemoteAddress 169.254.169.254 -LocalUser $BlockPrincipalSDDL
```

Or, you can consider only allowing access to particular users or groups, by using *allow rules*\. Allow rules might be easier to manage from a security perspective, because they require you to make a decision about what software needs access to instance metadata\. If you use *allow rules*, it's less likely you will accidentally allow software to access the metadata service \(that you did not intend to have access\) if you later change the software or configuration on an instance\. You can also combine group usage with allow rules, so that you can add and remove users from a permitted group without needing to change the firewall rule\.

The following example prevents access to instance metadata by all processes running as an OS group specified in the variable `blockPrincipal` \(in this example, the Windows group `Everyone`\), except for processes specified in `exceptionPrincipal` \(in this example, a group called `trustworthy-users`\)\. You must specify both deny and allow principals because Windows Firewall, unlike the `! --uid-owner trustworthy-user` rule in Linux iptables, does not provide a shortcut mechanism to allow only a particular principal \(user or group\) by denying all the others\.

```
PS C:\> $blockPrincipal = New-Object -TypeName System.Security.Principal.NTAccount ("Everyone")
PS C:\> $BlockPrincipalSID = $blockPrincipal.Translate([System.Security.Principal.SecurityIdentifier]).Value
PS C:\> $exceptionPrincipal = New-Object -TypeName System.Security.Principal.NTAccount ("trustworthy-users")
PS C:\> $ExceptionPrincipalSID = $exceptionPrincipal.Translate([System.Security.Principal.SecurityIdentifier]).Value
PS C:\> $PrincipalSDDL = "O:LSD:(D;;CC;;;$ExceptionPrincipalSID)(A;;CC;;;$BlockPrincipalSID)"
PS C:\> New-NetFirewallRule -DisplayName "Block metadata service for $($blockPrincipal.Value), exception: $($exceptionPrincipal.Value)" -Action block -Direction out `
-Protocol TCP -RemoteAddress 169.254.169.254 -LocalUser $PrincipalSDDL
```

**Note**  
To use local firewall rules, you need to adapt the preceding example commands to suit your needs\. 

**Using netsh rules to limit access**

You can consider blocking all software using `netsh` rules, but those are much less flexible\.

```
C:\> netsh advfirewall firewall add rule name="Block metadata service altogether" dir=out protocol=TCP remoteip=169.254.169.254 action=block
```

**Note**  
To use local firewall rules, you need to adapt the preceding example commands to suit your needs\. 
`netsh` rules must be set from an elevated command prompt, and can’t be set to deny or allow particular principals\.