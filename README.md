# This repo is just for advertisement please go to the original git.fok.systems/lyz/sshmask to get the files

sshmask is a python3 program to protect your service behind fwknop, pass, tor and proxychains. Typically used for ssh.

## Installation

Depending on what features you want to use you will need to install the required dependencies:

* the service [ ssh ]
* curl
* [[fwknop](http://www.cipherdyne.org/fwknop/)]
* [[pass](https://www.passwordstore.org/)]
* [[tor](https://www.torproject.org/),[torify](https://linux.die.net/man/1/torify)]
* [[proxychains](https://github.com/rofl0r/proxychains-ng)]

Clone the repo:

```bash
git clone https://git.fok.systems/lyz/sshmask
```

Go to the main sshmask directory and execute sshmask.py

```bash
cd sshmask
chmod +x sshmask
./sshmask
```

As it's going to be a command that you use every day (I hope ^^), I suggest to use `sshm` or `ssh` as alias.

## How it works

1. Load the contentes of `~/.config/sshmask/sshmaskrc` or the file specified by the `--rc_file` flag
2. Global and local service parse and configuration, including pass selection
3. If the service agent is selected, it starts it. If the service has passphrases protected under pass it loads it in the clipboard
4. Checks if the port is open, with a grep of a string after a netcat in the desired socket
5. If the port is open go to step 9
6. If not, it parses and configures fwknop
7. If fwknop passphrase is protected under pass it loads it in the clipboard
8. Launches fwknop
9. If the agent is not used and the passphrase of the service is protected under pass it loads it in the clipboard
10. Launch the service

Right now only ssh is supported as service, along with it's agent, ssh-agent. But the code and documentation will use service instead of ssh for future adoptions of other services.

You can also use sshmask as plain ssh because it passes through all the flags to the final ssh command.

### Dry run mode

If you want to open the port but don't execute the service this flag might come handy. As it will also copy the desired passwords.

### Proxychains

In case you need to use proxychains to pass the fwknop command, to configure ssh  to use proxychains look at the ProxyCommand config of sshrc.

### Tor

You can use ssh, fwknop or both over tor to an non .*onion address. I'm still debugging why fwknop to an .onion fails, ssh works.

## Usage

```bash
usage: usage: sshmask [-h] [-v] [-d] [--rc_file [RC_FILE]] [--use_fwknop]
               [--use_fwknop_over_tor] [--use_ssh_agent] [--use_proxychains]
               [--fwknop_stanza FWKNOP_STANZA]
               [--ssh_pass_stanza SSH_PASS_STANZA] [--fwknop_pass_stanza]
               [--interface INTERFACE] ssh_arguments [user@]hostname

Wrapper of ssh, ssh-agent, fwknop, tor, pass and proxychains

optional arguments:
  -h, --help            show this help message and exit
  -v, --verbose
  -d, --dry_run         Don't execute the ssh command
  --rc_file [RC_FILE]   sshmaskrc alternative file
  --use_fwknop          Use fwknop to open the ports
  --use_fwknop_over_tor
                        Torify fwknop command
  --use_ssh_agent       Use ssh-agent
  --use_proxychains     Use proxychains to route the fwknop command
  --fwknop_stanza FWKNOP_STANZA
                        Fwknop stanza name
  --ssh_pass_stanza SSH_PASS_STANZA
                        ssh pass stanza name
  --fwknop_pass_stanza  fwknop pass stanza name
  --interface INTERFACE
                        Select the interface to get the IP from, if external
                        it will retrieve the external ip
```

## Configuration

To configure the access to a remote host (assuming you are using all the features supported by sshmask) you'll first need to configure the ssh and fwknop instances. All the configuration done there will take effect as sshmask will use the original commands.

sshmaskrc is structured as a YAML file. It has at least two sections, one global that set's some default values and then a default section. Any further host sections (named by the hostname) will override the default values of the default section.

It's also possible to override the default configuration with the program flags specified through the command line.

The host sections must follow the same dictionary structure, with just the sections you want to modify.


### Global sshmask configuration

The available global configurations are:

* `fwknoprc`           : Location of the main configuration file of fwknop (Default: ~/.fwknoprc)
* `password_store`     : Location of the pass password store (Default: ~/.password-store)
* `servicerc`          : Location of the service user configuration file (Default: ~/.ssh/config)
* `serviceglobalrc`    : Location of the service global configuration file (Default: /etc/ssh/ssh_config)
* `servicebin`         : Location of the service executable (Default: /usr/bin/ssh)
* `tor_port`           : Tor localhost port (Default: 9050)
* `max_retries`        : Maximum number of retries before failing (Default: 3)
* `service_agent_key_life` : Service agent key lifespan in seconds (Default: 60)

### Default sshmask instance configuration
This configurations belong to the default or any host instance of sshmask.
* `service`            : Dictionary to configure the service
  * `name`             : Service name (Default: ssh)
  * `port`             : Service port (Default: 22)
  * `pass`             : If enabled it will use *pass* to load the service passphrase to the clipboard (Default: 1)
  * `pass_stanza`      : If set it will use the *pass_stanza* of pass, by default it will search one that has `service['name']` and the remote or local hostname strings in it (Default: empty)
  * `agent`            : If enabled it will use the service agent before executing the service command (Default: 1)
  * `discover_timeout` : Timeout in seconds to check if the port is open before launching sshmask machinery (Default: 0.1)
  * `grepstring`       : String to grep to the netcat command (default: OpenSSH)
  * `tor`              : If enabled it will execute the ssh command through tor (Default: 0)
  * `proxychains`      : If enabled it will use proxychains to execute the service (Default: 0)

 * `fwknop`            : Dictionary to configure fwknop
  * `active`           : If enabled it will execute fwknop before the service (Default: 1)
  * `stanza`           : If set it will use the *stanza* of fwknop, by default it will use the local hostname (Default: empty)
  * `pass`             : If enabled it will use *pass* to load the the fwknop passphrase to the clipboard (Default: 1)
  * `pass_stanza`      : If set it will use the *pass_stanza* of pass, by default it will search one that has 'fwknop' and the local hostname strings in it (Default: empty)
  * `tor`              : If enabled it will execute the fwknop command through tor (Default: 0)
  * `proxychains`      : If enabled it will use proxychains to execute fwknop (Default: 0)
 
* `network`            : Dictionary to configure the network
  * `interface`        : If the IP to be allowed is not set on fwknoprc, sshmask will obtain the IP, here we can specify if we want to use the external IP (obtained through https://check.torproject.org) or the internal IP of the *interface_name* (Default: external)

If you want a host named *my_personal_host* to listen to the *eth0* interface you should place the next lines in your sshmaskrc file.

```yaml
my_personal_host:
  network:
    interface: eth0
```

## Troubleshooting

### Enter passphrase for signing: 

If you ar prompted with this message when loading the fwknop key you must specify in the fwknoprc the `KEY` option with the same value as `GPG_SIGNER`


## Roadpath
* [ ] v0.2
  * [ ] Improve ssh parsing 
    * [ ] Use the information of sshrc_global
  * [ ] Support fwknop over tor to .onion services 
  * [ ] Support proxyjumps on ssh with/without tor
  * [?] Support for other protocols to use it as a wrapper for everything 

* [x] v0.1
  * [x] Update README.md
  * [x] Test if config file exists
  * [x] Change configparser to yaml
  * [x] Try plain ssh before sshmask
  * [x] Generalize the executable of ssh to support mosh
  * [x] Improve ssh parsing 
    * [x] Save the configuration of the sshrc in an ssh dictionary inside instance
    * [x] Use the port inside sshrc to check the nc
