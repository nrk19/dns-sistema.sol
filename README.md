# DNS sistema.sol # 

## INTRODUCTION ##

We are going to define a zone called sistema.sol, which will be located in the net **192.168.57.0/24**
The domain will consists of four servers:

|        FQDN          |  IP  |
|----------------------|------|
| mercurio.sistema.sol | .101 |
| venus.sistema.sol    | .102 |
| tierra.sistema.sol   | .103 |
| marte.sistema.sol    | .104 |

- **Tierra** will be the master, and will be authoritative of both zones, direct and reverse.
- **Venus** will be the slave nameserver.
- **Marte** will be the mail server.

## CONFIGURATION ## 

### NAMED CONFIGURATION ###

First, we will set tierra and venus as the default DNS servers (it will be useful to test the configuration later) 

```conf
nameserver 192.168.57.103
nameserver 192.168.57.102
```

#### named.conf.options config ####

*(All the configuration files of bind are located in `/etc/bind/`)* 

We will begin editing the file `named.conf.options`. In this file we will set settings such as, trusted networks, external forwarders, enable or disable recursion, etc.
To comply with the objetives of the task we will set the following options (in both servers):

```conf
acl trusted {

	127.0.0.0/8;
	192.168.57.0/24;
};

options {
	directory "/var/cache/bind";

	forwarders {
	 	208.67.222.222;
	 };
	
	forward only;

	allow-transfer { trusted;};
	
	listen-on port 53 { 192.168.57.103; };
	
	recursion yes;
	allow-recursion { trusted; };

	dnssec-validation yes;

	// listen-on-v6 { any; };
};
```

**Explanation of the choosen options**

- **acl**: here we set our trusted networks
- **forwarders**: we set 208.67.222.222 for non-authoritative request
- **allow-transfer { trusted; }**: we allow transfer from our trusted network. This will allow the transfer of the zone between the master and the slave.
- recursion: allow recursion from trusted network.

#### named.conf.local ####

In this file is were the zones are defined, is here where we will indicate where the files of each zone will be store. Also, we need to set the role of the server (master or slave). The configuration of the file will be sligthy different in this case.

**Master:**

```conf
zone "sistema.sol" {
	type master;
	file "/var/lib/bind/sistema.sol.dns";
};

zone "57.168.192.in-addr.arpa" {
	type master;
	file "/var/lib/bind/sistema.sol.rev";
};
```
**Slave:**

```conf
zone "sistema.sol" {
	type slave;
	masters { 192.168.57.103; };
	file "/var/lib/bind/sistema.sol.dns";
};

zone "57.168.192.in-addr.arpa" {
	type slave;
	masters { 192.168.57.103; };
	file "/var/lib/bind/sistema.sol.rev";
};
```

Now we can restart the named service to apply the changes. We can test the configuration with the command `# named-checkconf [file]` for config files and `# named-checkzone [zone] [file]` for checking zone files.

### ZONES CONFIGURATION ###

#### DIRECT ZONE (/var/lib/bind/sistema.sol.dns) ####

As we indicated in the file `named.conf.local`, we will store the zone config file at `/var/lib/bind/sistema.sol.dns`. We can use the file `/etc/bind/db.empty` as a template. To comply with the objetives, the file should look like this: 

```conf
;
; zone: sistema.sol.
;

$TTL	86400
@	IN	SOA	tierra.sistema.sol. admin.sistema.sol. (
			      1		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			  7200 )	; Negative Cache TTL
;

; RESOLVE RECORDS (RR)
; name servers (NS)
@			IN	NS	tierra.sistema.sol.
@			IN	NS	venus.sistema.sol.

; host addresses (A)
tierra.sistema.sol.	IN	A	192.168.57.103
mercurio.sistema.sol.	IN	A	192.168.57.101
venus.sistema.sol.	IN	A	192.168.57.102
marte.sistema.sol.	IN	A	192.168.57.102

; mail servers (MX)
sistema.sol.		IN	MX 10	marte.sistema.sol.

; aliases
ns1.sistema.sol.	IN	CNAME	tierra.sistema.sol.
ns2.sistema.sol.	IN	CNAME	venus.sistema.sol.
mail.sistema.sol.	IN	CNAME	marte.sistema.sol.
```

#### REVERSE ZONE (/var/lib/bind/sistema.sol.rev) ####

```conf
;
; 57.168.192
;

$TTL	86400
@	IN	SOA	tierra.sistema.sol. admin.sistema.sol. (
			      1		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			  7200 )	; Negative Cache TTL
;


; RESOLVE RECORDS (RR)
; name servers (NS)
@			IN	NS	tierra.sistema.sol.
@			IN	NS	venus.sistema.sol.

; reverse host resolution
103			IN	PTR	tierra.sistema.sol.
101			IN	PTR	mercurio.sistema.sol.
102			IN	PTR	venus.sistema.sol.
104			IN	PTR	marte.sistema.sol.
```

