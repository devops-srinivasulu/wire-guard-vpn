You need to configure WireGuard so **any authorized peer connected to your VPN can access both your database server and backend server**.

You don't need separate rules for every laptop. You can allow the entire WireGuard VPN subnet.

## 1. Required access

Your intended architecture:

| Server | Private IP | Ports to allow |
|---|---|---|
| Database server | `10.0.21.46` | `5432`, `27017` |
| Backend server | `10.0.11.10` | `8081`, `8085`, `8091` |

WireGuard VPN subnet:

```text
10.200.0.0/24
```

This covers peers such as:
- `10.200.0.2`
- `10.200.0.3`
- `10.200.0.4`

## 2. Update firewall rules

Connect to your public CICD server:

`sankranti-prod-cicd`

Run:

```bash
# Database access: PostgreSQL and MongoDB
sudo iptables -I WG-FWD 2 -i wg0 -o ens5 \
-s 10.200.0.0/24 -d 10.0.21.46/32 \
-p tcp -m multiport --dports 5432,27017 -j ACCEPT

# Backend access: Sankranti services
sudo iptables -I WG-FWD 2 -i wg0 -o ens5 \
-s 10.200.0.0/24 -d 10.0.11.10/32 \
-p tcp -m multiport --dports 8081,8085,8091 -j ACCEPT
```

## 3. Configure NAT

Run:

```bash
sudo iptables -t nat -I POSTROUTING 1 \
-o ens5 -s 10.200.0.0/24 \
-d 10.0.0.0/16 -j MASQUERADE
```

This allows return traffic to go through your WireGuard server without requiring separate VPC routes for every VPN peer.

## 4. Check AWS Security Groups

On the database and backend EC2 instances, ensure their security groups allow inbound traffic from:

```text
Source: 10.0.1.245/32
```

Here, `10.0.1.245` is your WireGuard public CICD server's private IP.

Allow the respective ports:

**Database server:**
```text
TCP 5432
TCP 27017
```

**Backend server:**
```text
TCP 8081
TCP 8085
TCP 8091
```

## 5. Test from your laptop

With WireGuard active, run PowerShell:

```powershell
Test-NetConnection 10.0.21.46 -Port 5432
Test-NetConnection 10.0.21.46 -Port 27017

Test-NetConnection 10.0.11.10 -Port 8081
Test-NetConnection 10.0.11.10 -Port 8085
Test-NetConnection 10.0.11.10 -Port 8091
```

All five should show:

```text
TcpTestSucceeded : True
```

## 6. Make the configuration permanent

After confirming all five connections work, add the equivalent forwarding and NAT rules to your WireGuard `wg0.conf` under `PostUp`, with matching cleanup rules under `PostDown`.

**Important:** Allowing the entire VPN subnet means every peer you authorize on WireGuard can access these database and backend ports. Only add trusted devices to your VPN.

My recommendation: First apply the runtime rules and test all five ports. Then we'll make the configuration persistent without disturbing your existing monitoring rules.
