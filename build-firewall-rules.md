# `build-firewall-rules` - create iptables IPv4 and IPv6 rules from template


This script uses `/etc/iptables/rules.template` as input to generate valid
 `/etc/iptables/rules.v4` for IPv4, and `/etc/iptables/rules.v6` iptables
 filter rule files, that can be loaded via `iptables-restore
 </etc/iptables/rules.v4` and `ip6tables-restore </etc/iptables/rules.v6`
 respectively.

> NOTE: Install `iptables-persistent` to load those filter-rules automatically
 at startup.



## Usage


build-firewall-rules is called as follows:
```
build-firewall-rules [OPTIONS] [TEMPLATE_FILE]
```

If the rules `TEMPLATE_FILE` is not provided, the default rules template
file `/etc/iptables/rules.template` is assumed.

The following options may be used on invocation:




Option | Argument | Description
:------|:---:|:------------
-v, --verbose | - | Print verbose log information to standard error
-n, --noexec | - | Generates and validates rules, but does not overwrite any files
-q, --quiet | - | Suppresses normal log output, only errors will be displayed
-h, --help | - | Shows this help text
-l, --live | - | Immediately activates generated rules after successful validation

> NOTE: Using the `-n, --noexec` and the `-l, --live` options at the same time
will reload the existing firewall rules `/etc/iptables/rules.v4` and
`/etc/iptables/rules.v6`, as only temporary files will be generated and
validated. Recommended usage is therefore to test new iptables rules with
`build-firewall-rules -n`, followed by a `build-firewall-rules -l` after
successful validation.



# Syntax of the rules template file `/etc/iptables/rules.template`




The rules template file is processed like any other shell script, so
employs POSIX-shell syntax. However, lots of useful shell functions for
firewall rule generation are predefined and form a kind of DSL
(Domain-Specific Language) to define a customized set of IPv4 and IPv6 rules.

Most of the DSL commands may be used with a "4" or "6" suffix to indicate
the corresponding rules should be generated for the IPv4 rules file
`/etc/iptables/rules.v4` or the IPv6 rules file `/etc/iptables/rules.v5`
respectively only.



## Comments


For example, comments in the `rules.template` file may be transliterated to
either both the `rules.v4` and `rules.v6` files only, when using the `CC` or
`C` DSL commands, while `C4` or `CC4` will transliterate the comment to the
`rules.v4` file only, and `C6` or `CC6` to the `rules.v6` file only.

Examples:
```
C 'This comment will appear both in the rules.v4 and rules.v6 file'
C4 'This comment will appear in the rules.v4 file only'
C6 'This comment will appear in the rules.v6 file only'
```



The difference between the 'C' and 'CC' variants is, that 'CC' will emit an
additional newline "`\n`" before the comment line, which enables structurizing
the generated rulesets and make if more human-readable.



> NOTE: Including a pure shell comment line starting with a hash sign "`#`" in
the rule template is possible, but will **not** transliterate the comment to
either ruleset!







`function` 
**`C`**
`()`


> A comment, transliterated both to v4 and v6 rulesets.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**String**</small> | **`in`** | The comment to transliterate.







`function` 
**`C4`**
`()`


> A comment, transliterated onyl to the v4 ruleset.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**String**</small> | **`in`** | The comment to transliterate.







`function` 
**`C6`**
`()`


> A comment, transliterated onyl to the v6 ruleset.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**String**</small> | **`in`** | The comment to transliterate.







`function` 
**`CC`**
`()`


> A comment preceded by a newline, transliterated to both v4 and v6 rulesets.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**String**</small> | **`in`** | The comment to transliterate.







`function` 
**`CC4`**
`()`


> A comment preceded by a newline, transliterated to only the v4 ruleset.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**String**</small> | **`in`** | The comment to transliterate.







`function` 
**`CC6`**
`()`


> A comment preceded by a newline, transliterated to only the v6 ruleset.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**String**</small> | **`in`** | The comment to transliterate.



