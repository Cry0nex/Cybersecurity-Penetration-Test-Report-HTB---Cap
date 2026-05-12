# Cybersecurity Penetration Test Report: HTB - Cap
Target System: Linux (Ubuntu)
Assessed by: Cry0nex
Compromise Level: Full System Control (Root)

1. Project Overview & Scope
The objective of this assessment was to identify and exploit vulnerabilities within the "Cap" environment. The system serves as a prime example of how unencrypted legacy protocols, web logic flaws, and improper Linux permission settings create a chain of high-risk vulnerabilities.  

2. Reconnaissance and Service Enumeration
The engagement began with a network-level scan to determine the target's active services. The scan identified three primary entry points: Port 21 (FTP), Port 22 (SSH), and Port 80 (HTTP).  
The presence of FTP on Port 21 is a critical security concern because the protocol lacks encryption, meaning all authentication data and file transfers occur in plaintext. The HTTP service on Port 80 hosts a web-based dashboard used for system monitoring and network analysis.  

3. Web Exploitation: Insecure Direct Object Reference (IDOR)
Upon exploring the web application, a dashboard was discovered that visualizes network traffic captures. The application uses a specific URL structure, such as /data/1, to retrieve these captures.  
A vulnerability test was performed by manually decrementing the ID in the URL to /data/0. The application failed to perform an authorization check, allowing access to a global network capture file that was not intended for public view. This confirms a critical IDOR (Insecure Direct Object Reference) vulnerability, which serves as the primary breach point for the system.  

4. Foothold: Credential Extraction from Network Traffic
The PCAP file (0.pcap) obtained from the IDOR vulnerability contained a historical record of the server's network activity. Because the server utilized the unencrypted FTP protocol, the communication logs revealed a successful login attempt.  
Within this traffic, the username nathan and his associated plaintext password were successfully intercepted. Due to common password reuse practices, these credentials were tested against the SSH service on Port 22, granting a stable remote shell on the target.  
• User Flag: 5b8a34fbf6b3ff4e48a268c4cf82897d  

5. Privilege Escalation: Exploiting Linux Capabilities
With low-privileged access as the user nathan, an audit of the filesystem was conducted to identify pathways to administrative (root) access.  
The audit revealed that the Python 3.8 binary (/usr/bin/python3.8) was configured with the cap_setuid+ep capability. In a hardened environment, only the root user can change a process's effective User ID (UID). However, this specific capability allows the Python interpreter to change its own UID to 0 (root) even when executed by a standard user.  
A custom Python script was executed to leverage this misconfiguration:
python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'  
This command successfully transitioned the session from a standard user to a root shell, providing full control over the entire operating system.  
• Root Flag: 1dc7735efb3dd768be44cdadb099f19c  

6. Remediation and Security Best Practices
To secure the "Cap" environment, the following actions are recommended:
• Secure Communications: Decommission the FTP service and replace it with SFTP to ensure all credentials and data are encrypted during transit.  
• Access Control Logic: Update the web application to validate that the authenticated user has explicit permission to access a requested data ID before serving the file.  
• Capability Hardening: Remove the cap_setuid capability from the Python interpreter. Powerful system binaries should never be granted the ability to escalate privileges for non-administrative users.
