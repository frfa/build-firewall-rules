# `build-firewall-rules` -- a tool to generate synchronized iptables IPv4 and IPv6 rules from a single template file

This tool reads a template file with rules defined in a DSL (domain-specific
language) that produces iptables filter rules suitable for loading via
`iptables-restore`. For details about the syntax of this DSL see the file
[build-firewall-rules.md](build-firewall-rules.md).

The default rules template file is `/etc/iptables/rules.template`, and the IPv4
and IPv6 rules files are `/etc/iptables/rules.v4` and `/etc/iptables/rules.v6`
respectively.

---

## 🛟 Usage

Use `build-firewall-rules --help` for usage information:

```
$ ./build-firewall-rules --help
W root privileges required, re-starting with sudo ...
Usage: build-firewall-rules [OPTIONS] [TEMPLATE_FILE]

Generates synchronized /etc/iptables/rules.v4 and /etc/iptables/rules.v6
from a single rules template file.

Options:
  -v, --verbose  Print verbose log information to standard error
  -n, --noexec   Generates and validates rules, but does not overwrite any files
  -q, --quiet    Suppresses normal log output, only errors will be displayed
  -h, --help     Shows this help text
  -l, --live     Immediately activates generated rules after successful validation

Default TEMPLATE_FILE: /etc/iptables/rules.template
```

> NOTE: `build-firewall-rules` requires root privileges for its operation, as
> writing to the `/etc/iptables` directory and loading iptables rules with
> `iptables-restore` are privileged operations. Therefore, invoke it as
> the `root` user, otherwise `build-firewall-user` will restart itself using
> `sudo` which requires you to type in the root password once.

To generate the default rulesets `/etc/iptables/rules.v4` and
`/etc/iptables/rules.v6` from the default template
`/etc/iptables/rules.template` run:

```
$ ./build-firewall-rules -v
```

This will generate the above mentioned rulesets, and validate them
afterwards. Note: If there are errors in the generated rulesets you have a
potentially broken iptables configuration -- do not activate those rules! To
avoid this situation of overwriting a _last known good_ configuration, use the
`-n/--noexec` flag, that runs rule generation in dry-mode, **not** overwriting
any `/etc/iptables/rules.v4` or `rules.v6` files. This uses temporary files as a
result, and tests the rulesets of those temporary files, flagging potential
errors. This allows you to correct possible errors in the source
`rules.template` file before loading and activating the rulesets:

```
# ./build-firewall-rules -v -n
I Processing rules template examples/rules.template
I Backing up existing rulesets
I Validating rule syntax with iptables-restore / ip6tables-restore ...
I [SUCCESS] /etc/iptables/rules.v4 and /etc/iptables/rules.v6 generated and validated.
```

The generated rules can be loaded and activated using `iptables-restore
</etc/iptables/rules.v4` or `ip6tables-restore /etc/iptables/rules.v6`
respectively. Even simpler, just use `./build-firewall-rules -v -l` after
testing the rules with `./build-firewall-rules -v -n`:

```
# bin/build-firewall-rules -v -l
I Processing rules template /etc/iptables/rules.template
I Backing up existing rulesets
I Validating rule syntax with iptables-restore / ip6tables-restore ...
I [SUCCESS] /etc/iptables/rules.v4 und /etc/iptables/rules.v6 generated and validated.
N [LIVE] Activating IPv4 rules /etc/iptables/rules.v4 now!
N [LIVE] Activating IPv6 rules /etc/iptables/rules.v6 now!
```

> ⚠️ NOTE: In order to make these rulesets persistent after reboots, install the
> package `iptables-persistent` (`apt install iptables-persistent` on
> Debian-based OSes), which creates a one-shot systemd job that loads
> those rules at startup.

---

## 🗒️ Examples and `ipset`

You may find a detailed example for a dual-homed firewall setup with droplists,
Geo-IP-based routing, and Wireguard VPN rules in [examples/rules.template](examples/rules.template).

⚠️ Note that in order to make this example functional you need `ipset` to be
installed, as the example uses droplists from <spamhaus.org> and GeoIP-based
routing. To make this example work without droplists and GeoIP-based routing ,
just comment out the respective rules loading the blocklists and GeoIP-based
packet marking.

```
#CC 'Spamhaus DROPlist using ipset'
#BLOCKLIST4 "$SPAMHAUS_SET_V4"
#BLOCKLIST6 "$SPAMHAUS_SET_V6"
```

And:

```
#C 'GeoIP-based routing for country domains (mark packets with mark 100)'
#R4 PREROUTING "`TARGET MARK 100`" `MATCHSET "$GEOIP_SET" dst`
#R4 OUTPUT "`TARGET MARK 100`" `MATCHSET "$GEOIP_SET" dst`
```

If you want to use those `ipset` features, install the `ipset` package (`apt
install ipset` on Debian-based OSes), and create the IP sets used in the
`rules.template` file:

```
ipset create spamhaus_drop hash:net family inet hashsize 1024 maxelem 65536
ipset create spamhaus_drop_v6 hash:net family inet6 hashsize 1024 maxelem 65536
ipset create geoip_country hash:net family inet hashsize 4096 maxelem 131072
```

This creates empty IP sets that you must populate using the `ipset SETNAME
ENTRY` command, or restore the set using `ipset restore`, after saving it using
`ipset save`. Note that in order to make those sets persistent you have to
create a systemd one-shot service to restore saved IP sets at startup:

```
[Unit]
Description=Restore ipset rules
Before=netfilter-persistent.service iptables.service
DefaultDependencies=no

[Service]
Type=oneshot
ExecStart=/sbin/ipset restore -f /etc/iptables/ipset.rules
ExecStop=/sbin/ipset save geoip_country -f /etc/iptables/ipset.rules
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

Save as `/etc/systemd/system/ipset-persistent.service` and activate with
`systemctl daemon-reload; systemctl enable --now ipset-persistent.service`.

---

Enjoy!

-- @frfa
