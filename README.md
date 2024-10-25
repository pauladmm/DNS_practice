# DNS Practice

This project includes a primary DNS server, a secondary DNS server, and configurations for a forward and reverse zone.

### Initial Setup

First, we set up virtual machines with the following configurations:

| Server   | OS         | Hostname              | IP             |
| -------- | ---------- | --------------------- | -------------- |
| mercurio |            | mercurio.sistema.test | 192.168.57.101 |
| venus    | Debian CLI | venus.sistema.test    | 192.168.57.102 |
| tierra   | Debian CLI | tierra.sistema.test   | 192.168.57.103 |
| marte    |            | marte.sistema.test    | 192.168.57.104 |

The host network is set to `192.168.57.0/24`.

### Steps

1. **Activate server listening for IPv4 protocol only**  
    Modify the BIND9 configuration files (`/etc/default/named`) on both servers to enable IPv4 listening only:

   `OPTIONS="-u bind -4"`
   Restart BIND with sudo systemctl restart bind9, and verify IPv4 listening with:

   `sudo ss -tuln | grep :53`

2. **Enable DNSSEC Validation**

Set dnssec-validation to yes in /etc/bind/named.conf.options and disable IPv6 listening if necessary. Restart BIND and verify answer is validated.

`dig +dnssec google.com`

3. **Configure Recursive Queries for Specific Networks**

Configure acl in named.conf.options to allow recursive queries only from 127.0.0.0/8 and 192.168.57.0/24.

4. **Configure Primary Server (tierra.sistema.test)**

Set tierra.sistema.test as the authoritative server for the forward (sistema.test) and reverse (192.168.57.0/24) zones.
Update /etc/bind/named.conf.local to define the server as master.
Check configurations with:

`named-checkconf /etc/bind/named.conf.local`

Create forward (/var/lib/bind/db.sistema.test) and reverse (/var/lib/bind/db.192.168.57) zone files and verify them:

`named-checkzone sistema.test /var/lib/bind/db.sistema.test`

`sudo named-checkzone 57.168.192.in-addr.arpa /var/lib/bind/db.192.168.57`

Restart BIND.

5. **Configure Secondary Server (venus.sistema.test)**

Define venus.sistema.test as the secondary server in /etc/bind/named.conf.local.
Set up zone transfers (allow-transfer) and notify settings on the primary server for venus.
Add DNS record for the secondary server in the master zone file and verify with:

`dig @192.168.57.103 sistema.test axfr`

6. **Set Negative Cache TTL**
   In the master zone file (db.sistema.test), set the TTL for negative responses to 7200 seconds. Restart BIND.

7. **Forward Unresolved Queries to OpenDNS**
   Configure the master server to forward unresolved queries to OpenDNS (208.67.222.222) in named.conf.options:

`forwarders { 208.67.222.222; };`

Verify with:

`named-checkconf /etc/bind/named.conf.local`

8. **Configure Aliases**

Set ns1.sistema.test as an alias for tierra.sistema.test and ns2.sistema.test as an alias for venus.sistema.test.
Update the primary zone file (db.sistema.test) to include these CNAME records and verify with:

`dig @localhost ns1.sistema.test`
`dig @localhost ns2.sistema.test`

9. **Add mail.sistema.test Alias**

Define mail.sistema.test as an alias for marte.sistema.test in the primary zone file. Verify with:

`dig @localhost mail.sistema.test`

10. **Configure marte.sistema.test as Mail Server**

Add an MX record for marte.sistema.test or mail.sistema.test in the primary zone file to set it as the mail server for sistema.test. Verify with:

`dig @localhost sistema.test. MX`
