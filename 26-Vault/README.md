# CDX Secrets Vault

CDX Vault is a simple, small, portable, cross-platform (Java Swing) password manager application with strong encryption. It allows you to store user names, passwords, URLs and generic notes in an encrypted file protected by one master password.

Features:

- **Strong Encryption** - AES-256-CBC algorithm
- **Cross-Platform** - Java Swing application supporting Windows, Unix, and Android
- **Portable** - Single JAR file (about 200k) which can be carried on a USB stick
- **Password Generator** - Built-in random password generator
- **JSON Import/Export** - Allows easier entry management between vaults
- **Blind Password Copy** - Copy passwords to clipboard without showing password on UI

#### Usage

CDX Vault is included in the CDX installation.  If you placed the `bin` directory of CDX on your execution path, you can just call:

```
csv [path to your vault file]
```

The above will open the CDX Vault graphical user interface and you will be able to manage your secrets simply. Your system will of course need a windowing system to use the tool.

Alternately, you can just execute the JAR directly. Most platforms have a mechanism to execute `.jar` files (e.g. double click the `CDXVault-X.X.X.jar`). You can also run the application from the command line by typing (the vault file is optional):

```
java -jar CDXVault-X.X.X.jar [vault_file]
```

For Windows use, some place the JAR in its own directory and add a batch file (e.g. `csv.bat`) to that directory with the following contents:

```
@if "%DEBUG%" == "" @echo off
start /B javaw -jar CDXVault-X.X.X.jar %*
```

Create a shortcut to that file and you can run the CDX Vault from anywhere on you workstation. Just make sure your startup directory points to the directory where the JAR is located. Some will go so far as making Windows shortcuts to the above batch file with the name of their vaults as arguments in the shortcut "Target". When CDX Vault loads, it prompts the user for the master password for the vault then opens and decrypts the vault. This give users a simple way to access encrypted secrets on their file systems.

#### Brute Force Attacks

If you lose your master password, it is next to impossible to brute force crack your encryption key. The SHA-256 hash of the master password undergoes 1000 iterations as recommended by RSA PKCS5. While this slightly slows down user entry into the vault, it greatly slows down a brute force attack. Chances are your passwords will cycle/expire well within the time it would take to access your vault.

**Don't lose your master password!**

## Managing Vault Files

Open the `demo.vault` by calling `csv` and opening the vault in this directory. 

The password for the `demo.vault` is "changeme".

