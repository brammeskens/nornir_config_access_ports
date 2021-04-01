# Nornir config access ports
## Dependencies
```
    pip3 install nornir
    pip3 install nornir_utils
    pip3 install nornir_jinja2
    pip3 install nornir_netmiko

    pip3 install rich
    pip3 install getpass
    pip3 install ntc-templates
```

## Overview
This script is built around Nornir3 and its various plugins. For ease of use and readability, getpass and rich were also incorporated. This Python script will run three tasks:
- get_access_ports
- generate_config
- push_config

For each host we will gather all switchports and filter only the access ports. Once the ports are gathered, we will generate configuration for the host device based on the jinja2 template. Last but not least, we push the generated configuration to the hosts. 

## How to Use
**Note 1:** General usage of Nornir is required. Please see [nornir.tech](https://nornir.tech) or [nornir-automation](https://github.com/nornir-automation/nornir/)

**Note 2:** Using the [ntc-templates](https://github.com/networktocode/ntc-templates) through PyPI never worked out for me. To ease the set-up I have copied the ntc_templates folder directly in the same destination folder. You may have better luck than me by using the PyPI-version.

**Step 1**
Make sure you have your Nornir inventory file setup as you would like. In case you would like to filter your hosts based on their parent group, you may want to add the following code after Nornir initialization (line 54):
`nr = nr.filter(F(has_parent_group='afg'))`

**Step 2**
Alter the jinja2 template file located at _templates/8021x_mon.j2_ or create a new jinja2 file to your liking in the folder. In our example _8021x_mon.j2_ file you will find a general part of the configuration we want to send to the host (change it as needed). Next you will find a section to generate the specific configuration to the access ports:
```
{% for i in host["access_ports"] %}
interface {{i['interface']}}
authentication event fail action next-method
authentication event server dead action authorize
authentication event server dead action authorize voice
authentication event server alive action reinitialize
authentication host-mode multi-domain
authentication open
authentication order dot1x mab
authentication priority dot1x mab
authentication port-control auto
authentication periodic
authentication timer reauthenticate server
authentication violation replace
mab
dot1x pae authenticator
dot1x timeout tx-period 8
spanning-tree portfast
!
{% endfor %}
```
Your custom access port configuration should be inserted between _interface {{i['interface']}}_ and _!_.

**Step 3**
You should be ready to run the script now. If you feel uncertain about the _push_config_ task, we advise you to comment out lines _68_ and _69_ before you run the script.
