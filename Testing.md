# VM Network Requirements Testing Guide

This document outlines the requirements for the VM networking project and provides specific commands to test and verify each requirement.

## 1. VM Setup Requirements

### 1.1 Four VMs with descriptive names
**Test**: Verify VM names in your hypervisor (VMware Workstation/Fusion) management interface.
```
Expected names:
- app-server
- web-server-1
- web-server-2
- load-balancer
```

### 1.2 VMs have appropriate resources allocated
**Test**: Check resources in hypervisor or from within the VM:
```bash
# CPU and Memory check
lscpu
free -h

# Disk space check
df -h
```
**Expected**: Resources should match the specified allocation:
```
| VM Name        |  CPU  | RAM (GB)  | Disk Space (GB) |
| -------------- | ----- | --------- | --------------- |
| app-server     |   1   |     2     |        8        |
| web-server-1   |   1   |     1     |        7        |
| web-server-2   |   1   |     1     |        7        |
| load-balancer  |   2   |     1     |        7        |
```

## 2. Hostname Configuration

### 2.1 Hostnames are correctly set and resolvable
**Test**: Run the following commands on each VM:
```bash
# Display current hostname
hostname

# Ping other VMs by hostname
ping -c 4 app-server
ping -c 4 web-server-1
ping -c 4 web-server-2
ping -c 4 load-balancer
```
**Expected**: Hostname displays correctly, and pings should return successful responses with 0% packet loss.

## 3. Network Configuration

### 3.1 Static IP addresses are assigned to each VM
**Test**: Check IP configuration and verify persistence after reboot:
```bash
# Check current IP configuration
ip a

# Reboot VM
sudo reboot

# After reboot, check IP configuration again
ip a
```
**Expected**: IPs should be consistent before and after reboot, matching your assigned static IPs (e.g., 192.168.189.101-104).

### 3.2 All VMs can communicate with each other
**Test**: Ping between VMs using IP addresses:
```bash
# From each VM, ping all other VMs
ping -c 4 192.168.189.101  # app-server
ping -c 4 192.168.189.102  # web-server-1
ping -c 4 192.168.189.103  # web-server-2
ping -c 4 192.168.189.104  # load-balancer
```
**Expected**: All pings should be successful with 0% packet loss.

### 3.3 Only Load Balancer VM is accessible via browser
**Test**: Try accessing each VM via browser using their IPs:
```
http://192.168.189.101  # app-server
http://192.168.189.102  # web-server-1
http://192.168.189.103  # web-server-2
http://192.168.189.104  # load-balancer
```
**Expected**: Only the load-balancer (192.168.189.104) should be accessible via browser.

**Explanation**: This is achieved by UFW firewall rules that only allow HTTP/HTTPS traffic to the load-balancer and only allow the load-balancer to access web servers on ports 80/443.

## 4. System Updates and Security

### 4.1 VMs are up-to-date with latest security patches
**Test**: Check for available updates:
```bash
sudo apt update && sudo apt list --upgradable
```
**Expected**: Either no upgradable packages or only packages that are intentionally deferred.

## 5. User Management

### 5.1 A user named devops is created on each VM
**Test**: Verify the existence of the devops user:
```bash
grep devops /etc/passwd
```
**Expected**: Should return a line showing the devops user account.

### 5.2 Password login is disabled
**Test**: Try SSH login using the devops user:
```bash
# From your local machine
ssh devops@<VM_IP>
```
**Expected**: Should connect without password prompt (using SSH key authentication).

### 5.3 The devops user is added to the sudo group
**Test**: Check group membership:
```bash
groups devops
```
**Expected**: Output should include `sudo` in the group list, like: `devops : devops sudo`

### 5.4 The devops sudo commands are password protected
**Test**: Execute a sudo command as devops:
```bash
sudo visudo
```
**Expected**: Should prompt for password before executing.

### 5.5 Root login is disabled for added security
**Test**: Try to connect as root:
```bash
ssh root@<VM_IP>
```
**Expected**: Connection should be refused.

### 5.6 Only user devops can login into all VMs
**Test**: Try to connect as another user:
```bash
ssh otheruser@<VM_IP>
```
**Expected**: Connection should be refused.

## 6. Network Security

### 6.1 Unused network interfaces are disabled
**Test**: Check active network interfaces:
```bash
ip link show
```
**Expected**: Should only show necessary interfaces (loopback and one network interface, e.g., ens33). Explain the purpose of each active interface.

### 6.2 UFW (Uncomplicated Firewall) is activated and configured
**Test**: Check firewall status:
```bash
sudo ufw status verbose
```
**Expected**: UFW should be active with specific rules for each server:
- All servers: SSH and WireGuard (51820/udp) allowed
- App server: NetData port (19999/tcp) allowed
- Web servers: Only allow connections from load balancer IP to ports 80,443
- Load balancer: HTTP/HTTPS ports (80,443) open

### 6.3 Secure umask is set for all users
**Test**: Check umask setting and demonstrate its effect:
```bash
# Check current umask value
umask

# Create a test file and check permissions
touch test-file
ls -l test-file
```
**Expected**: umask should be 027, and test file should have permissions like "-rw-r-----" (640).

### 6.4 Automatic security updates are enabled and configured
**Test**: Check auto-update configuration:
```bash
cat /etc/apt/apt.conf.d/20auto-upgrades
```
**Expected**: Should show:
```
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
```

### 6.5 SSH protocol is set to version 2 only
**Test**: Check SSH protocol configuration:
```bash
grep Protocol /etc/ssh/sshd_config
```
**Expected**: Should show "Protocol 2" or have it uncommented.

## 7. Extra Features

### 7.1 Intrusion Prevention Software (Fail2Ban)
**Test**: Check Fail2Ban status:
```bash
sudo fail2ban-client status sshd
```
**Expected**: Fail2Ban should be running with the SSH jail active. Show configuration settings (maxretry, findtime, bantime).

### 7.2 VPN Configuration (WireGuard)
**Test**: Check VPN connection status:
```bash
sudo wg show
```
**Expected**: Should show active WireGuard interface and peers.

### 7.3 Monitoring Setup (NetData)
**Test**: Access the monitoring dashboard:
```
http://192.168.189.101:19999
```
**Expected**: NetData dashboard should load showing monitoring information for all connected nodes.

## 8. Final Demonstration

For a complete assessment, demonstrate:
1. Accessing a web page through the load balancer
2. Showing that the load is distributed between web-server-1 and web-server-2
3. Secure SSH access using the devops user
4. System monitoring with NetData showing all VMs
5. Explain the network security architecture and firewall rules