## Table management


A new table is started with the `TABLE` command, followed by the table name
(e.v. `filter`, `mangle`, `nat`, etc.), and followed by pairs of chain name
and default policy for the chain for the chains in the table.







`function` 
**`TABLE`**
`()`


> Starts a new table. If the table name is `filter` not followed by any chain-policy pairs, the default setup will be `INPUT DROP FORWRD DROP OUTPUT ACCEPT`, meaning the default policy for the `INPUT` and `FORWARD` chains will be set to `DROP`, and the default policy for the `OUTPUT`chain will be `ACCEPT` -- a typical default firewall setup. A preceding `TABLE` will automatically be `COMMIT`ted, if applicable.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**name**</small> | **`in`** | The table name
> <small>**chain**</small> | **`in`** | The chain name
> <small>**policy**</small> | **`in`** | The default policy for the chain
> <small>**...**</small> | **`-`** | More chain-policy pairs


> **Examples:**  
> `TABLE filter`, same as  
 `TABLE filter INPUT DROP FORWARD DROP OUTPUT ACCEPT` <br> You can also add other chain-policy pairs as necessary, e.g `DOCKER-USER -`, indicating a Docker user chain without a default policy, as this chain is managed by the Docker daemon.  

## Generic DSL commands


Generic commands allow to construct any arbitrary iptables rule, as they
pass their arguments as-is to the v4 and/or v6 result rulesets. This means,
you have to use the exact same syntax for the arguments as for the iptables(8)
command. You'll hardly use them, as the whole purpose of the DSL is to make
life easier than hardcoding iptables rules.







`function` 
**`rule`**
`()`


> Generic rule for both v4 and v6 rulesets.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**iptables-args**</small> | **`in`** | Same argument syntax as for iptables(8) command arguments


> **Examples:**  
> `rule -A INPUT -i eth0 -j ACCEPT`  
  





`function` 
**`rule4`**
`()`


> Generic rule for v4 ruleset only.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**iptables-args**</small> | **`in`** | Same argument syntax as for iptables(8) command arguments







`function` 
**`rule6`**
`()`


> Generic rule for v6 ruleset only.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**iptables-args**</small> | **`in`** | Same argument syntax as for iptables(8) command arguments







`function` 
**`R`**
`()`


> Generic rule with fixed chain and optional jump target arguments, v4+v6. The jump target argument may be omitted, and defaults to `ACCEPT`. This will probbaly be the most-used DSL command in a `rules.template`, as for a typical firewall ruleset the default policy is `DROP`, and the rules then typically `ACCEPT` some specific traffic.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**chain**</small> | **`in`** | The chain this rule is appended to (`-A chain`)
> <small>**target**</small> | **`in`** | The jump target for this rule (`-j target`)
> <small>**iptables-args**</small> | **`in`** | Same argument syntax as for iptables(8) command arguments


> **Examples:**  
> `R INPUT ACCEPT -i lo`  
 same as: `R INPUT -i lo`<br> both translate to `-A INPUT -i lo -j ACCEPT` in the rulesets.  





`function` 
**`R4`**
`()`


> Generic rule with fixed chain and jump target arguments, v4 only.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**chain**</small> | **`in`** | The chain this rule is appended to (`-A chain`)
> <small>**target**</small> | **`in`** | The jump target for this rule (`-j target`)
> <small>**iptables-args**</small> | **`in`** | Same argument syntax as for iptables(8) command arguments







`function` 
**`R6`**
`()`


> Generic rule with fixed chain and jump target arguments, v6 only.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**chain**</small> | **`in`** | The chain this rule is appended to (`-A chain`)
> <small>**target**</small> | **`in`** | The jump target for this rule (`-j target`)
> <small>**iptables-args**</small> | **`in`** | Same argument syntax as for iptables(8) command arguments



## Target-focused DSL functions






`function` 
**`ACCEPT`**
`()`


