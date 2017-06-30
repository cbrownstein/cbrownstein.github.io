# cbrownstein@liquidityllc.com
PGP Key Fingerprint: D694 9CC2 320A B676 B914 99DB 9BB2 5A1A 5C1D 9108

## Configure Ubuntu GNOME 17.04 to allow SSH logins with a YubiKey
These instructions have been tested to work on Ubuntu GNOME 17.04. They may be
valid for other flavors and versions of Ubuntu (and other Linux
distributions). If they are, kindly let me know. Or even better, submit a pull
request.

These instructions assume you already have your YubiKey working and loaded
with an authentication subkey and that your YubiKey is working with gpg.

First, you have to export a public RSA key that can understood by SSH: an
id_rsa.pub file that can be added to authorized_keys. To do this, run `gpg
--export-ssh-key [your key ID] > id_rsa.pub` from a terminal. This command
will create a file named id_rsa.pub that can be added to an authorized_keys
file just like a regular SSH public key. It can also be added to anywhere else
an SSH public key can be added, e.g., GitHub.

Now is the somewhat difficult part. By default, Ubuntu GNOME 17.04 starts
ssh-agent right when your system boots. In fact, your entire session runs
under ssh-agent. Because of this, ssh uses ssh-agent by default. ssh-agent
does not know an authentication subkey is loaded onto your YubiKey. For that
reason, you need to configure ssh to use gpg-agent instead of ssh-agent. To do
that, you first have to add one line to your ~/.gnupg/gpg-agent.conf file. If
this file does not exist, create it. Add `enable-ssh-support` to the file.
This allows gpg-agent to be used in place of ssh-agent and allow SSH to use
PGP authentication subkeys for SSH.

At this point, gpg-agent can now function in place of ssh-agent. But, SSH does
not yet know that. You need to let it know. To do that, you will have to set
an environment variable so that ssh connects to gpg-agent instead of ssh-agent
as it currently does.

```shell
# use gpg-agent instead of ssh-agent
unset SSH_AGENT_PID
if [ "${gnupg_SSH_AUTH_SOCK_by:-0}" -ne $$ ]; then
  export SSH_AUTH_SOCK="$(gpgconf --list-dirs agent-ssh-socket)"
fi
```

With this added to your ~/.bashrc file, your terminal will automatically
set the SSH_AUTH_SOCK environment variable to gpg-agent instead of ssh-agent.
Accordingly, ssh will use gpg-agent instead of ssh-agent and now finally you
will be able to use the authentication subkey on your YubiKey.

Run `gpgconf --reload gpg-agent` from a terminal. This command makes gpg-agent
reload gpg-agent.conf. We want to make sure it know SSH support has been
enabled.

If your YubiKey is not already plugged in, plug it in now. Run ssh-add -L from
a terminal and you should see your PGP public key. You are now ready to
connect to a server that has the id_rsa.pub you created earlier added to
authorized_keys.

Connect to the server like you normally would, e.g., ssh 192.168.0.1. You
should be prompted to enter your PIN. And once you do, you should be connected
to the server.

Congratulations! Your YubiKey is now configured for authentication.

I want to improve these instructions to make them easier to follow. So, please
email your questions and comments to
[cbrownstein@liquidityllc.com](mailto:cbrownstein@liquidityllc.com).
(Encrypted email is preferable.)

-- Cody Brownstein
