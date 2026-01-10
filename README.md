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
### Step 1: System Preparation & DevStack Install
First, the Ubuntu 24.04 environment was updated, and a `stack` user was created.
```bash
sudo apt update && sudo apt upgrade -y
sudo useradd -s /bin/bash -d /opt/stack -m stack
```

### Step 2:Deploy DevStack: Clone the repository and run stack.sh. Ensure your local.conf includes the Keystone service. 
```bash
git clone [https://opendev.org/openstack/devstack](https://opendev.org/openstack/devstack)
cd devstack
./stack.sh
```
