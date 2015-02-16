# needless-steel
Utility to assist in keeping track of validity TLS certificates and DNSSEC signatures

## How to install

1. Copy relevant files to /opt/chkexp/bin and ensure correct owner and permission

# mkdir -p /opt/chkexp/bin
# cp chkexp dnssec-chkexp tls-chkexp /opt/chkexp/bin
# chmod -R 555 /opt/chkexp
# chown -R root:root /opt/chkexp

2. Copy chkexp.conf to /etc/opt/chkexp/chkexp.conf and modify it to add the checks you need

To add a check for a TLS certificate on a SMTP MTA and to check the validity
of DNSSEC signature on a zone, one might add something like

```
%CFG = (
	'DNSSEC' => {
		'domain.se' => {
			'threshold'	=> '24',
			'contact'	=> 'hostmaster@domain.se',
		}
	},
	'SMTP' => {
		'smtp.domain.se' => {
			'threshold'	=> '30',
			'contact'	=> 'postmaster@domain.se,hostmaster@domain.org',
		},
	}
);
```

Make sure to check validity of the file afterward by issuing a "perl -c ./chkexp.conf",
it should say something like

```
$ perl -c ./chkexp.conf
./chkexp.conf syntax OK
```

3. Create a harmless user, in our example we create a user called "chkexp"

`# useradd chkexp`

4. Create a crontab entry for the new user, to run the "chkexp" script at least once a day.

```
# crontab -u chkexp <<!EOF!
# m h  dom mon dow   command
5 7 * * * /opt/chkexp/bin/chkexp --alert >/dev/null 2>&1
!EOF!
```

5. Do a manual test to see that everything is working as expected

```
$ /opt/chkexp/bin/chkexp -v domain.se
Running test "/opt/chkexp/bin/dnssec-chkexp  -v --warn=24 domain.se" -- passed
domain.se is delegated to a.dns.se (10.20.30.19)
domain.se is delegated to c.dns.se (10.30.40.2)
domain.se is delegated to b.dns.se (192.168.100.95)
10.20.30.19: zone "domain.se" verified with signature made with key 61821.
10.30.40.2: zone "domain.se" verified with signature made with key 61821.
192.168.100.95: zone "domain.se" verified with signature made with key 61821.

Running test "/opt/chkexp/bin/tls-chkexp  -v --warn=30 --delay=0 domain.se" -- passed
INFO: Certificate expires in 586 days. (expires on Sep 25 06:40:08 2016 GMT)

Running test "/opt/chkexp/bin/tls-chkexp  -v --warn=30 --type=mx --delay=0 --smtp domain.se" -- passed
INFO: Certificate expires in 586 days. (expires on Sep 25 06:40:08 2016 GMT)
```

6. Done :)
