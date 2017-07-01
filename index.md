# cbrownstein@liquidityllc.com
PGP Key Fingerprint:
[D694 9CC2 320A B676 B914 99DB 9BB2 5A1A 5C1D 9108](cbrownstein.asc)

## YubiKey SSH Authentication on Ubuntu GNOME 17.04 (Zesty Zapus)
These instructions have been tested to work on Ubuntu GNOME 17.04 (Zesty
Zapus). They may be valid for other flavors and versions of Ubuntu (and
possibly other Linux distributions). If they are,
[please let me know](mailto:cbrownstein@liquidityllc.com) so that I may update
these instructions and give you credit for your contribution.

These instructions assume that you already have your YubiKey working with
`gpg` and that an authentication subkey is already loaded onto your YubiKey.
If the output of `gpg --list-keys [your key ID]` has a line like the one
below, you have an authentication subkey on your YubiKey.

```
sub   rsa4096 2017-03-04 [A] [expires: 2017-12-31]
```

First, you have to export a public RSA key that `ssh` understands. To do this,
run `gpg --export-ssh-key [your key ID] > id_rsa_YubiKey.pub` from a terminal.
This command creates a file named `id_rsa_YubiKey.pub` that contains the SSH
public key for your YubiKey authentication subkey. The contents of this file
should be appended to the `~/.ssh/authorized_keys` file on the server you want
to connect to.

Ubuntu GNOME 17.04 starts `ssh-agent` right when you login. Because of this,
`ssh` uses `ssh-agent` by default. The authentication subkey on your YubiKey
cannot be used by `ssh-agent` since `ssh-agent` does not have the ability to
read your YubiKey. For that reason, you need to configure `ssh` to use
`gpg-agent` instead of `ssh-agent` as it does by default. To make `ssh` use
`gpg-agent` you have to add the line `enable-ssh-support` to your
`~/.gnupg/gpg-agent.conf` file. If this file does not exist, create it with
that line. This setting allows `gpg-agent` to be used in place of `ssh-agent`
and allows `ssh` to read your YubiKey.

At this point, `gpg-agent` can now function in place of `ssh-agent` for SSH
authentication. But, `ssh` does not yet know that. Add the following lines to your `~/.bashrc` so that `ssh` knows to use `gpg-agent` for SSH authentication:

```shell
# start gpg-agent if it is not running already
gpg-connect-agent /bye

# use gpg-agent instead of ssh-agent
unset SSH_AGENT_PID
if [ "${gnupg_SSH_AUTH_SOCK_by:-0}" -ne $$ ]; then
  export SSH_AUTH_SOCK="$(gpgconf --list-dirs agent-ssh-socket)"
fi
```

With these lines added to your `~/.bashrc` file, `ssh` will use `gpg-agent`
which can access your YubiKey.

Open a new terminal (to make sure your updated `~/.bashrc` file is read) and
run `gpgconf --reload gpg-agent` from this new terminal. This command makes
`gpg-agent` reload your `~/.gnupg/gpg-agent.conf` file just in case
`gpg-agent` does not already know

If your YubiKey is not already plugged in, plug it in now. Run `ssh-add -L`
from your new terminal. You should see the contents of the
`id_rsa_YubiKey.pub` file that you created earlier. If you do, you are now
ready to connect to a server.

Connect to the server like you normally would, e.g., ssh 192.168.0.1. You
should be prompted to enter your PIN. And once you do, you should be connected
to the server.

Congratulations! Your YubiKey is now configured for authentication.

If these instructions do not work for you, please
[email me](mailto:cbrownstein@liquidityllc.com) so that hopefully I can help
you. (Encrypted email is preferable.)

-- Cody Brownstein
