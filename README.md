# Configuring FIDO Authentication on Oracle Linux

## Objective
Enable a user to log in to the GNOME Desktop using a security key without a password.

## Tested on
Oracle Linux 9.4

## Reference
YubiKey Guide https://support.yubico.com/hc/en-us/articles/360016649099-Ubuntu-Linux-Login-Guide-U2F

## Steps

### 1. Enable EPEL Repository
Command: `sudo dnf install oracle-epel-release-el9-x.x.x`
Note: Replace `x.x.x` with the version number.

### 2. Install YubiKey Manager
Command: `sudo dnf install yubikey-manager`

### 3. Install PAM-U2F
Command: `sudo dnf install pam-u2f`

### 4. Install PAMU2FCFG
Command: `sudo dnf install pamu2fcfg`

### 5. Associate the U2F Key with User Account
- Open Terminal.
- Insert your U2F Key.
- Run: `mkdir -p ~/.config/Yubico`
- Run: `pamu2fcfg > ~/.config/Yubico/u2f_keys`

### 6. Create a configuration file to use the FIDO PAM module
- Create file: `sudo nano /etc/pam.d/fido2-auth`
- Add this line: `auth    required    pam_u2f.so cue pinverification=1 userverification=0`
- Save and close this file

### 7. Test configuration with the sudo command only
- Open Terminal.
- Open file: `sudo nano /etc/pam.d/sudo`
- Comment out the line: `auth include system-auth`
- Add new line: `auth substack fido2-auth`
- Press Ctrl+O and then Enter to save the file. Do not close the file, otherwise you will not be able to revert the changes.
- Remove your YubiKey from the computer.
- Open a new Terminal.
- In the new Terminal, run: `sudo echo test`. The authentication should fail as the U2F Key is not plugged in. If the authentication succeeds without the U2F Key, that indicates the U2F PAM module was not installed or there is a typo in the changes you made to `/etc/pam.d/sudo`.
- Insert your device.
- Open a new Terminal and run `sudo echo test` again. When prompted, enter your YubiKey PIN and press Enter. Then, touch the metal contact on your U2F Key when it begins flashing.
- If accepted this time, you have configured the system correctly and can continue on to the next section for requiring the U2F Key to login. Note: if you do not want to require the U2F Key to run the sudo command, remove the line you added to the `/etc/pam.d/sudo` file.

### 8. Configure the System to Require the YubiKey for Login
- Open File: `nano /etc/pam.d/gdm-password`
- In the opened file, add the following line: `auth substack fido2-auth`
- Comment out the following line by adding a `#` at the beginning: `auth substack password-auth`
- Save and close `gdm-password` file
- Open File: `nano /etc/pam.d/polkit-1`
- In the opened file, add the following line: `auth substack fido2-auth`
- Comment out the following line by adding a `#` at the beginning: `auth include system-auth`
- Save and cose `polkit-1` file


