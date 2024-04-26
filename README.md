# Introduction

This is _a_ fix for a surprise issue that I've encountered after an update of an ESET product.

For information (and the reason for this repo) check: 
https://forum.eset.com/topic/40702-eset-ssl-protection-produces-an-invalid-certificate-chain-for-nodejs-apps/

## The Problem
ESET started replacing some certificates with their own certificates which are unknown to 
node.  There is presently no real answer from support, no option to set an additional 
certificate in node, nor is there one to use a system/specified trust store.

The easy solution is to turn off the certificate check.  _Let's not call that one an option if 
we can._

We could also disable the ESET feature (but let's avoid that one too).

The solution here is to export the certificate from the Windows certificate store and set it as
an additional certificate for NPM.

# Steps

## Step 1. Setup the Directory
The certificate needs to exist somewhere visible to node.  

For the purpose of this document, let's place it in the user directory: `%USERPROFILE%/certs`.

## Step 2. Export the ESET certificate

1. Open the certificate manager (e.g. run `certmgr.msc`)![Screenshot of run dialog ready to open the certificate manager.](/run-certmgr.png)
2. Find the `ESET SSL Filter CA` certificate (or whichever is applicable to your use-case) ![Screenshot of certificate manager with ESET SSL selected.](certmgr-display.png)
3. Right click, `All Tasks` | `Export...`.  This will open a wizard.
4. Within the wizard, choose to export as PKCS7 with all certificates in the path. ![Screenshot of the described option selected](export-file-wizard.png)
5. Export to the new folder created in Step 1

## Step 3. Convert the certificate

1. In `cmd` (or equivalent), navigate to the folder created in Step 1.
2. Run `openssl pkcs7 -print_certs -inform DER -in exported.p7b -outform PEM -out converted.cer` (thanks @nagyszabi, @ferdiusa, and @rstefko).  Here it is assumed the 
certificate exported in Step 2 is named `exported.p7b`.  Update the script to match your
actual filenames.

### Conversion Issues
Your milage may vary with the above but the general concensus seems that the current state here seems to work.

Check this issue for the latest: https://github.com/the-last-byte/ESET-NPM-Breakage-Fix/issues/1

## Step 4. Store converted certificate in environmental variable

Note that the `\m` in the below command saves the variable in a system, rather than user 
context.  See [setx documentation](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/setx).

1. In `cmd` (or equivalent), navigate to the folder created in Step 1.
2. Run `setx NODE_EXTRA_CA_CERTS %USERPROFILE%\certs\converted.cer /m` where `converted.cer` 
is the name of the converted certificate from Step 3.

# Notes
## OpenSSL
If you have Git for Windows, then you should have openssl (even if `cmd` cannot find it).

Check the install directory, e.g. `C:\Program Files\Git\usr\bin\openssl.exe`.  Failing that
you may need to install openssl.


# Conversion PS Script
@rstefko provided the below powershell script in [this comment](https://github.com/the-last-byte/ESET-NPM-Breakage-Fix/issues/1#issuecomment-2069024027).

I haven't tested it - with all things (including this readme), your milage may vary :).

```ps
$cert = Get-ChildItem -Path Cert:\LocalMachine\Root | Where { $_.Subject -like "*CN=ESET SSL Filter CA" } Export-Certificate -Cert $cert -FilePath C:\Temp\ESET-SSL-Filter-CA.cer &certutil -f -encode C:\Temp\ESET-SSL-Filter-CA.cer C:\Temp\ESET-SSL-Filter-CA.pem
```
