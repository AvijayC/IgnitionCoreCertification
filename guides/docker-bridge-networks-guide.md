# Docker Bridge Networks and Gateway Configuration Guide

## Overview

Docker bridge networks enable container communication and connectivity to external networks. This guide covers creating, customizing, and managing bridge networks with custom gateways.

## Default Bridge Network

### Understanding the Default Bridge
```bash
# Inspect default bridge
docker network inspect bridge

# Container on default bridge
docker run -d --name web nginx
```

**Limitations:**
- Containers communicate via IP only (no DNS resolution)
- All containers share same network segment
- Limited isolation and security

## Custom Bridge Networks

### Basic Custom Bridge
```yaml
version: '3.8'
networks:
  custom_bridge:
    driver: bridge
```

### Advanced Bridge Configuration
```yaml
networks:
  app_network:
    driver: bridge
    driver_opts:
      com.docker.network.bridge.name: custom_br0
      com.docker.network.bridge.enable_icc: "true"
      com.docker.network.bridge.enable_ip_masquerade: "true"
      com.docker.network.driver.mtu: "1500"
    ipam:
      config:
        - subnet: 172.20.0.0/16
          gateway: 172.20.0.1
          ip_range: 172.20.1.0/24
        - subnet: 2001:db8::/32
          gateway: 2001:db8::1
```

## IPAM (IP Address Management)

### Static IP Assignment
```yaml
version: '3.8'
services:
  gateway:
    image: alpine
    networks:
      app_network:
        ipv4_address: 172.20.0.10
        
  web:
    image: nginx
    networks:
      app_network:
        ipv4_address: 172.20.0.20

networks:
  app_network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
          gateway: 172.20.0.1
```

### Multiple Subnets
```yaml
networks:
  multi_subnet:
    driver: bridge
    ipam:
      config:
        - subnet: 10.0.1.0/24
          gateway: 10.0.1.1
        - subnet: 10.0.2.0/24 
          gateway: 10.0.2.1
```

## Custom Gateway Container

### Linux Router/Gateway Container
```dockerfile
# Dockerfile for custom gateway
FROM alpine:latest

RUN apk add --no-cache \
    iptables \
    iproute2 \
    dnsmasq \
    openssh \
    tcpdump

# Enable IP forwarding
RUN echo 'net.ipv4.ip_forward=1' >> /etc/sysctl.conf

COPY gateway-setup.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/gateway-setup.sh

EXPOSE 22 53 67

CMD ["/usr/local/bin/gateway-setup.sh"]
```

### Gateway Setup Script
```bash
#!/bin/sh
# gateway-setup.sh

# Enable IP forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward

# Configure iptables for NAT
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
iptables -A FORWARD -i eth0 -o eth1 -m state --state RELATED,ESTABLISHED -j ACCEPT

# Start dnsmasq for DHCP/DNS
dnsmasq --no-daemon --log-queries --log-dhcp \
    --interface=eth1 \
    --dhcp-range=192.168.100.10,192.168.100.100,12h \
    --dhcp-option=option:router,192.168.100.1 \
    --dhcp-option=option:dns-server,8.8.8.8
```

### Docker Compose with Custom Gateway
```yaml
version: '3.8'
services:
  gateway:
    build: ./gateway
    container_name: custom_gateway
    privileged: true
    cap_add:
      - NET_ADMIN
      - SYS_ADMIN
    networks:
      external_network:
        ipv4_address: 172.20.0.1
      internal_network:
        ipv4_address: 192.168.100.1
    ports:
      - "2222:22"  # SSH access to gateway
    volumes:
      - /lib/modules:/lib/modules:ro

  web1:
    image: nginx
    networks:
      - internal_network
    depends_on:
      - gateway

  web2:
    image: nginx  
    networks:
      - internal_network
    depends_on:
      - gateway

networks:
  external_network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
          gateway: 172.20.0.1

  internal_network:
    driver: bridge
    ipam:
      config:
        - subnet: 192.168.100.0/24
          gateway: 192.168.100.1
    internal: true  # No external access except through gateway
```

## Advanced Gateway Configurations

### Multi-Homed Gateway
```yaml
services:
  gateway:
    image: custom-gateway
    networks:
      dmz:
        ipv4_address: 10.0.1.1
      internal:
        ipv4_address: 192.168.1.1
      management:
        ipv4_address: 172.16.1.1
    cap_add:
      - NET_ADMIN
    privileged: true

networks:
  dmz:
    driver: bridge
    ipam:
      config:
        - subnet: 10.0.1.0/24
  
  internal:
    driver: bridge  
    ipam:
      config:
        - subnet: 192.168.1.0/24
    internal: true

  management:
    driver: bridge
    ipam:
      config:
        - subnet: 172.16.1.0/24
```

### Gateway with VPN
```dockerfile
FROM alpine:latest

RUN apk add --no-cache \
    openvpn \
    iptables \
    iproute2 \
    wireguard-tools

COPY openvpn.conf /etc/openvpn/
COPY wg0.conf /etc/wireguard/

COPY vpn-gateway.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/vpn-gateway.sh

CMD ["/usr/local/bin/vpn-gateway.sh"]
```

## Host Network Integration

### Bridge to Host Network
```yaml
version: '3.8'
services:
  bridge_gateway:
    image: custom-gateway
    network_mode: host
    privileged: true
    cap_add:
      - NET_ADMIN
    volumes:
      - ./create-bridge.sh:/usr/local/bin/create-bridge.sh
    command: /usr/local/bin/create-bridge.sh
```

