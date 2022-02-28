# Linux ssh no matching key exchange method found

Created: February 1, 2022 5:22 PM
Tags: OS

OpenSSH options might behave somehow strange on the first sight. But manual page for `ssh_config` documents it well:

> For each parameter, the first obtained value will be used. The configuration files contain sections separated by “Host” specifications, and that section is only applied for hosts that match one of the patterns given in the specification. The matched host name is usually the one given on the command line (see the CanonicalizeHostname option for exceptions.)
> 

You might rewrite your config like this to achieve what you need (the star `*` match should be last):

```
Host 10.0.0.1
    KexAlgorithms diffie-hellman-group-exchange-sha256
#[...]
Host *
    KexAlgorithms curve25519-sha256@libssh.org

```

[From my duplicate answer](https://unix.stackexchange.com/a/274173/121504)

And to explain why the commandline option works, also from the same manual page for `ssh_config`:

> command-line optionsuser's configuration file (~/.ssh/config)system-wide configuration file (/etc/ssh/ssh_config)
> 

[https://unix.stackexchange.com/questions/274274/specifying-ssh-kexalgorithms-works-at-cli-but-not-via-ssh-config](https://unix.stackexchange.com/questions/274274/specifying-ssh-kexalgorithms-works-at-cli-but-not-via-ssh-config)