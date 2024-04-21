# Introduction

This is _a_ fix for a surprise issue that I've encountered after an update of an ESET product.

For information (and the reason for this repo) check: 
https://forum.eset.com/topic/40702-eset-ssl-protection-produces-an-invalid-certificate-chain-for-nodejs-apps/

## The Problem
ESET started replacing some certificates with their own, certificates which are not known to 
node.  There is no option to set an additional certificate in node, nor is there one to use a
system/specified trust store.

The easy solution is to turn off the certificate check.  _Let's not call that one an option if 
we can._

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
2. Run `pkcs7 -print_certs -in exported.p7b -out converted.cer`.  Here it is assumed the 
certificate exported in Step 2 is named `exported.p7b`.  Update the script to match your
actual filenames.

## Step 4. Store converted certificate in environmental variable

1. In `cmd` (or equivalent), navigate to the folder created in Step 1.
2. Run `setx NODE_EXTRA_CA_CERTS %USERPROFILE%\certs\converted.cer /m` where `converted.cer` 
is the name of the converted certificate from Step 3.

# Notes
## OpenSSL
If you have Git for Windows, then you should have openssl (even if `cmd` cannot find it).

Check the install directory, e.g. `C:\Program Files\Git\usr\bin\openssl.exe`.  Failing that
you may need to install openssl.