### Host Bridge Creation Script
```bash
#!/bin/bash
# create-bridge.sh

# Create bridge interface
brctl addbr br-custom
ip addr add 192.168.50.1/24 dev br-custom
ip link set br-custom up

# Add host interface to bridge
brctl addif br-custom eth0

# Configure iptables
iptables -t nat -A POSTROUTING -s 192.168.50.0/24 ! -d 192.168.50.0/24 -j MASQUERADE
iptables -A FORWARD -i br-custom -o eth0 -j ACCEPT
iptables -A FORWARD -i eth0 -o br-custom -m state --state RELATED,ESTABLISHED -j ACCEPT

# Keep container running
tail -f /dev/null
```

### macvlan for Direct Host Access
```yaml
networks:
  macvlan_net:
    driver: macvlan
    driver_opts:
      parent: eth0  # Host interface
    ipam:
      config:
        - subnet: 192.168.1.0/24
          gateway: 192.168.1.1
          ip_range: 192.168.1.128/25
```

## Network Security and Isolation

### Firewall Gateway
```dockerfile
FROM alpine:latest

RUN apk add --no-cache iptables ipset fail2ban

COPY firewall-rules.sh /usr/local/bin/
COPY fail2ban.conf /etc/fail2ban/

CMD ["/usr/local/bin/firewall-rules.sh"]
```

### Firewall Rules Script
```bash
#!/bin/bash
# firewall-rules.sh

# Flush existing rules
iptables -F
iptables -t nat -F

# Default policies
iptables -P INPUT DROP
iptables -P FORWARD DROP  
iptables -P OUTPUT ACCEPT

# Allow loopback
iptables -A INPUT -i lo -j ACCEPT

# Allow established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow internal network
iptables -A FORWARD -s 192.168.100.0/24 -j ACCEPT

# Allow specific services
iptables -A INPUT -p tcp --dport 22 -j ACCEPT   # SSH
iptables -A INPUT -p tcp --dport 53 -j ACCEPT   # DNS
iptables -A INPUT -p udp --dport 53 -j ACCEPT   # DNS
iptables -A INPUT -p udp --dport 67 -j ACCEPT   # DHCP

# NAT for outbound traffic
iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -o eth0 -j MASQUERADE

# Start fail2ban
fail2ban-server -f
```

## Load Balancer Gateway

### HAProxy Gateway
```dockerfile
FROM haproxy:alpine

COPY haproxy.cfg /usr/local/etc/haproxy/haproxy.cfg
```

### HAProxy Configuration
```
# haproxy.cfg
global
    daemon
    maxconn 4096

defaults
    mode http
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms

frontend web_frontend
    bind *:80
    default_backend web_servers

backend web_servers
    balance roundrobin
    server web1 web1:80 check
    server web2 web2:80 check
    server web3 web3:80 check
```

### Complete Load Balancer Setup
```yaml
version: '3.8'
services:
  lb_gateway:
    image: haproxy:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg
    networks:
      - frontend
      - backend
    depends_on:
      - web1
      - web2

  web1:
    image: nginx
    networks:
      - backend

  web2:
    image: nginx
    networks:
      - backend

networks:
  frontend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.30.0.0/16
          
  backend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.31.0.0/16
    internal: true
```

## Network Troubleshooting

### Network Diagnostics Container
```dockerfile
FROM alpine:latest

RUN apk add --no-cache \
    curl \
    wget \
    netcat-openbsd \
    nmap \
    tcpdump \
    iperf3 \
    mtr \
    traceroute \
    dig \
    nslookup

CMD ["sh"]
```

### Troubleshooting Commands
```bash
# Inspect network
docker network inspect network_name

# Test connectivity between containers
docker exec container1 ping container2

# Check routing table
docker exec gateway ip route show

# Monitor traffic
docker exec gateway tcpdump -i any

# Test port connectivity
docker exec container nc -zv target_host port
```

## Performance Optimization

### Bridge Network Tuning
```yaml
networks:
  high_performance:
    driver: bridge
    driver_opts:
      com.docker.network.driver.mtu: "9000"  # Jumbo frames
      com.docker.network.bridge.enable_icc: "true"
      com.docker.network.bridge.host_binding_ipv4: "0.0.0.0"
```

### Gateway Performance Settings
```bash
# Increase network buffers
echo 'net.core.rmem_max = 134217728' >> /etc/sysctl.conf
echo 'net.core.wmem_max = 134217728' >> /etc/sysctl.conf
echo 'net.ipv4.tcp_rmem = 4096 131072 134217728' >> /etc/sysctl.conf
echo 'net.ipv4.tcp_wmem = 4096 65536 134217728' >> /etc/sysctl.conf

# Apply settings
sysctl -p
```

## Monitoring and Logging

### Network Monitoring Stack
```yaml
version: '3.8'
services:
  gateway:
    image: custom-gateway
    volumes:
      - ./monitoring:/monitoring
    networks:
      - monitoring
      - app_network

  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    networks:
      - monitoring

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    networks:
      - monitoring

networks:
  monitoring:
    driver: bridge
  app_network:
    driver: bridge
```

## Best Practices

### Security
- Use internal networks for backend services
- Implement proper firewall rules
- Regular security updates for gateway containers
- Monitor network traffic for anomalies

### Performance  
- Use appropriate MTU sizes
- Configure buffer sizes for high throughput
- Consider SR-IOV for high performance requirements
- Monitor network metrics

### Maintenance
- Regular backup of network configurations
- Document custom network topologies
- Test failover scenarios
- Keep gateway containers updated

## Common Use Cases

### Development Environment
```yaml
networks:
  dev_network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.25.0.0/16
```

### Production Multi-Tier
```yaml
networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true
  database:
    driver: bridge
    internal: true
```

### Hybrid Cloud
```yaml
networks:
  cloud_bridge:
    driver: bridge
    ipam:
      config:
        - subnet: 10.0.0.0/8
```