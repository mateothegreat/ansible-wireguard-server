# Wireguard via ansible like a boss

This ansible role deploys a wireguard server and generates client configurations.

## Requirements

- [ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

## Usage

Create a file called `deploy.yaml` with the following contents:

```yaml
- hosts: "bastion"                                    # This is the host that we will be running the role on.
  module_defaults:                                    # Configure the module defaults.
    ansible.builtin.setup:                            # Configure the setup module defaults.
      gather_subset:                                  # This is a list of facts that will be gathered (we don't need this so let's speed things up).
        - "!min"                                      # Ignore the minimum facts..
        - "!all"                                      # and then ignore all remaining facts.
  roles:                                              # This is a list of roles that will be run.
    - name: "wireguard-server"                        # This is the name of the role.
      vars:                                           # These are the variables that will be passed to the role.
        wireguard:                                    # This is the wireguard configuration passed to the role.
          dns_servers:                                # This is a list of DNS servers that the wireguard clients will use.
            - "168.63.129.16"                         # Azure private zone DNS resolver.
            - "1.1.1.1"                               # Cloudflare DNS resolver.
          server:                                     # This is the server side configuration.
            interface_name: "wg0"                     # This is the name of the wireguard interface server side.
            cidr: "172.172.0.0/22"                    # This is the CIDR block that the wireguard server will use (/22 has the range of 172.172.0.0 - 172.172.3.255).
            address: "172.172.0.1/32"                 # This is the IP address that the wireguard server will assume.
            public_address: "bastion.matthewdavis.io" # This is the public IP address or hostname that the wireguard server is listening on.
            public_port: 51820                        # This is the port that the wireguard server will listen on.
            routes:                                   # This is a list of CIDR blocks that will be routed through the wireguard clients.
              - "10.1.0.0/24"                         # This is a CIDR block that will be routed through the wireguard clients.
          clients:                                    # This is the client side configurations.
            output_path: "/tmp/wireguard-clients"     # This is the path where the client config files will be written to.
            peers:                                    # This is a list of clients that will be able to connect to the server.
              - name: "matthew@matthewdavis.io"       # This is the name of the client config file (mainly for documentation purposes).
                address: "172.172.0.10/32"            # This is the IP address that the end users wireguard client will assume.
              - name: "it@aint.easy"
                address: "172.172.0.11/32"
```

For your convenience, you can use the following references:

* For an inventory reference, see [test/inventory.yaml](test/inventory.yaml).
* For a playbook reference, see [test/deploy.yaml](test/deploy.yaml).

### End-to-end wireguard server deployment

To deploy the wireguard server, generate keys, configure, and
generate client configurations, run the following command:

```bash
ansible-playbook -i inventory.yaml deploy.yaml
```

Example output:

```bash
...

Playbook run took 0 days, 0 hours, 0 minutes, 10 seconds
Wednesday 10 May 2023  05:50:51 -0400 (0:00:01.093)       0:00:10.142 ********* 
=============================================================================== 
wireguard-server : Install wireguard package ---------------------------------------------------------------------- 1.54s
Gathering Facts --------------------------------------------------------------------------------------------------- 1.53s
wireguard-server : Write server private key to host --------------------------------------------------------------- 1.46s
wireguard-server : Copy wireguard configuration to remote --------------------------------------------------------- 1.24s
wireguard-server : Write server public key to host ---------------------------------------------------------------- 1.22s
wireguard-server : (re)enable and (re)start wireguard service ----------------------------------------------------- 1.09s
wireguard-server : Generate client configuration files ------------------------------------------------------------ 0.62s
wireguard-server : Create client configuration output path (if necessary) ----------------------------------------- 0.35s
wireguard-server : Generate client private key -------------------------------------------------------------------- 0.34s
wireguard-server : Generate client public key --------------------------------------------------------------------- 0.33s
wireguard-server : Generate server private key -------------------------------------------------------------------- 0.24s
wireguard-server : Generate server public key --------------------------------------------------------------------- 0.17s
```

### Install wireguard package only

To install or upgrade the wireguard package only, run the following command:

```bash
ansible-playbook -i inventory.yaml deploy.yaml --tags install
```

Example output:

```bash
../ansible 🌱 main [!+] ➜ venv/bin/ansible-playbook -i inventory.yaml deploy.yaml --tags install

PLAY [bastion] ***********************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************
Wednesday 10 May 2023  05:48:02 -0400 (0:00:00.013)       0:00:00.013 ********* 
ok: [bastion-1]

TASK [wireguard-server : Install wireguard package] **********************************************************************
Wednesday 10 May 2023  05:48:03 -0400 (0:00:00.595)       0:00:00.609 ********* 
ok: [bastion-1]

TASK [wireguard-server : (re)enable and (re)start wireguard service] *****************************************************
Wednesday 10 May 2023  05:48:04 -0400 (0:00:01.530)       0:00:02.140 ********* 
changed: [bastion-1]

PLAY RECAP ***************************************************************************************************************
bastion-1                  : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

Playbook run took 0 days, 0 hours, 0 minutes, 3 seconds
Wednesday 10 May 2023  05:48:06 -0400 (0:00:01.196)       0:00:03.336 ********* 
=============================================================================== 
wireguard-server : Install wireguard package ---------------------------------------------------------------------- 1.53s
wireguard-server : (re)enable and (re)start wireguard service ----------------------------------------------------- 1.20s
Gathering Facts --------------------------------------------------------------------------------------------------- 0.60s
```

---
```text
┌───────────────────────┐       ┌──────────────────────────────┐
│ Install wireguard via │───────►       │ Generate sever config        │
│ package manager.      ├ i.e. /etc/wireguard/wg0.conf │
└───────────────────────┘       └──────────────────────────────┘
```
