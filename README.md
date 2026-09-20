# msgwing.postfix_relay

Configure Postfix as a satellite/smarthost relay through
[ZeroSMTP](https://github.com/msgwing/ZeroSMTP) (`mx.msgwing.com`) — a free
SMTP relay for printers, scanners, NAS units and legacy apps that can't do
OAuth2, built for the December 2026 shutdown of Basic Auth for SMTP AUTH in
Exchange Online.

Only a ZeroSMTP username and password are required. The role installs
Postfix, sets satellite mode, configures SASL auth, and rewrites the
envelope/From sender — without that last step, system mail (`cron`,
`unattended-upgrades`, `systemd` failure notices) leaves as `root@yourhost`
and the relay refuses it with `553 5.7.1 ... not owned by user`.

This mirrors the same setup documented at
[docs.msgwing.com/SYSTEM-MTA.html](https://docs.msgwing.com/SYSTEM-MTA.html)
and the standalone
[`setup-postfix-relay.sh`](https://github.com/msgwing/ZeroSMTP/blob/main/setup-postfix-relay.sh)
script in that repository, as an Ansible role for rolling it out to more
than a handful of servers.

## Requirements

- Debian (bullseye/bookworm) or Ubuntu (focal/jammy/noble)
- A free ZeroSMTP account — register and activate at
  [msgwing.com](https://msgwing.com), then copy the generated
  `you@msgwing.com` login and password (shown once)

## Role Variables

See [`defaults/main.yml`](defaults/main.yml):

| Variable | Default | Meaning |
|---|---|---|
| `zerosmtp_username` | *(required)* | Your `you@msgwing.com` login |
| `zerosmtp_password` | *(required)* | Your ZeroSMTP password |
| `zerosmtp_port` | `587` | `587` (STARTTLS) or `465` (implicit TLS) |
| `zerosmtp_relay_host` | `mx.msgwing.com` | The relay's hostname |
| `zerosmtp_test_recipient` | `""` | If set, sends a test message here and confirms the sender rewrite in `/var/log/mail.log` |

Two limits come with a free ZeroSMTP account, stated here rather than
discovered later: mail leaves from the generated `@msgwing.com` address
rather than your own domain, and the cap is 200 messages a day with no paid
tier that lifts it. If either rules this out, see
[the alternatives page](https://docs.msgwing.com/ALTERNATIVES.html).

## Example Playbook

```yaml
- hosts: all
  become: true
  roles:
    - role: msgwing.postfix_relay
      vars:
        zerosmtp_username: "you@msgwing.com"
        # Keep the real password in ansible-vault or your inventory's
        # secrets file - never commit it in plain text next to the playbook.
        zerosmtp_password: "{{ vault_zerosmtp_password }}"
        zerosmtp_test_recipient: "you@example.com"
```

Run with `ansible-playbook -i inventory.ini site.yml --ask-vault-pass` (or
however your vault secret is normally supplied). Safe to re-run.

## License

MIT

## Author

[msgwing](https://github.com/msgwing) —
[ZeroSMTP](https://github.com/msgwing/ZeroSMTP)