> Accept rule, for both v4 and v6 rulesets.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**chain**</small> | **`in`** | The chain name of the chain the rule is appended to
> <small>**iptables-args**</small> | **`in`** | Same argument syntax as for iptables(8) command arguments







`function` 
**`ACCEPT4`**
`()`


> Accept rule, for v4 rulesets only.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**chain**</small> | **`in`** | The chain name of the chain the rule is appended to
> <small>**iptables-args**</small> | **`in`** | Same argument syntax as for iptables(8) command arguments







`function` 
**`ACCEPT6`**
`()`


> Accept rule, for v6 ruleset only.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**chain**</small> | **`in`** | The chain name of the chain the rule is appended to
> <small>**iptables-args**</small> | **`in`** | Same argument syntax as for iptables(8) command arguments







`function` 
**`ACC`**
`()`


> Same as `ACCEPT()` function.





`function` 
**`ACC4`**
`()`


> Same as `ACCEPT4()` function.





`function` 
**`ACC6`**
`()`


> Same as `ACCEPT6()` function.





`function` 
**`DROP`**
`()`


> Drop rule, for both v4 and v6 rulesets.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**chain**</small> | **`in`** | The chain name of the chain the rule is appended to
> <small>**iptables-args**</small> | **`in`** | Same argument syntax as for iptables(8) command arguments







`function` 
**`DROP4`**
`()`


> Drop rule, for v4 ruleset only.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**chain**</small> | **`in`** | The chain name of the chain the rule is appended to
> <small>**iptables-args**</small> | **`in`** | Same argument syntax as for iptables(8) command arguments







`function` 
**`DROP6`**
`()`


> Drop rule, for v6 ruleset only.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**chain**</small> | **`in`** | The chain name of the chain the rule is appended to
> <small>**iptables-args**</small> | **`in`** | Same argument syntax as for iptables(8) command arguments



## Chain-focused DSL functions






`function` 
**`INPUT`**
`()`


> Input chain rule, for both v4 and v6 rulesets.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**Optional**</small> | **`in`** | jump target chain, defaults to `ACCEPT`
> <small>**iptables-args**</small> | **`in`** | Same argument syntax as for iptables(8) command arguments







`function` 
**`IN`**
`()`


> Same as `INPUT()` function.





`function` 
**`IN4`**
`()`


> Input chain rule, for v4 ruleset only.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**Optional**</small> | **`in`** | jump target chain, defaults to `ACCEPT`
> <small>**iptables-args**</small> | **`in`** | Same argument syntax as for iptables(8) command arguments







`function` 
**`IN6`**
`()`


> Input chain rule, for v6 ruleset only.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**Optional**</small> | **`in`** | jump target chain, defaults to `ACCEPT`
> <small>**iptables-args**</small> | **`in`** | Same argument syntax as for iptables(8) command arguments







`function` 
**`FORWARD`**
`()`


> Forward chain rule, for both v4 and v6 rulesets.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**Optional**</small> | **`in`** | jump target chain, defaults to `ACCEPT`
> <small>**iptables-args**</small> | **`in`** | Same argument syntax as for iptables(8) command arguments







`function` 
**`FWD`**
`()`


> Same as `FORWARD()` function





`function` 
**`FWD4`**
`()`


> Forward chain rule, for v4 ruleset only.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**Optional**</small> | **`in`** | jump target chain, defaults to `ACCEPT`
> <small>**iptables-args**</small> | **`in`** | Same argument syntax as for iptables(8) command arguments







`function` 
**`FWD6`**
`()`


> Forward chain rule, for v6 ruleset only.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**Optional**</small> | **`in`** | jump target chain, defaults to `ACCEPT`
> <small>**iptables-args**</small> | **`in`** | Same argument syntax as for iptables(8) command arguments







`function` 
**`OUTPUT`**
`()`


