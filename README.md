# SecretManagement.KeyChain

macOS KeyChain vault extension for SecretManagement module

## Cmdlets

These cmdlets are exported by this module for use by the user:

### Get-KeyChainConfiguration

Returns the current configuration for the specific KeyChain used by this module. If the `SecretManagement.KeyChain` has not been created, you will be prompted for a new password specifically for this key-chain.

```powershell-output
PS> Get-KeyChainConfiguration

Name                      PasswordTimeout
----                      ---------------
SecretManagement.KeyChain 300
```

### Set-KeyChainConfiguration

Set the timeout for the password.  Default is 300 seconds.  Set to 0 for no-timeout for when the system prompts for the password.

### Unlock-KeyChain

Allows programmatic ability to unlock KeyChain if there is a timeout set for automation.  Password needs to be
provided via `-Password` parameter otherwise you will be prompted at the console.

## Configuration of SecretManagement.KeyChain

The specific keychain created for PowerShell will require you to enter a password twice. If you forget this password, it can be reset with `Set-KeyChainConfiguration`.

```console
PS> Register-SecretVault -Name KeyChain -Module SecretManagement.KeyChain

# new keychain will be created in next step
PS> Get-KeyChainConfiguration

# optional if you want to change the default timeout of 300 seconds
PS> Set-KeyChainConfiguration -PasswordTimeout <int>

PS> Set-Secret -Name justask -Vault KeyChain
cmdlet Set-Secret at command pipeline position 1
Supply values for the following parameters:
SecureStringSecret: ***************

PS> Get-SecretInfo -Vault KeyChain
Name            Type VaultName
----            ---- ---------
justask SecureString KeyChain

PS> Get-Secret -Name justask -Vault KeyChain -AsPlainText
```

## Module Design

This module wraps the `security` command line tool in macOS and leverages specifically the `generic-password` capability stored
in a KeyChain called `SecretManagement.KeyChain`.
Because `generic-password` only accepts a text password, all secret types string, SecureString, hashtable, PSCredential, and byte array
are converted to strings and base64 encoded to avoid any escaping problems as the string content is passed directly to the
`security` command line tool as an argument value.

SecureStrings are converted to plaintext when stored in KeyChain.
PSCredential is converted to JSON where the password is converted to plaintext.
Hashtables are recursively converted to strings and then converted to JSON.
However, PSCredential members contain an additional `__type` member with value "PSCredential" so that they can be
converted back to an actual PSCredential object when retrieving the hashtable.
Byte arrays are converted to a hex string and converted back to a byte array on retrieval.
