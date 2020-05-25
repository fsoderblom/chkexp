# chkexp

Utility to assist in keeping track validity of TLS certificates and DNSSEC signatures

## How to install

### Copy files

Copy relevant files to /opt/chkexp/bin and ensure correct owner and permission

```bash
# mkdir -p /opt/chkexp/bin
# cp chkexp dnssec-chkexp tls-chkexp /opt/chkexp/bin
# chmod -R 555 /opt/chkexp
# chown -R root:root /opt/chkexp
```

### Add checks

Copy the distributed example file, chkexp.conf, to /etc/opt/chkexp/chkexp.conf and modify it to add the checks you need.

```bash
# cp chkexp.conf /etc/opt/chkexp/chkexp.conf
```

To add a check for a TLS certificate on a SMTP MTA and to check the validity
of DNSSEC signature on a zone, one might add something like

```perl
%CFG = (
    'DNSSEC' => {
        'domain.se' => {
            'threshold'  => '24',
            'contact'    => 'hostmaster@domain.se',
        }
    },
    'SMTP' => {
        'domain.se' => {
            'threshold'  => '30',
            'contact'    => 'postmaster@domain.se,hostmaster@domain.org',
        },
        'smtp.domain.se' => {
            'threshold'  => '30',
            'contact'    => 'postmaster@domain.se,hostmaster@domain.org',
            'type'       => 'host',
        },
    }
);
```

The "type" directive is optional and defaults to 'mx' if it is missing from the configuration.

Make sure to check validity of the file afterward by issuing a "perl -c ./chkexp.conf",
it should say something like

```bash
$ perl -c ./chkexp.conf
./chkexp.conf syntax OK
```

### Create a harmless user

In our example we create a mostly harmless user called "chkexp" to run the checks.

```bash
# useradd chkexp
```

### crontab entry

Create a crontab entry for the new user, to run the "chkexp" script at least once a day, the example below will run the check once every day at 7:05am in the morning.

```bash
# crontab -u chkexp <<!EOF!
# m h  dom mon dow   command
5 7 * * * /opt/chkexp/bin/chkexp --alert >/dev/null 2>&1
!EOF!
```

### Test the installation

Do a manual test to see that everything is working as expected

```bash
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

### Done

If you reached this far, you should be done.

### Contributors

Victor Johansson