> Output chain rule, for both v4 and v6 rulesets.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**Optional**</small> | **`in`** | jump target chain, defaults to `ACCEPT`
> <small>**iptables-args**</small> | **`in`** | Same argument syntax as for iptables(8) command arguments







`function` 
**`OUT`**
`()`


> Same as `OUTPUT()` function.





`function` 
**`OUT4`**
`()`


> Output chain rule, for v4 ruleset only.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**Optional**</small> | **`in`** | jump target chain, defaults to `ACCEPT`
> <small>**iptables-args**</small> | **`in`** | Same argument syntax as for iptables(8) command arguments







`function` 
**`OUT6`**
`()`


> Output chain rule, for v4 ruleset only.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**Optional**</small> | **`in`** | jump target chain, defaults to `ACCEPT`
> <small>**iptables-args**</small> | **`in`** | Same argument syntax as for iptables(8) command arguments







`function` 
**`DOCKER`**
`()`


> Docker chain rule, for both v4 and v6 rulesets.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**Optional**</small> | **`in`** | jump target chain, defaults to `ACCEPT`
> <small>**iptables-args**</small> | **`in`** | Same argument syntax as for iptables(8) command arguments







`function` 
**`DOCKER_USER`**
`()`


> Docker-User chain rule, for both v4 and v6 rulesets.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**Optional**</small> | **`in`** | jump target chain, defaults to `ACCEPT`
> <small>**iptables-args**</small> | **`in`** | Same argument syntax as for iptables(8) command arguments







`function` 
**`PREROUTING`**
`()`


> Prerouting chain rule, for both v4 and v6 rulesets.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**Optional**</small> | **`in`** | jump target chain, defaults to `ACCEPT`
> <small>**iptables-args**</small> | **`in`** | Same argument syntax as for iptables(8) command arguments







`function` 
**`POSTROUTING`**
`()`


> Postrouting chain rule, for both v4 and v6 rulesets.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**Optional**</small> | **`in`** | jump target chain, defaults to `ACCEPT`
> <small>**iptables-args**</small> | **`in`** | Same argument syntax as for iptables(8) command arguments



## Protocol-specific DSL functions


### ICMP input DSL functions






`function` 
**`ICMP`**
`()`


> ICMP type function, for both v4 and v6 rulesets.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**icmp-type**</small> | **`in`** | ICMP accept type this rule applies to, e.g. `echo-request`
> <small>**iptables-args**</small> | **`in`** | Same argument syntax as for iptables(8) command arguments


> **Examples:**  
> `ICMP echo-request 'LIMIT 1 5'`  
 _Defines a rate-limited rule to allow pings. See below for the `LIMIT` in-rule helper function._ <br> `ICMP DROP echo-request` <br> _Denies ICMP echo requests._  





`function` 
**`ICMP4`**
`()`


> ICMP type function, for v4 ruleset only.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**icmp-type**</small> | **`in`** | ICMP accept type this rule applies to, e.g. `echo-request`
> <small>**iptables-args**</small> | **`in`** | Same argument syntax as for iptables(8) command arguments







`function` 
**`ICMP6`**
`()`


> ICMP type function, for v6 ruleset only.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**icmp-type**</small> | **`in`** | ICMP accept type this rule applies to, e.g. `echo-request`
> <small>**iptables-args**</small> | **`in`** | Same argument syntax as for iptables(8) command arguments



### TCP input DSL functions






`function` 
**`TCP`**
`()`


> TCP traffic on a certain interface and specific port(s), both v4 and v6.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**target**</small> | **`in`** | Optional jump target chain, defaults to `ACCEPT`
> <small>**interface**</small> | **`in`** | The interface where TCP traffic should be handled
> <small>**port(s)**</small> | **`in`** | Port specification, single, multiple comma-separated, or port range


> **Examples:**  
> `TCP eth0 22`  
 _Allow SSH input on the `eth0` interface._ <br> `TCP DROP eth1 80,443` <br> _Deny HTTP(S) traffic on the `eth1` interface._ <br> `TCP enp4s0 0-1023` _Allow all well-known ports for TCP on the `enp4s0` interface._  





