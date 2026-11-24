Finals Project: Enterprise Server Setup, Hardening & Incident Audit

Scenario: NexusTech Solutions is launching a new internal web portal and upgrading its branch office network security. Starting from a completely blank VirtualBox virtual machine, your team (2–3 students) will install Ubuntu Linux, configure user accounts and directory permissions, set up and verify an Nginx web server, investigate live packet captures and system logs, harden a simulated branch network in Cisco Packet Tracer, and produce a security awareness campaign for staff.

Phase 1: VM Provisioning & Linux User/System Setup
Build a clean Ubuntu Linux environment from scratch and establish basic administrative access controls.
Step-by-Step Instructions:
Create & Install the VM:
Open Oracle VM VirtualBox, click New, name the VM NexusTech-Server, select Linux / Ubuntu (64-bit), and allocate at least 2048 MB (2 GB) RAM and a 25 GB dynamic virtual hard disk.
Mount your downloaded Ubuntu .iso file under Settings > Storage, start the VM, and complete the standard OS installation using a primary admin account (sysadmin).
Update & Verify Networking:
Open the Terminal (Ctrl + Alt + T) and update the package repositories:
Bash
sudo apt update && sudo apt upgrade -y
Create Users, Groups & Sudo Access:
Create a dedicated department group and two staff accounts (one standard user, one junior admin):
Bash
sudo groupadd 
sudo adduser
Assign both users to the it_staff group and grant administrative (sudo) privileges only to junior_admin: 
Create a Restricted Shared Directory:
Create a folder /srv/nexustech_logs owned by root:it_staff with permissions set so only group members can read/write (770)


Phase 2: Web Server Setup, Packet Forensics & Log Auditing
Deploy the Nginx web server from scratch, install Wireshark, and investigate two simulated security incidents: cleartext credential interception and local DNS hijacking.
Scenario Hook Employees at NexusTech Solutions reported two anomalies after the new internal portal went live:
Their browser shows a "Not Secure" warning when logging in, raising concerns that credentials could be sniffed on the local network.
One workstation keeps getting redirected to an unknown IP address when visiting portal.nexustech.local.
Step-by-Step Instructions:
Part A: Set Up and Verify Nginx
Install Nginx & Wireshark:
Update packages and install both nginx and wireshark (select Yes when prompted to allow non-superusers to capture packets):
Bash
sudo apt install nginx wireshark -y
Log out and back in so your wireshark group membership takes effect.

Start, Enable & Verify the Web Service:
Ensure Nginx starts automatically at boot and check its active status:
sudo systemctl enable nginx
sudo systemctl start nginx
sudo systemctl status nginx
Open Firefox inside the VM and navigate to http://localhost (or [http://127.0.0.1](http://127.0.0.1)) to confirm the "Welcome to nginx!" default landing page loads.
Part B: Deploy the Portal & Catch Cleartext Credentials (HTTP Sniffing)
Create the Mock Login Page on Nginx:
Replace or add a simple login page in /var/www/html/login.html that submits credentials over unencrypted HTTP:
<!DOCTYPE html>
<html>
<body>
  <h2>NexusTech Employee Login</h2>
  <form action="/login.html" method="GET">
    Username: <input type="text" name="username"><br><br>
    Password: <input type="password" name="password"><br><br>
    <input type="submit" value="Login">
  </form>
</body>
</html>

Capture the Login Attempt in Wireshark:
Launch wireshark, start capturing packets on the Loopback (lo) interface, and apply the display filter: http
In Firefox, open http://localhost/login.html and log in using username: analyst1 and password: NexusPass2026!.
Inspect the HTTP Stream:
Stop the capture, locate the GET /login.html?username=analyst1... packet, right-click it, and select Follow > HTTP Stream.
Document how the username and password appear in plain text despite being masked in the browser.
