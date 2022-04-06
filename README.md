# Ansible role - KYPO Sandbox Logging

This role represents the master Ansible role for the logging configuration of the hosts. It uses several Ansible roles: 

1. [sandbox-logging-bash](https://gitlab.ics.muni.cz/muni-kypo/ansible-roles/sandbox-logging-bash) - local terminal command logging.
2. [sandbox-logging-msf](https://gitlab.ics.muni.cz/muni-kypo/ansible-roles/sandbox-logging-msf) - local Metasploit command logging.
3. [sandbox-logging-forward](https://gitlab.ics.muni.cz/muni-kypo/ansible-roles/sandbox-logging-forward) - forwarding of the local logs from the sandbox host to the specified remote host. 


[[_TOC_]]

## Requirements

* Bash/ZSH on the machine from which you want to collect the commands.

* The role requires root access, so you either need to specify `become` directive as a global or while invoking the role.

    ```yml
    become: yes
    ```
* The above-mentioned roles are applied to the hosts based on the Ansible variables, therefore don't disable directive `gather_facts`.
* If the role is applied to the host with Kali Linux distribution it needs to have Metasploit installed. 


## Sub-roles parameters 

### 1. sandbox-logging-bash 

* None 

### 2. sandbox-logging-msf

* None

### 3. sandbox-logging-forward

* **Mandatory parameters**
  * `slf_destination` - A resolvable hostname or IP address of the destination log host that should be active and able to process logs in the format defined in the RFC5424 standard. The variable doesn't have to be set if the variable `hostvars['man'].ansible_host` is defined instead, but `slf_destination` has the highest priority.

* **Dependant mandatory parameters** - depends on the used environment. Define parameters from one of the following environments.
  1. **Cloud environment** => `slf_local_env = false`: 
     * `slf_sandbox_id` - Sandbox ID. The variable doesn't have to be set if the variable `kypo_global_sandbox_allocation_unit_id` is defined instead, but `slf_sandbox_id` has the highest priority.
     * `slf_pool_id` - Pool ID. The variable doesn't have to be set if the variable `kypo_global_pool_id` is defined instead, but `slf_pool_id` has the highest priority.
  2. **Local environment** => `slf_local_env = true`: 
     * `slf_user_id` - User ID.
     * `slf_access_token` - Access token of the training instance.

* **Optional parameters**
  * `slf_sandbox_name` - Sandbox name (default: `None`). The variable `kypo_global_sandbox_name` can be used instead, but `slf_sandbox_name` has the highest priority.
  * `slf_destination_port` - Remote host destination port (default: `514`).
  * `slf_destination_protocol` - The transport protocol used for transmission of the logs to the remote host. Values 'tcp', 'tls' and 'udp' are supported. (default: `tcp`).
  * `slf_forward_log_severity` - Severity of the forwarded log entries (default: `*`).
  * `slf_tag_regexes` - Filter messages based on the Syslog tags that matches one of these regexes (default: `['bash.command', 'msf.command']`)

* **Encrypted communication**

  To use encrypted communication you must set `slf_destination_protocol` to `tls` and set the following parameter:
  * `slf_rsyslog_ca_cert_file` - Path to the trusted CA certificate (default: `None`).

  To use **mutual authentication** set the following parameters:
  * `slf_rsyslog_cert_file` - Path to the client certificate in PEM format matching the private key set in the `slf_rsyslog_key_file` (default: `None`).
  * `slf_rsyslog_key_file` - Path to the unencrypted client private key in PEM format. Define this variable to enable **encrypted communication** (default: `None`). TCP protocol must be set.

## Example

The simplest example of sandbox logging configuration.

```
roles:
  - role: sandbox-logging
    slf_destination: 192.168.0.5
    slf_sandbox_id: sandbox-1
    become: yes

```

## Caution

* Please make sure you have synchronized time of the VMs with the desired remote system, e.g. your local computer. See [chrony](https://gitlab.ics.muni.cz/muni-kypo/ansible-roles/chrony) for Linux time configuration or [w32time](https://gitlab.ics.muni.cz/muni-kypo/ansible-roles/w32time) for Windows time configuration.

* If you face up the error: **"E: Package 'python-apt' has no installation candidate"** during provisioning of the host, set up the `ansible_python_interpreter: /usr/bin/python3` variable for the host, e.g., as follows: 
  ```
  - name: Configure logging 
    hosts: 
      - home
    ...  
    vars:  
      ansible_python_interpreter: /usr/bin/python3
    roles:
      - role: sandbox-logging
    ...  

  ```