`function` 
**`TCP4`**
`()`


> TCP traffic on a certain interface and specific port(s), v4 only.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**target**</small> | **`in`** | Optional jump target chain, defaults to `ACCEPT`
> <small>**interface**</small> | **`in`** | The interface where TCP traffic should be handled
> <small>**port(s)**</small> | **`in`** | Port specification, single, multiple comma-separated, or port range







`function` 
**`TCP6`**
`()`


> TCP traffic on a certain interface and specific port(s), v6 only.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**target**</small> | **`in`** | Optional jump target chain, defaults to `ACCEPT`
> <small>**interface**</small> | **`in`** | The interface where TCP traffic should be handled
> <small>**port(s)**</small> | **`in`** | Port specification, single, multiple comma-separated, or port range



### UDP input DSL functions






`function` 
**`UDP`**
`()`


> UDP traffic on a certain interface and specific port(s), both v4 and v6.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**target**</small> | **`in`** | Optional jump target chain, defaults to `ACCEPT`
> <small>**interface**</small> | **`in`** | The interface where UDP traffic should be handled
> <small>**port(s)**</small> | **`in`** | Port specification, single, multiple comma-separated, or port range


> **Examples:**  
> `UDP eth0 53`  
 _Allow DNS input on the `eth0` interface._ <br> `UDP DROP eth1 137,138` <br> _Deny NETBIOS traffic on the `eth1` interface._ <br> `UDP enp4s0 0-1023` _Allow all well-known ports for UDP on the `enp4s0` interface._  





`function` 
**`UDP4`**
`()`


> UDP traffic on a certain interface and specific port(s), v4 only.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**target**</small> | **`in`** | Optional jump target chain, defaults to `ACCEPT`
> <small>**interface**</small> | **`in`** | The interface where UDP traffic should be handled
> <small>**port(s)**</small> | **`in`** | Port specification, single, multiple comma-separated, or port range







`function` 
**`UDP6`**
`()`


> UDP traffic on a certain interface and specific port(s), v6 only.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**target**</small> | **`in`** | Optional jump target chain, defaults to `ACCEPT`
> <small>**interface**</small> | **`in`** | The interface where UDP traffic should be handled
> <small>**port(s)**</small> | **`in`** | Port specification, single, multiple comma-separated, or port range



## In-Rule helper functions


