---
id: 758
title: Remove Unknown users from Access Control Lists using PowerShell
date: 2021-03-21T17:34:57+00:00
author: tom
layout: post
guid: https://tomaustin.xyz/?p=758
permalink: /2021/03/21/remove-unknown-users-from-access-control-lists-using-powershell/
image: /wp-content/uploads/2021/03/uk2.png
categories:
  - PowerShell
tags:
  - access control list
  - powershell
---
I recently had a scenario where I had a certificate with hundreds of unknown users added to it&#8217;s [access control list](https://docs.microsoft.com/en-us/windows/win32/secauthz/access-control-lists) which then needed removing, how had that happened and how did you resolve it? Good question! Let&#8217;s first quickly discuss how this had happened so you can avoid falling into the same issue. If you run IIS and host anything slightly complex you may need to grant an app pool permission to a certificate on the IIS server (a good example is if your hosting [Identity Server](https://identityserver4.readthedocs.io/en/latest/)), if you then remove that App Pool the permissions on the folder are left how they are and instead of your App Pool name showing within the ACL of the certificate you&#8217;ll just see a GUID (a SID). Eventually you&#8217;ll want to clean this up (or ideally fix the issue leaving the permissions behind when the App Pool&#8217;s are removed) and depending on how many entries there are in the ACL it could take some time. This is where PowerShell can save you a lot of time!

I modified this script from the [following locatio](https://razor3dg3.wordpress.com/2011/10/30/powershell-remove-unknown-user-permission/)n to cater for my needs a bit more but if you are wanting to modify file or folder ACL permissions it should be fairly straight forward to tweak it. Update cert to be the thumbprint of your certificate and that should be all you need to do to remove all the unknown user permissions.

<pre class="wp-block-code"><code>$cert = '66bdf623569d167001f546456c6f0867a'

$certificate = Get-ChildItem "Cert:\LocalMachine\My" | Where-Object thumbprint -eq $using:cert
$rsaCert = &#91;System.Security.Cryptography.X509Certificates.RSACertificateExtensions]::GetRSAPrivateKey($certificate)
$fileName = $rsaCert.key.UniqueName
$certificatePath = "$env:ALLUSERSPROFILE\Microsoft\Crypto\RSA\MachineKeys\$fileName"
$acl = Get-Acl $certificatePath
$acl.Access | Where-Object {
    $_.IdentityReference -like "*S-1*" -and $_.isinherited -like $false } | ForEach-Object {
    write-host $_.IdentityReference
    $acl.purgeaccessrules($_.IdentityReference);
    Set-Acl -AclObject $acl -path $certificatePath;
}</code></pre>

If you have any questions or comments please feel free to leave a comment below or contact me on [Twitter](https://twitter.com/tomaustin700). Thanks.