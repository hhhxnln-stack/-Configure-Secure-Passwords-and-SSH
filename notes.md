# Notes — Configure Secure Passwords and SSH

## Command reference

| Device | Command | Purpose |
|--------|---------|---------|
| RTA | `hostname RTA` | Set device name (used in RSA key name) |
| RTA | `ip address 172.16.1.1 255.255.255.0` | Assign LAN IP |
| RTA | `no shutdown` | Enable interface (router ports are down by default) |
| RTA | `service password-encryption` | Encrypt plaintext passwords as type 7 |
| RTA | `security passwords min-length 10` | Reject passwords shorter than 10 chars |
| RTA | `enable secret Cisco12345!` | Protect privileged EXEC (hashed) |
| RTA | `no ip domain-lookup` | Prevent delays on mistyped commands |
| RTA | `ip domain-name netsec.com` | Required for RSA key generation |
| RTA | `username admin secret Admin12345!` | Local user for SSH authentication |
| RTA | `crypto key generate rsa` → 1024 | Generate RSA key pair (required for SSH) |
| RTA | `login block-for 180 attempts 4 within 120` | Block logins 180s after 4 failed attempts in 120s |
| RTA | `transport input ssh` | Allow only SSH on VTY (Telnet disabled) |
| RTA | `login local` | Authenticate against local user DB |
| RTA | `exec-timeout 6 0` | Auto-logout after 6 minutes idle |
| RTA | `copy running-config startup-config` | Save config to NVRAM |
| SW1 | `hostname SW1` | Set device name |
| SW1 | `interface vlan 1` + `ip address 172.16.1.2 255.255.255.0` | Management IP on L2 switch |
| SW1 | `no shutdown` | Enable SVI |
| SW1 | `ip default-gateway 172.16.1.1` | Allow switch to respond from other subnets |
| SW1 | `interface range f0/2-24, g0/2` + `shutdown` | Disable unused ports |
| SW1 | `service password-encryption` | Encrypt plaintext passwords |
| SW1 | `enable secret Cisco12345!` | Protect privileged mode |
| SW1 | `no ip domain-lookup` | Disable DNS resolution |
| SW1 | `ip domain-name netsec.com` | Set domain for RSA |
| SW1 | `username admin secret Admin12345!` | Create local user |
| SW1 | `crypto key generate rsa` → 1024 | Generate RSA keys |
| SW1 | `transport input ssh` | SSH only on VTY |
| SW1 | `login local` | Local authentication |
| SW1 | `exec-timeout 6 0` | 6-minute idle timeout |
| SW1 | `copy running-config startup-config` | Save config |

## Password encryption levels

| Type | Algorithm | Reversible? | Where used |
|------|-----------|-------------|------------|
| 0 | Plaintext | Yes | Default line passwords |
| 7 | Vigenère | Yes (weak) | After `service password-encryption` |
| 5 | MD5 | No (hash) | `enable secret` in Packet Tracer; legacy in real IOS |
| 9 | SCRYPT | No (hash) | `enable algorithm-type scrypt secret` in real IOS |

**Type 5 and 9 are harder to crack** because they are one-way hashes. Type 7 is reversible XOR-based obfuscation and can be decoded with widely available tools.

## `login` vs `login local`

| | `login` | `login local` |
|---|---------|---------------|
| What it checks | Line password | Local user database |
| Username prompt | No | Yes |
| Per-user privilege | No | Yes |
| Visibility (`show users`) | No | Yes |
| Use case | Simple labs | Real networks |

`login local` is required for SSH because SSH needs to know **who** is connecting and what privileges they should have.

## Packet Tracer limitations

- `enable algorithm-type scrypt secret` — not supported. Use `enable secret` (type 5 MD5).
- `username ... algorithm-type scrypt` — not supported. Use `username ... secret` (type 5 MD5).
- `show users` — the Location column is empty in PT; on real hardware it shows the source IP.
- `ip ssh version 2` — may not be supported in some PT versions; SSH 1.99 still works.

## What I learned

- RSA keys are mandatory for SSH — without them, `transport input ssh` will not work.
- `service password-encryption` only protects against casual reading, not against a determined attacker.
- `login local` is required for SSH because SSH needs to identify the connecting user.
- Disabling unused switch ports removes potential entry points.
- `security passwords min-length` applies to line passwords and user passwords.
- `login block-for` is a simple but effective brute-force mitigation.
