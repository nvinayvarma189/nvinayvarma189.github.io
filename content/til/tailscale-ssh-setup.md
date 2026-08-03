+++
title = "Tailscale SSH Setup"
date = 2026-04-15
updated = 2026-04-15
type = "post"
description = "How to setup Tailscale SSH and how is it different from regular SSH"
in_search_index = true
[taxonomies]
TIL-Tags = ["Tailscale", "SSH"]
+++

## 1. Does Tailscale SSH use the regular SSH port?

### Question

Does Tailscale SSH use the regular SSH port? If yes, then what is the point of Tailscale SSH?

### Answer

There are two different things:

- Regular SSH over the Tailscale network
- Tailscale SSH feature

#### Regular SSH over Tailscale

This is when you connect to your VPS's Tailscale IP using your normal SSH server.

Example:

```bash
ssh root@100.x.x.x -p 7576
```

In this case:

- Your normal sshd server handles the connection
- You still use your usual SSH port (7576 in your case)
- You still use your normal password or SSH key
- Tailscale is only providing the private encrypted network path

#### Tailscale SSH Feature

This is a separate Tailscale feature.

In this mode:

- Tailscale itself handles SSH authentication
- It only works on port 22
- You can authenticate using your Tailscale account identity instead of passwords or SSH keys
- You can make rules like:
  - only my phone can SSH into the VPS
  - only certain users can SSH as root

Example:

```bash
tailscale ssh root@server-name
```

So in your setup, because your SSH daemon is listening on port 7576, you are currently using regular SSH over the Tailscale network, not the Tailscale SSH feature.

## 2. Why did I still have to specify the port and password in Termius?

### Question

Even with Tailscale turned on, why did I still need to provide:

- username
- password
- custom SSH port

### Answer

Because you are still using your regular SSH server.

Your VPS SSH server is configured like this:

- Port: 7576
- User: root
- Authentication: password or SSH key

Tailscale does not automatically replace those settings unless you explicitly use the Tailscale SSH feature.

So even when using Tailscale, you still had to enter:

- Tailscale IP
- Port 7576
- Username root
- Password

## 3. Can I completely shut off the public SSH port?

### Question

If I shut down port 7576 from the public internet, can I still connect as long as both the VPS and phone are on Tailscale?

### Answer

Yes.

If:

- your VPS is connected to Tailscale
- your phone is connected to the same Tailscale network
- public access to port 7576 is blocked

then only devices inside your Tailscale network can connect to the VPS.

People on the public internet will not even see port 7576 open.

Your phone will connect using the VPS's Tailscale IP address.

## 4. What happens if I close public SSH before Tailscale is ready?

### Question

What if I close public SSH access, then get disconnected before Tailscale is fully working?

### Answer

That is a real lockout risk.

Possible scenario:

- You close public SSH access
- Your SSH session disconnects
- Tailscale is not running or not logged in
- You can no longer connect through Tailscale
- Public SSH is already blocked

At that point, you may be locked out of the VPS.

Your only recovery methods would be:

- VPS provider console
- Rescue mode
- Serial console
- Web console
- Reinstall/reset by the VPS provider

## 5. What is the safe order to switch from public SSH to Tailscale-only SSH?

### Question

How should I safely move to Tailscale-only access without risking lockout?

### Answer

Safe sequence:

1. Install Tailscale
2. Confirm Tailscale is connected
3. Test SSH using the VPS Tailscale IP from another device
4. Confirm Tailscale starts automatically on boot
5. Only after successful testing, block public SSH access

## 6. Should I still keep some backup access method?

### Question

Should I completely rely on Tailscale?

### Answer

It is safer to keep at least one backup access method.

Examples:

- VPS provider web console
- Serial console
- Rescue mode
- A second admin account
- Public SSH restricted to your home IP only
- Temporary emergency firewall rule

That way, if Tailscale fails, you are not permanently locked out.

## 7. What is the advantage of SSH over Tailscale compared to leaving SSH open publicly?

### Question

What is the advantage of connecting over Tailscale instead of just leaving SSH open publicly on port 7576?

### Answer

Using SSH over Tailscale has several benefits:

- Your SSH port is invisible to the public internet
- No brute-force attacks
- No password guessing attempts
- No bot scanning
- Only devices inside your Tailscale network can even reach the SSH port
- Traffic stays inside the encrypted Tailscale network
- You can safely use passwords if needed because outsiders cannot even attempt to connect
- You do not need to remember your VPS public IP
- It still works if the VPS public IP changes
- It works even behind NAT or CGNAT

## 8. Is public SSH still safe if configured properly?

### Question

If I keep public SSH open, is it still safe?

### Answer

Yes, public SSH can still be safe if configured correctly.

Good practices:

- Disable root login
- Use SSH keys instead of passwords
- Use fail2ban
- Restrict by IP if possible
- Keep SSH updated
- Use a non-default port if desired

However, Tailscale is simpler and quieter because your SSH service is not exposed publicly at all.

## 9. What is the recommended setup?

### Recommended Setup

- SSH port: 7576
- Public firewall: block port 7576 completely
- Tailscale: enabled on boot
- Connection method: use VPS Tailscale IP only
- Emergency backup: VPS provider console access

This gives:

- No public SSH exposure
- No scanning or brute-force attempts
- Simple private access from your devices
- Backup recovery method if Tailscale fails
