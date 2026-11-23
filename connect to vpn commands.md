Perfect. Now you can add another WireGuard VPN peer.

Since we configured access for the entire VPN subnet `10.200.0.0/24`, the new peer should automatically be able to access the same resources, provided the firewall and routing rules are in place.

### Steps for adding another peer

**1. Generate a new key pair on the new device.**

```bash
wg genkey | tee privatekey | wg pubkey > publickey
```

**2. Add the new peer's public key on `sankranti-prod-cicd`:**

Edit:

```bash
sudo nano /etc/wireguard/wg0.conf
```

Add:

```ini
[Peer]
# New VPN User
PublicKey = NEW_PEER_PUBLIC_KEY
AllowedIPs = 10.200.0.5/32
```

Use an unused VPN IP. Your existing peers are `.2`, `.3`, and `.4`, so `.5` is the next available address.

**3. Apply the new peer without restarting WireGuard:**

```bash
sudo wg set wg0 peer NEW_PEER_PUBLIC_KEY allowed-ips 10.200.0.5/32
```

**4. Configure the new device:**

```ini
[Interface]
PrivateKey = NEW_PEER_PRIVATE_KEY
Address = 10.200.0.5/32

[Peer]
PublicKey = SERVER_PUBLIC_KEY
Endpoint = YOUR_PUBLIC_SERVER_IP:51820
AllowedIPs = 10.0.0.0/16, 10.200.0.0/24
PersistentKeepalive = 25
```

**5. Test from the new device:**

```powershell
Test-NetConnection 10.0.21.46 -Port 27017
Test-NetConnection 10.0.21.46 -Port 5432
Test-NetConnection 10.0.11.10 -Port 8081
Test-NetConnection 10.0.11.10 -Port 8085
Test-NetConnection 10.0.11.10 -Port 8091
```

All should return `TcpTestSucceeded : True`.

**Important:** Save the new `[Peer]` entry in `wg0.conf` as well as applying it with `wg set`, otherwise it won't survive a WireGuard restart. Keep the private key only on the new device; never share it.
