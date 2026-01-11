# OpenStack-Keystone-MFA
Implementation and validation of Multi-Factor Authentication (MFA) for OpenStack Keystone

## 1. Project Overview
This project demonstrates the implementation and validation of Multi-Factor Authentication (MFA) within an OpenStack environment. The lab focuses on securing high-privilege administrative accounts using TOTP (Time-based One-Time Passwords).

## 2. Lab Environment
To ensure a modern and reproducible setup, the following environment was used:
* **Operating System:** Ubuntu 24.04 LTS 
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
```
#### Create a minimal local.conf (recommended)
```bash
nano local.conf
```
Example configuration (basic single-node):

```ini
[[local|localrc]]
ADMIN_PASSWORD=StrongAdminPass123!
DATABASE_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD

HOST_IP=127.0.0.1
```
#### Run DevStack:
```bash
./stack.sh
```



## 4. Enabling TOTP in Keystone Configuration
By default, Keystone only uses password/token auth. We must manually enable the totp method.
### Step 1: Edit the configuration file:
```bash
sudo nano /etc/keystone/keystone.conf
```

### Step 2:Locate the [auth] section and update the methods:
```bash
[auth]
methods = password,token,totp
```

### Step 3:Restart the Identity Service:
```bash
sudo systemctl restart devstack@keystone
```
This restart approach is also used inside DevStack’s Keystone service scripts

## 5.User Enrollment & Secret Generation
Each user needs a unique Base32 secret key to generate codes on their mobile device (Google Authenticator/Authy).
#### Step 1: Source admin credentials (DevStack)
DevStack provides an OpenRC file:
```bash
cd /opt/stack/devstack
source openrc admin admin
```

#### Step 2: Generate a Base32 Secret Key
```bash
SECRET=$(python3 - <<'PY'
import os, base64
print(base64.b32encode(os.urandom(16)).decode().rstrip("="))
PY
)
echo "$SECRET"
```
#### Step 3: Register the Secret to the User
First, get the user ID (example: admin):
```bash
USER_ID=$(openstack user show admin -f value -c id)
echo "$USER_ID"
```

Then create the TOTP credential:

```bash
openstack credential create --type totp "$USER_ID" "$SECRET"
```
#### Step 4: Add the Secret to an Authenticator App
On your phone:

- Open Google Authenticator / Authy 
- Add a new account
- Choose “Enter a setup key”
- Paste the Base32 secret from SECRET
- You should now see a rotating 6-digit code (TOTP)

## 6.Enforcing the MFA Policy
Now we tell OpenStack that the user cannot log in with just a password; they must provide both password and TOTP.

#### Step 1: Set the MFA Rule (admin user)
```bash
openstack user set --enable-multi-factor-auth --multi-factor-auth-rule password,totp admin
```
This CLI option is documented for the Identity v3 user command.

#### Step 2 (recommended): Create a “high-privilege” demo user 
This helps you demonstrate “admin vs normal user” clearly for the final deliverable.

Create a project + user:
```bash
openstack project create mfa-lab
openstack user create --project mfa-lab --password "UserPass123!" lab-user
openstack role add --project mfa-lab --user lab-user member
```
(Optional) Create a second privileged user:
```bash
openstack user create --password "SecAdminPass123!" sec-admin
openstack role add --project admin --user sec-admin admin
openstack user set --enable-multi-factor-auth --multi-factor-auth-rule password,totp sec-admin
```
## 7. Validation & Testing 
This section demonstrates that MFA is actually enforced and working.

#### Step 1: Confirm password-only authentication fails (admin)
Try to request a token using password only.
```bash
curl -i -s -X POST http://127.0.0.1/identity/v3/auth/tokens \
  -H "Content-Type: application/json" \
  -d '{
    "auth": {
      "identity": {
        "methods": ["password"],
        "password": {
          "user": {
            "name": "admin",
            "domain": {"name": "Default"},
            "password": "YOUR_ADMIN_PASSWORD"
          }
        }
      }
    }
  }'
```
Expected result:

HTTP 401 Unauthorized.
Response contains an Openstack-Auth-Receipt header (this is the “partial auth” receipt).

#### Step 2: Complete authentication using TOTP + auth receipt
Copy the Openstack-Auth-Receipt value from Step 1, then run:
```bash
curl -i -s -X POST http://127.0.0.1/identity/v3/auth/tokens \
  -H "Content-Type: application/json" \
  -H "Openstack-Auth-Receipt: PASTE_RECEIPT_HERE" \
  -d '{
    "auth": {
      "identity": {
        "methods": ["totp"],
        "totp": {
          "user": {
            "name": "admin",
            "domain": {"name": "Default"},
            "passcode": "the code from Google Authenticator app"
          }
        }
      }
    }
  }'
```
Expected result:

HTTP 201 Created
Response header includes X-Subject-Token: ...


