# Ansible Playbook for Installing and Configuring Octopus Tentacle

This experimental Ansible playbook installs the Octopus Tentacle on a Windows machine and configures it to be a listening Tentacle. 

## Tasks

1. **Download Tentacle MSI**:
    - This task downloads the latest Octopus Tentacle MSI installer from the official Octopus website to the `C:\temp` directory.

2. **Install Tentacle silently**:
    - Installs the Tentacle MSI silently to the specified directory (`C:\OctopusTentacle`) without user interaction.

3. **Create Tentacle instance**:
    - Creates a new Tentacle instance with the specified configuration file (`C:\OctopusTentacle\tentacle.config`).

4. **Generate new certificate**:
    - Generates a new certificate for the Tentacle instance if none exists.

5. **Configure Tentacle to listen**:
    - Configures the Tentacle to listen on port 10933 and sets the home and application directories.

6. **Register Tentacle with Octopus Server**:
    - Registers the Tentacle with the Octopus Server using the provided server URL and API key.

7. **Allow Tentacle through firewall**:
    - Adds a firewall rule to allow incoming traffic on port 10933.

## Ansible Playbook

```yaml
---
- name: Install and configure Octopus Tentacle
  hosts: windows
  become: yes
  tasks:
    - name: Download Tentacle MSI
      win_get_url:
        url: https://octopus.com/downloads/latest/WindowsX64/OctopusTentacle.msi
        dest: C:\temp\OctopusTentacle.msi

    - name: Install Tentacle silently
      win_command: msiexec /i C:\temp\OctopusTentacle.msi /quiet INSTALLLOCATION=C:\OctopusTentacle

    - name: Create Tentacle instance
      win_command: "C:\OctopusTentacle\Tentacle.exe create-instance --instance Tentacle --config C:\OctopusTentacle\tentacle.config --console"

    - name: Generate new certificate
      win_command: "C:\OctopusTentacle\Tentacle.exe new-certificate --instance Tentacle --if-blank --console"

    - name: Configure Tentacle to listen
      win_command: "C:\OctopusTentacle\Tentacle.exe configure --instance Tentacle --reset-trust --home C:\Octopus --app C:\Octopus\Applications --port 10933 --console"

    - name: Register Tentacle with Octopus Server
      win_command: "C:\OctopusTentacle\Tentacle.exe register-with --instance Tentacle --server http://YOUR_OCTOPUS_SERVER --apiKey=API-KEY"

    - name: Allow Tentacle through firewall
      win_command: netsh advfirewall firewall add rule name=OctopusDeployTentacle dir=in action=allow protocol=TCP localport=10933
