# Generic Electron Auto Update Example
After struggling with this for few days I decided to write this guide, hopefully it will help someone. 
I have to give credit to the guy who wrote made the initial guide. I updated his code to work with electron 5.0.1.
Here is the link to [`Matt Haggard and his electron-updater-example`](https://github.com/iffy/electron-updater-example)
This repo contains the **bare minimum code** to have an auto-updating Electron app using [`electron-updater`](https://github.com/electron-userland/electron-builder/tree/master/packages/electron-updater).
## Getting started
Clone the repo cd into project folder and execute these commands.
```
npm install
npm start
```
If everything went well, you can continue with this guide.
## Code Signing Certificates
* For macOS, you will need a code-signing certificate.
    Install Xcode (from the App Store), then follow [these instructions](https://developer.apple.com/library/content/documentation/IDEs/Conceptual/AppDistributionGuide/MaintainingCertificates/MaintainingCertificates.html#//apple_ref/doc/uid/TP40012582-CH31-SW6) to make sure you have a "Mac Developer" certificate.  If you'd like to export the certificate (for automated building, for instance) [you can](https://developer.apple.com/library/content/documentation/IDEs/Conceptual/AppDistributionGuide/MaintainingCertificates/MaintainingCertificates.html#//apple_ref/doc/uid/TP40012582-CH31-SW7).  You would then follow [these instructions](https://www.electron.build/code-signing).
* On Windows, you can generate self-signed certificate for testing and development purposes. This certificate is required for electron-updater to work properly.
    You can generate this certificate by following steps below:
    Open Windows PowerShell with administrative privileges and execute these commands:
    ```
    $rootCert = New-SelfSignedCertificate -DnsName "martin" -Type CodeSigningCert -CertStoreLocation "C:\projects\nsis-test\certs\"
    [System.Security.SecureString]$rootcertPassword = ConvertTo-SecureString -String "some_password" -Force -AsPlainText
    [String]$rootCertPath = Join-Path -Path 'cert:\CurrentUser\My\' -ChildPath "$($rootcert.Thumbprint)"
    [String]$rootCertPath = Join-Path -Path 'cert:\LocalMachine\My\' -ChildPath "$($rootcert.Thumbprint)"
    Export-PfxCertificate -Cert $rootCertPath -FilePath 'RootCA.pfx' -Password $rootcertPassword
    Export-Certificate -Cert $rootCertPath -FilePath 'RootCA.crt'
    ```
    
    After successful creation of certificate files, make sure that they are in correct location. Open Certificate Manager by clicking on the Start button, type certmgr.msc in the search field and press Enter key.
    
    If your newly created certificate is not in *Trusted Root Certification Authorities* certificate folder, drag and drop it from *Personal*. Otherwise automatic updates will not work.
    
    Next, open package.json in your favorite text editor and modify these two lines.
 
    ```
    "certificateFile": "./certs/some_file.pfx",
    "certificatePassword": "some_password"
    ```
    Note: self-signed certificate can only be used for testing purposes.
    You can get more help on these websites:
    * [`How to: Create Temporary Certificates for Use During Development`](https://docs.microsoft.com/en-us/dotnet/framework/wcf/feature-details/how-to-create-temporary-certificates-for-use-during-development)
    * [`New-SelfSignedCertificate`](https://docs.microsoft.com/en-us/powershell/module/pkiclient/new-selfsignedcertificate?view=win10-ps)
## Packaging And Testing
When you are done with code signing mess, it's time for building and testing. In package.json under scripts.dist key you can set your desired os. See the example below.
```
"scripts": {
    "dist": "build --linux --mac --win"
}
```
After configuring dist key for your os, run the command below.
```
npm run dist
```
### Testing Auto Update
When npm run dist finishes with the building process, open /dist folder and install the app on your system. After successful install, bump the version in package.json and and run the "npm run dist" command once again. 
When done, copy paste the contents of /dist folder to /wwwroot folder and run "npm run serve". Open the previous version that you installed on your system and voilà :)!