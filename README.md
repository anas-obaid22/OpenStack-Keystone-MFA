# OpenStack-Keystone-MFA
Implementation and validation of Multi-Factor Authentication (MFA) for OpenStack Keystone

## 1. Project Overview
This project demonstrates the implementation and validation of Multi-Factor Authentication (MFA) within an OpenStack environment. The lab focuses on securing high-privilege administrative accounts using TOTP (Time-based One-Time Passwords).

## 2. Lab Environment
To ensure a modern and reproducible setup, the following environment was used:
* **Operating System:** Ubuntu 24.04 LTS (Noble Numbat)
* **Deployment Tool:** DevStack (Latest Master branch)
* **Identity Service:** OpenStack Keystone



## 3. Implementation & Configuration
### Environment Preparation
DevStack was deployed with a minimal configuration focusing on Identity and Compute services. Once the stack was functional, the Keystone configuration was modified.

### Keystone Setup
The `/etc/keystone/keystone.conf` file was updated to enable the `totp` authentication method:
```ini
[auth]
methods = password,token,totp
