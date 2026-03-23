
# kerberos-vscode

A CLI tool for macOS to acquire a Kerberos Ticket Granting Ticket (TGT) into an isolated cache and launch Visual Studio Code for Integrated/Windows Authentication (e.g., SQL Server, Active Directory resources).

---

## Overview

This tool was specifically designed so users can access SQL Servers via the SQL Server (mssql) VS Code Extension.
It may also work for other workflows that require Active Directory authentication within VS Code.

Why is this needed?
- VS Code on macOS cannot reliably use the default Kerberos API cache for Integrated/Windows Authentication.
- Microsoft Entra Platform SSO issues a cloud Kerberos ticket (KERBEROS.MICROSOFTONLINE.COM) that does NOT work with classic SQL Server Windows Authentication.
- This helper:
  - Forces a file-based Kerberos cache
  - Obtains an on-prem AD Kerberos TGT (valid for ~10 hours)
  - Launches VS Code in the correct environment
- Avoids manual kinit usage and reduces user error.

Best Practice:
A better long-term solution is to enable Kerberos SSO to on-premises Active Directory and Microsoft Entra ID Kerberos resources using Platform SSO.
If the account used to authenticate to SQL Server is different from the account used to sign in to the device, this tool is the best fit for that scenario.

---

## How to Use

1. Download the project

Clone or download the repository:
  ```sh
  git clone https://github.com/yourorg/kerberos-vscode.git
  or download and unzip the repository
  ```
2. Customize for your environment

Edit the kerberos-vscode script and update:
- Kerberos realm (for example: EXAMPLE.COM)
- KDC list (for example: kdc1.example.com, kdc2.example.com)

Optionally update the man page (kerberos-vscode.1).

3. Build the package

Run:
  ```sh
  ./build
  ```
This produces a PKG file in the dist directory.

4. Deploy using Intune (or other MDM)

Upload the PKG as a macOS Line-of-Business app and assign it as you would any other package.

---

## End-User Workflow

1. Connect to VPN if required.
2. Open Terminal.
3. Run:
  ```sh
  kerberos-vscode
  ```
4. Enter your Kerberos username when prompted.
5. Enter your password.
6. Visual Studio Code opens automatically.

---

## Connecting to SQL Server in VS Code

Use the SQL Server (mssql) extension:

- Server name: Fully Qualified Domain Name (FQDN)
- Authentication: Windows Authentication
- Encrypt: Enabled (default)
- Trust server certificate: Approve if prompted

If Kerberos succeeds, no username or password prompt will appear.

---

## Verification (Optional)

In VS Code -> Terminal:
  ```sh
  klist
  ```
Expected output:

  Credentials cache: FILE:/tmp/krb5cc_vscode
  Principal: user@EXAMPLE.COM
  krbtgt/EXAMPLE.COM@EXAMPLE.COM

---

## Troubleshooting

Cannot authenticate using Kerberos:
- Ensure VS Code was launched using kerberos-vscode
- Verify klist shows the on-prem AD realm, not KERBEROS.MICROSOFTONLINE.COM
- Confirm the SQL Server SPN exists and matches the FQDN

SQL Login works but Windows Authentication does not:
- Kerberos is failing before SQL Server
- Almost always caused by using a cloud Kerberos ticket instead of on-prem

---

## Security Notes

- Kerberos tickets are time-limited (~10 hours)
- No passwords are stored
- File-based cache exists only for the session
- Aligns with Microsoft-supported Kerberos flows for macOS

---

## References

- https://marketplace.visualstudio.com/items?itemName=ms-mssql.mssql
- https://learn.microsoft.com/en-us/previous-versions/azure-data-studio/enable-kerberos?tabs=mac
- https://learn.microsoft.com/entra/identity/devices/device-join-macos-platform-single-sign-on-kerberos-configuration
