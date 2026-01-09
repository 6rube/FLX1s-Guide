# How to run an SSH Server on the FLX1

Be careful when using SSH, always verify the host key fingerprint and avoid trusting unknown hosts blindly, because enabling SSH exposes a remote entry point that attackers can brute‑force or abuse if it is misconfigured.

## Installation and config

1. Install needed packages

For SSH over Internet:
```bash
sudo apt install openssh-server
```
For SSH over USB:
```bash
sudo apt install usb-tethering 
```
2. Goto Settings App -> System -> Secure Shell
3. Enable Secure Shell
4. Connect from client with 
```bash
ssh furios@FuriPhoneFLX1s
```
