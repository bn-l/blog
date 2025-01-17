---
title: "Running WireGuard GUI on windows with a non-admin account"
description: "Unlike other solutions, this does not require unsafe adding to the admin group or running an admin account"
date: 2023-04-12
draft: false
slug: wireGuard-windows-non-admin
---

All other solutions to the problem of running wireguard on windows are not ideal. The methods I've seen are:

1. Just running as an admin
2. Adding the user account to the Network Configuration Operators
    - Pros: Wireguard gui, works smoothly
    - Cons: Your regular user gets added to an admin group and will appear in UAC prompts. This is very annoying and probably bad security.
3. Running the WireGuard tunnel as a windows service (as suggested in [this](https://serverfault.com/a/1126482) answer)
    - Pros: Works perfectly
    - Cons: No gui, needs an elevated console.
4. Using Task Scheduler as in [this](https://superuser.com/a/1751055/1789647) answer (didn't work for me at all).

**Solution:**

1. Install the latest MSI: https://download.wireguard.com/windows-client/

2. Then run this command in elevated console with your .conf file:

    `wireguard /installtunnelservice C:\path\to\some\myconfname.conf`

> This creates a service called `WireGuardTunnel$myconfname`, which can be controlled using standard Windows service management utilites, such as services.msc or sc. — [source](https://git.zx2c4.com/wireguard-windows/about/docs/enterprise.md)

1. Control the service with "ServiceTray": https://www.coretechnologies.com/products/ServiceTray/

    (This give you a nice icon on the system tray that shows the up status of the WireGuard tunnel service. Green = connected, red = not connected, and you can start and stop it by right clicking. See image below)

    Note: When creating the service controller, save the shortcut to desktop not startup (doesn't seem to work and you can copy to startup later)

2. (optional) Change the service's startup type to manual if you don't want to be connected to the tunnel on startup.

[An example of what this looks like on Win 10](https://i.stack.imgur.com/DerWx.png). Hovering the icon shows the name of the tunnel.



From my answer at: https://superuser.com/a/1778666/1789647