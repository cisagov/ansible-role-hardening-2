# Hardening - the Ansible role

An [Ansible](https://www.ansible.com/) role to make a AlmaLinux, Debian, or
Ubuntu server a bit more secure.
[systemd edition](https://freedesktop.org/wiki/Software/systemd/).

Requires Ansible >= 2.13.

Available on
[Ansible Galaxy](https://galaxy.ansible.com/konstruktoid/hardening).

[AlmaLinux 8](https://wiki.almalinux.org/release-notes/#almalinux-8),
[AlmaLinux 9](https://wiki.almalinux.org/release-notes/#almalinux-9),
[Debian 11](https://www.debian.org/releases/bullseye/),
[Ubuntu 20.04 LTS (Focal Fossa)](https://releases.ubuntu.com/focal/) and
[Ubuntu 22.04 LTS (Jammy Jellyfish)](https://releases.ubuntu.com/jammy/) are supported.

> **Note**
>
> Do not use this role without first testing in a non-operational environment.

> **Note**
>
> There is a [SLSA](https://slsa.dev/) artifact present under the
> [slsa action workflow](https://github.com/konstruktoid/ansible-role-hardening/actions/workflows/slsa.yml)
> for verification.

## Dependencies

None.

## Note regarding UFW firewall rules

Instead of resetting `ufw` every run and by doing so causing network traffic
disruption, the role deletes every `ufw` rule without
`comment: ansible managed` task parameter and value.

The role also sets default deny policies, which means that firewall rules
needs to be created for any additional ports except those specified in
the `sshd_port` and `ufw_outgoing_traffic` variables.

## Task Execution and Structure

See [STRUCTURE.md](STRUCTURE.md) for tree of the role structure.

## Role testing

See [TESTING.md](TESTING.md).

## Role Variables with defaults

### ./defaults/main/aide.yml

```yaml
install_aide: true
aide_checksums: sha512
```

If `install_aide: true` then [AIDE](https://aide.github.io/) will be installed
and configured.

`aide_checksums` modifies the AIDE `Checksums` variable. Note that the
`Checksums` variable might not be present depending on distribution.

[aide.conf(5)](https://linux.die.net/man/5/aide.conf)

### ./defaults/main/auditd.yml

```yaml
auditd_apply_audit_rules: true
auditd_action_mail_acct: root
auditd_admin_space_left_action: suspend
auditd_disk_error_action: suspend
auditd_disk_full_action: suspend
auditd_flush: sync
auditd_max_log_file: 20
auditd_max_log_file_action: rotate
auditd_mode: 1
auditd_num_logs: 5
auditd_space_left: 75
auditd_space_left_action: email
grub_audit_backlog_cmdline: audit_backlog_limit=8192
grub_audit_cmdline: audit=1
```

Enable `auditd` at boot using Grub.

When `auditd_apply_audit_rules: 'yes'`, the role applies the auditd rules
from the included template file.

`auditd_action_mail_acct` should be a valid email address or alias.

`auditd_admin_space_left_action` defines what action to take when the system has
detected that it is low on disk space. `suspend` will cause the audit daemon to
stop writing records to the disk.

`auditd_flush: sync` tells the audit daemon to keep both the data and meta-data
fully sync'd with every write to disk.

`auditd_max_log_file_action` sets what action to take when the system has
detected that the max file size limit has been reached. E.g. the `rotate` option
will cause the audit daemon to rotate the logs. The `keep_logs` option is
similar to `rotate` except it does not use the `num_logs` setting. This prevents
audit logs from being overwritten.

`auditd_space_left_action` tells the system what action to take when the system
has detected that it is low on disk space. `email` means that it will send a
warning to the email account specified in `action_mail_acct` as well as
sending the message to syslog.

`auditd_mode` sets `auditd` failure mode, 0=silent 1=printk 2=panic.

[auditd.conf(5)](https://man7.org/linux/man-pages/man5/auditd.conf.5.html)

### ./defaults/main/compilers.yml

```yaml
compilers:
  - as
  - cargo
  - cc
  - cc-[0-9]*
  - clang-[0-9]*
  - gcc
  - gcc-[0-9]*
  - go
  - make
  - rustc
```

List of compilers that will be restricted to the root user.

### ./defaults/main/crypto_policies.yml

```yaml
set_crypto_policy: true
crypto_policy: DEFAULT:NO-SHA1
```

Set and use [cryptographic policies](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/security_hardening/using-the-system-wide-cryptographic-policies_security-hardening)
if `/etc/crypto-policies/config` exists and `set_crypto_policy: true`.

### ./defaults/main/disablewireless.yml

```yaml
disable_wireless: false
```

If `true`, turn off all wireless interfaces.

### ./defaults/main/dns.yml

```yaml
dns: 127.0.0.1 1.1.1.1
fallback_dns: 9.9.9.9 1.0.0.1
dnssec: allow-downgrade
dns_over_tls: opportunistic
```

IPv4 and IPv6 addresses to use as system and fallback DNS servers.
If `dnssec` is set to "allow-downgrade" DNSSEC validation is attempted, but if
the server does not support DNSSEC properly, DNSSEC mode is automatically
disabled.

If `dns_over_tls` is true, all connections to the server will be encrypted if
the DNS server supports DNS-over-TLS and has a valid certificate.

[systemd](https://github.com/konstruktoid/hardening/blob/master/systemd.adoc#etcsystemdresolvedconf)
option.

### ./defaults/main/ipv6.yml

```yaml
disable_ipv6: false
ipv6_disable_sysctl_settings:
  net.ipv6.conf.all.disable_ipv6: 1
  net.ipv6.conf.default.disable_ipv6: 1
```

If `disable_ipv6: true`, IPv6 will be disabled.

### ./defaults/main/limits.yml

```yaml
limit_nofile_hard: 1024
limit_nofile_soft: 512
limit_nproc_hard: 1024
limit_nproc_soft: 512
```

Maximum number of processes and open files.

### ./defaults/main/misc.yml

```yaml
reboot_ubuntu: false
redhat_signing_keys:
  - 567E347AD0044ADE55BA8A5F199E2F91FD431D51
  - 47DB287789B21722B6D95DDE5326810137017186
epel7_signing_keys:
  - 91E97D7C4A5E96F17F3E888F6A2FAEA2352C64E5
epel8_signing_keys:
  - 94E279EB8D8F25B21810ADF121EA45AB2F86D6A1
epel9_signing_keys:
  - FF8AD1344597106ECE813B918A3872BF3228467C
```

If `reboot_ubuntu: true` an Ubuntu node will be rebooted if required.

`redhat_signing_keys` are [RedHat Product Signing Keys](https://access.redhat.com/security/team/key/).

The `epel7_signing_keys`, `epel8_signing_keys` and
`epel9_signing_keys` are release specific
[Fedora EPEL signing keys](https://getfedora.org/security/).

### ./defaults/main/module_blocklists.yml

```yaml
block_blacklisted: false
fs_modules_blocklist:
  - cramfs
  - freevxfs
  - hfs
  - hfsplus
  - jffs2
  - squashfs
  - udf
misc_modules_blocklist:
  - bluetooth
  - bnep
  - btusb
  - can
  - cpia2
  - firewire-core
  - floppy
  - ksmbd
  - n_hdlc
  - net-pf-31
  - pcspkr
  - soundcore
  - thunderbolt
  - usb-midi
  - usb-storage
  - uvcvideo
  - v4l2_common
net_modules_blocklist:
  - atm
  - dccp
  - sctp
  - rds
  - tipc
```

Blocked kernel modules.

Setting `block_blacklisted: true` will block, or disable, any automatic loading
of `blacklisted` kernel modules. The reasoning behind this is that a blacklisted
module can still be loaded manually with `modprobe module_name`. Using
`install module_name /bin/true` prevents this.

This will affect modules that the distribution has blacklisted as
part of its default setup, or that were added manually at some point, by anyone
with access to your system. Please verify the affected modules before turning
this on by running `modprobe --showconfig | grep '^blacklist'`

> **Note**
>
> Disabling the `usb-storage` module will disable all USB
> storage devices. If such devices are needed [USBGuard](https://github.com/USBGuard/usbguard),
> or a similar tool, should be configured accordingly.

### ./defaults/main/mount.yml

```yaml
hide_pid: 2
process_group: root
```

`hide_pid` sets `/proc/<pid>/` access mode.

The `process_group` setting configures the group authorized to learn processes
information otherwise prohibited by `hidepid=`.

[/proc mount options](https://www.kernel.org/doc/html/latest/filesystems/proc.html#mount-options)

### ./defaults/main/ntp.yml

```yaml
enable_timesyncd: true
fallback_ntp: 2.ubuntu.pool.ntp.org 3.ubuntu.pool.ntp.org
ntp: "0.ubuntu.pool.ntp.org 1.ubuntu.pool.ntp.org"
aws_fallback_ntp: "169.254.169.123"
aws_ntp: "169.254.169.123"
```

If `enable_timesyncd: true` then configure systemd
[timesyncd](https://manpages.ubuntu.com/manpages/jammy/man8/systemd-timesyncd.service.8.html),
otherwise installing a NTP client is recommended.

### ./defaults/main/packages.yml

```yaml
system_upgrade: true
packages_blocklist:
  - apport*
# This package is required for freeipa-server on Fedora.
#   - autofs
# These packages are required for freeipa-server on Fedora.
#   - avahi*
#   - avahi-*
  - beep
  - git
  - pastebinit
  - popularity-contest
  - prelink
# This package is required for freeipa-server on Fedora.
#  - rpcbind
  - rsh*
  - rsync
  - talk*
  - telnet*
  - tftp*
  - tuned
  - whoopsie
  - xinetd
  - yp-tools
  - ypbind
packages_debian:
  # We are only installing packages needed to support the tasks being run.
  # - acct
  # - apparmor-profiles
  # - apparmor-utils
  # - apt-show-versions
  # - audispd-plugins
  - auditd
  - cracklib-runtime
  # - debsums
  # - gnupg2
  # - haveged
  # - libpam-apparmor
  # - libpam-cap
  # - libpam-modules
  # We are only installing this package as it provides /etc/security/pwquality.conf
  - libpam-pwquality
  # - libpam-tmpdir
  # - lsb-release
  # - needrestart
  # - openssh-server
  # - postfix
  # - rkhunter
  # - rsyslog
  # - sysstat
  # - systemd-journal-remote
  # - tcpd
  # - vlock
  # - wamerican
packages_redhat:
  # We are only installing packages needed to support the tasks being run.
  # - audispd-plugins
  - audit
  - cracklib
  # - gnupg2
  # - haveged
  - libpwquality
  # - openssh-server
  # - needrestart
  # - postfix
  # - psacct
  - python3-dnf-plugin-post-transaction-actions
  # - rkhunter
  # - rsyslog
  # - rsyslog-gnutls
  # - systemd-journal-remote
  # - vlock
  # - words
packages_ubuntu: []
  # We are only installing packages needed to support the tasks being run.
  # - fwupd
  # - secureboot-db
  # - snapd
```

`system_upgrade: 'yes'` will run `apt upgrade` or
`dnf update` if required.

Packages to be installed depending of distribution
and packages to be removed (`packages_blocklist`).

### ./defaults/main/password.yml

```yaml
crypto_policy: "DEFAULT:NO-SHA1"
pwquality_config:
  dcredit: -1
  dictcheck: 1
  difok: 8
  enforcing: 1
  lcredit: -1
  maxclassrepeat: 4
  maxrepeat: 3
  minclass: 4
  minlen: 15
  ocredit: -1
  ucredit: -1
```

Configure the [libpwquality](https://manpages.ubuntu.com/manpages/focal/man5/pwquality.conf.5.html)
library.

### ./defaults/main/sshd.yml

```yaml
sshd_accept_env: LANG LC_*
sshd_admin_net:
  - 0.0.0.0/0
sshd_allow_agent_forwarding: "no"
sshd_allow_groups: sudo
sshd_allow_users: "{{ ansible_user }}"
sshd_allow_tcp_forwarding: "no"
sshd_authentication_methods: any
sshd_banner: /etc/issue.net
sshd_ca_signature_algorithms: >-
  ecdsa-sha2-nistp256,
  sk-ecdsa-sha2-nistp256@openssh.com,
  ecdsa-sha2-nistp384,
  ecdsa-sha2-nistp521,
  ssh-ed25519,
  sk-ssh-ed25519@openssh.com,
  rsa-sha2-256,
  rsa-sha2-512
sshd_challenge_response_authentication: "no"
sshd_ciphers: >-
  chacha20-poly1305@openssh.com,
  aes256-gcm@openssh.com,
  aes128-gcm@openssh.com,
  aes256-ctr,
  aes192-ctr,
  aes128-ctr
sshd_client_alive_count_max: 3
sshd_client_alive_interval: 15
sshd_compression: "no"
sshd_gssapi_authentication: "no"
sshd_gssapi_kex_algorithms: >-
  gss-curve25519-sha256-,
  gss-nistp256-sha256-,
  gss-group14-sha256-,
  gss-group16-sha512-
sshd_hostbased_authentication: "no"
sshd_host_key_algorithms: >-
  ssh-ed25519-cert-v01@openssh.com,
  ssh-rsa-cert-v01@openssh.com,
  ssh-ed25519,
  ssh-rsa,
  ecdsa-sha2-nistp521-cert-v01@openssh.com,
  ecdsa-sha2-nistp384-cert-v01@openssh.com,
  ecdsa-sha2-nistp256-cert-v01@openssh.com,
  ecdsa-sha2-nistp521,
  ecdsa-sha2-nistp384,
  ecdsa-sha2-nistp256
sshd_ignore_rhosts: "yes"
sshd_ignore_user_known_hosts: "yes"
sshd_kerberos_authentication: "no"
sshd_kex_algorithms: >-
  curve25519-sha256,
  curve25519-sha256@libssh.org,
  diffie-hellman-group14-sha256,
  diffie-hellman-group16-sha512,
  diffie-hellman-group18-sha512,
  ecdh-sha2-nistp521,
  ecdh-sha2-nistp384,
  ecdh-sha2-nistp256,
  diffie-hellman-group-exchange-sha256
sshd_login_grace_time: 20
sshd_log_level: VERBOSE
sshd_macs: >-
  hmac-sha2-512-etm@openssh.com,
  hmac-sha2-256-etm@openssh.com,
  hmac-sha2-512,
  hmac-sha2-256
sshd_max_auth_tries: 3
sshd_max_sessions: 4
sshd_max_startups: 10:30:60
sshd_password_authentication: "no"
sshd_permit_empty_passwords: "no"
sshd_permit_root_login: "no"
sshd_permit_user_environment: "no"
sshd_port: 22
sshd_print_last_log: "yes"
sshd_print_motd: "no"
sshd_pubkey_accepted_algorithms: >-
  ecdsa-sha2-nistp256,
  ecdsa-sha2-nistp256-cert-v01@openssh.com,
  sk-ecdsa-sha2-nistp256@openssh.com,
  sk-ecdsa-sha2-nistp256-cert-v01@openssh.com,
  ecdsa-sha2-nistp384,
  ecdsa-sha2-nistp384-cert-v01@openssh.com,
  ecdsa-sha2-nistp521,
  ecdsa-sha2-nistp521-cert-v01@openssh.com,
  ssh-ed25519,
  ssh-ed25519-cert-v01@openssh.com,
  sk-ssh-ed25519@openssh.com,
  sk-ssh-ed25519-cert-v01@openssh.com,
  rsa-sha2-256,
  rsa-sha2-256-cert-v01@openssh.com,
  rsa-sha2-512,
  rsa-sha2-512-cert-v01@openssh.com
sshd_rekey_limit: 512M 1h
sshd_required_rsa_size: 2048
sshd_strict_modes: "yes"
sshd_subsystem: sftp internal-sftp
sshd_tcp_keep_alive: "no"
sshd_use_dns: "no"
sshd_use_pam: "yes"
sshd_x11_forwarding: "no"
```

> **Note**
>
> `CASignatureAlgorithms`, `Ciphers`, `HostKeyAlgorithms`, `KexAlgorithms` and `MACs`
> will be configured as defined by cryptographic policies if
> `/etc/crypto-policies/config` exists and `set_crypto_policy: true`.

For a explanation of the options not described below, please read
[https://man.openbsd.org/sshd_config](https://man.openbsd.org/sshd_config).

Only the network(s) defined in `sshd_admin_net` are allowed to
connect to `sshd_port`. Note that additional rules need to be set up in order
to allow access to additional services.

OpenSSH login is allowed only for users whose primary group or supplementary
group list matches one of the patterns in `sshd_allow_groups`.
OpenSSH login is also allowed for users in `sshd_allow_users`.

`sshd_allow_agent_forwarding` specifies whether ssh-agent(1) forwarding is
permitted.

`sshd_allow_tcp_forwarding` specifies whether TCP forwarding is permitted.
The available options are `yes` or all to allow TCP forwarding, `no` to prevent
all TCP forwarding, `local` to allow local (from the perspective of ssh(1))
forwarding only or `remote` to allow remote forwarding only.

`sshd_authentication_methods` specifies the authentication methods that must
be successfully completed in order to grant access to a user.

`sshd_log_level` gives the verbosity level that is used when logging messages.

`sshd_max_auth_tries` and `sshd_max_sessions` specifies the maximum number of
SSH authentication attempts permitted per connection and the maximum number of
open shell, login or subsystem (e.g. sftp) sessions permitted per network
connection.

`sshd_password_authentication` specifies whether password authentication is
allowed.

`sshd_port` specifies the port number that sshd(8) listens on.

`sshd_required_rsa_size`, RequiredRSASize, will only be set if SSH version is
higher than 9.1.

### ./defaults/main/suid_sgid_blocklist.yml

```yaml
suid_sgid_permissions: true
suid_sgid_blocklist:
  - 7z
  - ab
  - agetty
  - alpine
  - ansible-playbook
  - aoss
  - apt
  - apt-get
  [...]
```

If `suid_sgid_permissions: true` loop through `suid_sgid_blocklist` and remove
any SUID/SGID permissions.

A complete file list is available in
[defaults/main/suid_sgid_blocklist.yml](defaults/main/suid_sgid_blocklist.yml)
and is based on the work by [@GTFOBins](https://github.com/GTFOBins).

### ./defaults/main/sysctl.yml

```yaml
sysctl_dev_tty_ldisc_autoload: 0
sysctl_net_ipv6_conf_accept_ra_rtr_pref: 0

ipv4_sysctl_settings:
  net.ipv4.conf.all.accept_redirects: 0
  net.ipv4.conf.all.accept_source_route: 0
  net.ipv4.conf.all.log_martians: 1
  net.ipv4.conf.all.rp_filter: 1
  net.ipv4.conf.all.secure_redirects: 0
  net.ipv4.conf.all.send_redirects: 0
  net.ipv4.conf.all.shared_media: 0
  net.ipv4.conf.default.accept_redirects: 0
  net.ipv4.conf.default.accept_source_route: 0
  net.ipv4.conf.default.log_martians: 1
  net.ipv4.conf.default.rp_filter: 1
  net.ipv4.conf.default.secure_redirects: 0
  net.ipv4.conf.default.send_redirects: 0
  net.ipv4.conf.default.shared_media: 0
  net.ipv4.icmp_echo_ignore_broadcasts: 1
  net.ipv4.icmp_ignore_bogus_error_responses: 1
  net.ipv4.ip_forward: 0
  net.ipv4.tcp_challenge_ack_limit: 2147483647
  net.ipv4.tcp_invalid_ratelimit: 500
  net.ipv4.tcp_max_syn_backlog: 20480
  net.ipv4.tcp_rfc1337: 1
  net.ipv4.tcp_syn_retries: 5
  net.ipv4.tcp_synack_retries: 2
  net.ipv4.tcp_syncookies: 1

ipv6_sysctl_settings:
  net.ipv6.conf.all.accept_ra: 0
  net.ipv6.conf.all.accept_redirects: 0
  net.ipv6.conf.all.accept_source_route: 0
  net.ipv6.conf.all.forwarding: 0
  net.ipv6.conf.all.use_tempaddr: 2
  net.ipv6.conf.default.accept_ra: 0
  net.ipv6.conf.default.accept_ra_defrtr: 0
  net.ipv6.conf.default.accept_ra_pinfo: 0
  net.ipv6.conf.default.accept_ra_rtr_pref: 0
  net.ipv6.conf.default.accept_redirects: 0
  net.ipv6.conf.default.accept_source_route: 0
  net.ipv6.conf.default.autoconf: 0
  net.ipv6.conf.default.dad_transmits: 0
  net.ipv6.conf.default.max_addresses: 1
  net.ipv6.conf.default.router_solicitations: 0
  net.ipv6.conf.default.use_tempaddr: 2

generic_sysctl_settings:
  fs.protected_fifos: 2
  fs.protected_hardlinks: 1
  fs.protected_symlinks: 1
  fs.suid_dumpable: 0
  kernel.core_uses_pid: 1
  kernel.dmesg_restrict: 1
  kernel.kptr_restrict: 2
  kernel.panic: 60
  kernel.panic_on_oops: 60
  kernel.perf_event_paranoid: 3
  kernel.randomize_va_space: 2
  kernel.sysrq: 0
  kernel.unprivileged_bpf_disabled: 1
  kernel.yama.ptrace_scope: 2
  net.core.bpf_jit_harden: 2

conntrack_sysctl_settings:
  net.netfilter.nf_conntrack_max: 2000000
  net.netfilter.nf_conntrack_tcp_loose: 0
```

`sysctl` configuration.

[sysctl.conf](https://linux.die.net/man/5/sysctl.conf)

### ./defaults/main/templates.yml

```yaml
adduser_conf_template: etc/adduser.conf.j2
common_account_template: etc/pam.d/common-account.j2
common_auth_template: etc/pam.d/common-auth.j2
common_password_template: etc/pam.d/common-password.j2
coredump_conf_template: etc/systemd/coredump.conf.j2
hardening_rules_template: etc/audit/rules.d/hardening.rules.j2
hosts_allow_template: etc/hosts.allow.j2
hosts_deny_template: etc/hosts.deny.j2
initpath_sh_template: etc/profile.d/initpath.sh.j2
issue_template: etc/issue.j2
journald_conf_template: etc/systemd/journald.conf.j2
limits_conf_template: etc/security/limits.conf.j2
logind_conf_template: etc/systemd/logind.conf.j2
login_defs_template: etc/login.defs.j2
login_template: etc/pam.d/login.j2
logrotate_conf_template: etc/logrotate.conf.j2
motd_template: etc/motd.j2
resolved_conf_template: etc/systemd/resolved.conf.j2
rkhunter_template: etc/default/rkhunter.j2
ssh_config_template: etc/ssh/ssh_config.j2
sshd_config_template: etc/ssh/sshd_config.j2
system_conf_template: etc/systemd/system.conf.j2
timesyncd_conf_template: etc/systemd/timesyncd.conf.j2
useradd_template: etc/default/useradd.j2
user_conf_template: etc/systemd/user.conf.j2
```

Paths in order to support overriding the default [role templates](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_module.html).

### ./defaults/main/ufw.yml

```yaml
ufw_enable: true
ufw_outgoing_traffic:
  - 22
  - 53
  - 80
  - 123
  - 443
  - 853
```

`ufw_enable: true` installs and configures `ufw` with related rules. Set it to
`false` in order to install and configure a firewall manually.
`ufw_outgoing_traffic` opens the specific `ufw` ports,
allowing outgoing traffic.

### ./defaults/main/umask.yml

```yaml
umask_value: "027"
```

Set default [umask value](https://manpages.ubuntu.com/manpages/jammy/man2/umask.2.html).

### ./defaults/main/users.yml

```yaml
delete_users:
  - games
  - gnats
  - irc
  - list
  - news
  - sync
  - uucp
```

Users to be removed.

## Recommended Reading

[Comparing the DISA STIG and CIS Benchmark values](https://github.com/konstruktoid/publications/blob/master/ubuntu_comparing_guides_benchmarks.md)

[Center for Internet Security Linux Benchmarks](https://www.cisecurity.org/cis-benchmarks/)

[Common Configuration Enumeration](https://nvd.nist.gov/cce/index.cfm)

[DISA Security Technical Implementation Guides](https://public.cyber.mil/stigs/downloads/?_dl_facet_stigs=operating-systems%2Cunix-linux)

[SCAP Security Guides](https://static.open-scap.org/)

[Security focused systemd configuration](https://github.com/konstruktoid/hardening/blob/master/systemd.adoc)

## Contributing

Do you want to contribute? Great! Contributions are always welcome,
no matter how large or small. If you found something odd, feel free to submit a
issue, improve the code by creating a pull request, or by
[sponsoring this project](https://github.com/sponsors/konstruktoid).

## License

Apache License Version 2.0

## Author Information

[https://github.com/konstruktoid](https://github.com/konstruktoid "github.com/konstruktoid")
