Get the agent installed and running:

```
git clone https://github.com/tillitis/tkey-ssh-agent.git
cd tkey-ssh-agent
sudo make install
sudo make reload-rules
systemctl --user enable --now tkey-ssh-agent
```

You could of course also run tkey-ssh-agent manually, instead of using
a systemd user unit. For example like: `tkey-ssh-agent --uss -a $HOME/tkey-agent-sock`.
Remember to adjust SSH_AUTH_SOCK everywhere to your desired location.

Now run this [./setup](setup) script in your repo. Upon loading the
device app, you should get a pinentry for inputting a USS, which is
mixed into the private key generated on the TKey. Making the private
key be based both on the unique TKey device, and something you know.
The script will configure your repos `.git/config`, including setting
`user.signingkey` to the TKey public key, given your USS. So you're
expected to use the same USS next time you commit to this repo.

The script also opportunistically looks up your `git config
user.email`, and adds you to a `allowed_signers` file, along with the
public key from the TKey (if your email is not already in that file).
Since `gpg.ssh.allowedSignersFile` is also set to this file, git can
verify signatures.

To make git use tkey-ssh-agent for signing, the `SSH_AUTH_SOCK`
environment variable has to point to its socket. You can try showing
the pubkey like this:

```
SSH_AUTH_SOCK=/run/user/$(id -u)/tkey-ssh-agent/sock ssh-add -L
```

Try committing, touching the TKey when it flashes green:

```
SSH_AUTH_SOCK=/run/user/$(id -u)/tkey-ssh-agent/sock git commit -S -m msg
git show
```

One could imagine various tricks to avoid having to set the
`SSH_AUTH_SOCK`. Some wrapper script? Something clever involving
`gpg.program` and `gpg.ssh.defaultKeyCommand` (see `man git-config`)?
