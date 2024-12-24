# Ports & Services
This is about standard ports that are used for all kinds of services runnen on a machine. 
- The ports from 0 to 1023 are well known, these are used by system processes and a process must have superuser privileges to bind. 
- The ports from 1024 to 49151 are the registered ports, these are registered to specific services andd can be used without superuser. 
- The ports from 49152 to 65535 are ports that can be used for private, custom and temporary purposes.

## List

### 20,21 - FTP
File Transfer Protocol
- Unsecure file transfer between machines over network

### 22 - SSH
Secure SHell
- Secure remote shell over network

### 23 - TelNet
Teletype Network
- Communication over network, service debugging, insecure

### 80 - HTTP
HyperText Transfer Protocol

### 389 - LDAP
Lightweight Directory Access Protocol
- Used by Java Naming and Directory Interface API (JNDI)

### 443 - HTTPS
HyperText Transfer Protocol Secure

### 445 - SMB
Server Message Block
- Microsoft file sharing

### 1433 - MSSQL
Microsoft SQL Server database management system server

### 5985 - WinRM
Windows Remote Management Service, Windows Powershell
- Use evil-winrm in terminal

### 6379 - Redis
- In-memory database
- redis-cli command to connect


# Sources
- Basic info, and long port + services list: https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers
- Common experiences with certain ports and services, and how to pwn: https://app.hackthebox.com/
