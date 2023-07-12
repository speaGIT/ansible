Note: 

For demonstration purposes only. We will be performing a number of very bad IT security practices in order to make this demo run smoothly.

In a production environment, your target Windows Server is probably in a domain with a Windows Certificate Authority and Microsoft's Local Admin Password Solution (LAPS) so you'd set WinRM to operate only on HTTPS and use Certificate Authentication or lookup server specific local admin passwords from LAPS. You'd also set the Windows Server Firewall to only allow WinRM on HTTPS and specify specific TrustedHosts for Powershell remoting.

We are also using a work-around for creating a new external Hyper-V switch which causes the network to drop and thereby ends the WinRM session. 

Requirements:

-A Windows Server OS fresh from its out-of-box experience in a WORKGROUP (this was tested on Windows Server 2022)
-Network connectivity between the Ansible Server and the Windows Server

-Clone this GIT repo to your Ansible Server
    Run: git clone https://github.com/speaGIT/ansible

Ansible Server Setup:
-Ansible Server able to communicate over WinRM
    Run: python3 -m pip install --user --ignore-installed pywinrm
-Install required Ansible collections (in this case, Chocolatey)
    Run: ansible-galaxy collection install -r requirements.yml

Windows Server Setup:
-Windows Server target has a local administrator account named "ansible" with a password of "N0t4prodU$e"
    Run in Powershell as Admin: 
        New-LocalUser -Name "ansible" -Password (ConvertTo-SecureString 'N0t4prodU$e' -AsPlainText -Force) -PasswordNeverExpires
        Add-LocalGroupMember -Group "Administrators" -Member "ansible"
-Windows Server target has WinRM setup, its local firewall disabled, WinRM set to allow with Basic Authentication and trust any host
    Run in Powershell as Admin: 
        winrm quickconfig -force -q
        Set-NetFirewallProfile -Enabled False
        Set-Item -Path "WSMan:\localhost\Service\Auth\Basic" -Value $True
        Set-Item WSMan:\localhost\Client\TrustedHosts -Value "*" -Force
        Restart-Service -Force -Name WinRM

Playbook Use:
-Adjust "ansible_host=172.26.198.245" in the inventory.txt file to target your Windows Server's IP
-Run the playbook: ansible-playbook hyperv-server-playbook.yaml -i inventory.txt