These functions are not to be used as DSL rule commands; rather they are
designed to be used within a rule definition using the backtick \` or shell
command substitution `$(...)` syntax, as they expand to a list of arguments
for iptables(8). These helper functions make it easier to specify features
like ICMP types, conntrack state information, rate limiting, or TCP flags
within a rule.







`function` 
**`ICMPTYPE`**
`()`


> Inserts ICMP protocol and ICMP type within a rule.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**icmp-type**</small> | **`in`** | The icmp-type to include


> **Examples:**  
> ``IN4 `ICMPTYPE echo-request` ``  
 Inserts the expansion `-p icmp --icmp-type echo-request` within the rule, allowing ICMPv4 echo requests on input.  





`function` 
**`ICMP6TYPE`**
`()`


> Inserts ICMPv6 protocol and ICMPv6 type within a rule.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**icmp-type**</small> | **`in`** | the icmp-type to include


> **Examples:**  
> ``IN6 `ICMP6TYPE echo-request` ``  
 Inserts the expansion `-p icmpv6 --icmpv6-type echo-request` within the rule, allowing ICMPv6 echo requests on input.  





`function` 
**`STATE`**
`()`


> Inserts conntrack state matching within a rule.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**ctstate**</small> | **`in`** | The conntrack states to be included


> **Examples:**  
> ``IN `STATE ESTABLISHED,RELATED` ``  
 Accepts already established connections connections on input.  





`function` 
**`ESTABLISHED`**
`()`


> Inserts conntrack `ESTABLISHED,RELATED` information in the rule. This is a shortcut for the `STATE ESTABLISHED,RELATED` example above.
> **Examples:**  
> ``IN `ESTABLISHED` `` -- same as above example.  
  





`function` 
**`LIMIT`**
`()`


> Inserts rate limiting within the rule

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**limit**</small> | **`in`** | The packets per second limit to include
> <small>**burst**</small> | **`in`** | The burst packet limit to include


> **Examples:**  
> ``ICMP echo-request `LIMIT 1 5` ``  
 This allows rate-limited ping requests, with a limit of 1 packet per second, and a burst rate of 5 packets per second. This is the default btw, so you may use just ``ICMP echo-request `LIMIT` ``, which has the same effect.  





`function` 
**`TCPFLAGS`**
`()`


> Insert TCP flag information within the rule

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**check**</small> | **`in`** | The set of flags to check against
> <small>**flags**</small> | **`in`** | The flags that must be set for the expression to match


> **Examples:**  
> ``TCP eth0 22 `TCPFLAGS SYN,RST SYN` ``  
 Allows new SSH connections on input.  





`function` 
**`MATCHSET`**
`()`


> Matches a named `ipset` IP set for eitcher source (default) or destination address.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**ipset-name**</small> | **`in`** | Name of the IP set to match against
> <small>**src-dst**</small> | **`in`** | Match against source (`src`, default) or destination (`dst`) addresses







`function` 
**`TARGET`**
`()`


> Target-specific helpers for MSS clamping, or packet marking.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**tgt**</small> | **`in`** | Target specification: `TCPMSS`, or `MARK nnn`


> **Examples:**  
> `` POSTROUTING "`TARGET TCPMSS`" -o "$EXT_IF" `TCPFLAGS SYN,RST SYN` ``  
 Introduces MSS clamping for the `TCPMSS` target for new connections on the external interface (using the `TCPFLAGS` in-rule helper). This results in the rule `-A POSTROUTING -o eth1 -p tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu` appended to the `/etc/iptables/rules.v4` file. <br> `` R4 PREROUTING "`TARGET MARK 100`" `MATCHSET "$GEOIP_SET" dst` `` <br> This marks packets that match against a certain IP set with mark 100 during pre-routing.  

## Shortcuts for blocklists, NAT and port forwarding






`function` 
**`BLOCKLIST4`**
`()`


> Blocks IPv4 source addresses on input and forward that match against a certain IP set.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**ipset**</small> | **`in`** | Name of the IP set to match against


> **Examples:**  
> `BLOCKLIST4 "spamhaus_drop"`  
 Drop packets from IPv4 addresses contained in the Spamhaus DROP list.  





`function` 
**`BLOCKLIST6`**
`()`


> Blocks IPv6 packets on input and forward that match against a certain IP set.

> <small>*Arg.*</small> | <small>*Dir.*</small> | <small>*Description*</small>
> :---------:|:---|:------------------------------------------
> <small>**ipset**</small> | **`in`** | Name of the IP set to match against


> **Examples:**  
> `BLOCKLIST6 "spamhaus_drop_v6"`  
 Drop packets from IPv6 addresses contained in the Spamhaus IPv6 DROP list.  





`function` 
**`MASQUERADE`**
`()`


> Establishes masquerading for IPv4 source addresses during post-routing. <br> **Note**: This DSL function has to be used within the `TABLE nat` section!
> **Examples:**  
> `MASQUERADE -o "$EXT_IF"`  
 Establish source address masquerading on outgoing packets on the external interface.  





`function` 
**`DNAT`**
`()`


> Establishes destination NAT (DNAT) for IPv4 destination addresses during pre-routing. <br> **Note**: This DSL function has to be used within the `TABLE nat` section!
> **Examples:**  
> `DNAT -o "$EXT_IF"` ! -s "$INTRANET"  
 Establish destination NAT on outgoing packets, but not for internal Intranet addresses.  








