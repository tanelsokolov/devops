# VM Setup Verification Checklist

## Basic VM Setup Verification

### 1. VM Creation and Naming
```bash
# No specific command - visually verify that 4 VMs are created with descriptive names:
# - app-server
# - web-server-1
# - web-server-2
# - load-balancer
```

### 2. Resource Allocation
```bash
# Check CPU allocation
lscpu | grep "CPU(s):"

# Check RAM allocation
free -h

# Check disk space
df -h
```
Student should explain why specific resources were allocated to each VM (e.g., why load balancer has 2 CPUs, app server has more RAM)

## Network Configuration

### 3. Hostname Verification
```bash
# On each VM, verify the hostname is set correctly
hostname

# From each VM, ping all other VMs using hostnames
ping app-server -c 4
ping web-server-1 -c 4
ping web-server-2 -c 4
ping load-balancer -c 4

# Check hosts file configuration
cat /etc/hosts
```

### 4. Static IP Verification
```bash
# Check current IP configuration
ip a

# Verify netplan configuration
cat /etc/netplan/50-cloud-init.yaml

# Reboot to verify persistence
sudo reboot

# After reboot, check IP again
ip a
```

### 5. VM Communication Test
```bash
# From each VM, ping all other VMs using IP addresses
ping 192.168.189.101 -c 4  # app-server
ping 192.168.189.102 -c 4  # web-server-1
ping 192.168.189.103 -c 4  # web-server-2
ping 192.168.189.104 -c 4  # load-balancer
```

### 6. Browser Accessibility Test
```bash
# Test accessibility to load balancer from external machine
curl http://192.168.189.104

# Verify other servers are NOT accessible
curl http://192.168.189.101
curl http://192.168.189.102
curl http://192.168.189.103
```
Student should explain that only the load balancer should be accessible from outside because it provides a controlled entry point for external traffic, while other servers should only communicate internally for security.

## System Security

### 7. Update Status
```bash
# Check for upgradable packages on each VM
sudo apt update && sudo apt list --upgradable
```

### 8. DevOps User Creation
```bash
# Verify the devops user exists
grep devops /etc/passwd

# Check user groups
groups devops
```

### 9. SSH Authentication
```bash
# Test SSH with devops user (should work without password prompt)
ssh devops@192.168.189.101
ssh devops@192.168.189.102
ssh devops@192.168.189.103
ssh devops@192.168.189.104

# Test root login (should be denied)
ssh root@192.168.189.101

# Test other user login (should be denied)
ssh test_user@192.168.189.101

# Test SSH password authentication (should be denied)
# Try SSH with -o to force password authentication
ssh -o PubkeyAuthentication=no devops@192.168.189.101
```

### 10. Sudo Password Protection
```bash
# As devops user, run a sudo command
sudo ls -la /root

# Should prompt for password even if used recently
sudo visudo
```

### 11. Network Interface Management
```bash
# List all network interfaces
ip link show

# Check for disabled interfaces
ls -l /sys/class/net/
```
Student should explain which interfaces are active and why (typically lo and ens33), and confirm any unused interfaces are disabled.

### 12. Firewall Configuration
```bash
# Check UFW status with verbose output
sudo ufw status verbose

# Test specific ports based on VM role
# On load balancer - these should succeed:
curl localhost:80
curl localhost:443
curl localhost:19999

# On web servers - these should fail from external sources but work from load balancer
# Test from external machine:
curl http://192.168.189.102:80
# Test from load balancer:
curl http://10.0.0.2:80
```

### 13. Umask Configuration
```bash
# Check current umask
umask

# Test file creation permissions
touch test_file
ls -la test_file

# Check umask configuration
cat /etc/profile | grep umask
```

### 14. Automatic Updates
```bash
# Check unattended-upgrades configuration
cat /etc/apt/apt.conf.d/20auto-upgrades

# Verify service is running
systemctl status unattended-upgrades
```

### 15. SSH Protocol Version
```bash
# Check SSH protocol configuration
grep Protocol /etc/ssh/sshd_config

# Verify SSH version used (should be protocol 2)
ssh -v devops@localhost 2>&1 | grep "protocol version"
```

## Extra Requirements

### 16. Intrusion Prevention (Fail2Ban)
```bash
# Check Fail2Ban status
sudo fail2ban-client status

# Check specific jail status
sudo fail2ban-client status sshd

# View Fail2Ban configuration
cat /etc/fail2ban/jail.local

# Demonstrate functionality (Warning: This will temporarily lock you out!)
# Simulate 3 failed login attempts
ssh invalid_user@192.168.189.101
```
Student should explain how Fail2Ban works, what it's monitoring, and how it helps secure the server.

### 17. VPN Configuration (WireGuard)
```bash
# Check WireGuard status
sudo wg show

# Test VPN connectivity
ping 10.0.0.1 -c 4  # app-server VPN IP
ping 10.0.0.2 -c 4  # web-server-1 VPN IP
ping 10.0.0.3 -c 4  # web-server-2 VPN IP
ping 10.0.0.4 -c 4  # load-balancer VPN IP

# Check WireGuard configuration
cat /etc/wireguard/wg0.conf
```
Student should explain why VPN is important for secure internal communication.

### 18. Monitoring Setup (NetData)
```bash
# Check NetData service status
sudo systemctl status netdata

# Try accessing NetData dashboard (only from load balancer)
curl http://192.168.189.104:19999

# On child nodes, check streaming configuration
cat /etc/netdata/stream.conf
```
Student should explain how NetData works, the parent-child relationship, and how it helps with system monitoring.

### 19. Load Balancer Configuration
```bash
# Check NGINX configuration on load balancer
cat /etc/nginx/sites-available/default

# Test load balancing by refreshing multiple times
for i in {1..10}; do curl -s http://192.168.189.104 | grep "Server"; done
```
Student should explain how the load balancer distributes traffic between web servers.
