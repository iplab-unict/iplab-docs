# Migration to New Server
Automatic migration isn't possible due to past malware infection—files must be manually reviewed for safety. Prefer static sites (Hugo/Jekyll) over PHP for better security. 

## Step 1: Activate Tenant
Request a new tenant (for group/project) and 1+ users from [admin](mailto:antonino.furnari@unict.it). This provisions your isolated Docker space. 

## Step 2: Request Old Files Access
Ask [admin](mailto:antonino.furnari@unict.it) for read access to specific directories/files on the legacy machine.

## Step 3: Copy via SFTP
Connect to new tenant:
- SFTP: `192.168.90.55:2022` (VPN/VLAN 90 needed).
- Download your files from old access, upload to new tenant.

**Critical: Manually scan all PHP files for malware before upload**—delete suspicious code. 

For big files/directories, request direct copy from admin (faster/safer).
