# Nornir config access ports
## Dependencies
```
    pip3 install nornir
    pip3 install nornir_utils
    pip3 install nornir_jinja2
    pip3 install nornir_netmiko

    pip3 install rich
    pip3 install ntc-templates
```

## Overview
This script is built around Nornir 3.0 and its various plugins. For ease of use and readability, getpass and rich were also incorporated. This Python script will run three tasks:
- get_access_ports
- generate_config
- push_config

For each host we will gather all switchports and filter only the access ports. Once the ports are gathered, we will generate configuration for the host device based on the jinja2 template. The jinja2 template conists of general host configuration and interface configuration. Last but not least, we push the generated configuration to the hosts. 

## How to Use
**Note:** Using the [ntc-templates](https://github.com/networktocode/ntc-templates) through PyPI never worked out for me. To ease the set-up I have copied the ntc_templates folder directly in the same destination folder. The corresponding code can be found at line _52_. You may have better luck than me by using the PyPI-version.

**Step 1 - Download the repository**

Choose a path of your liking and clone the GitHub repositoy in this path:
`git clone https://github.com/brammeskens/nornir_config_access_ports.git`

**Step 2 - Create your Python virtual environment**

It's generally cleaner to use Python virtual environments as each virtual environment has its own Python binary and independent packages. So we will create one:
`python3 -m venv nornir_config_access_ports`

**Step 3 - Activate your Python virutal environment**

After the creation of the virtual environment, we should activate it so we can actually make use of it:
```
cd nornir_config_access_ports
source bin/activate
```

**Step 4 - Install the dependencies**

Let's install the dependencies with the requirements.txt file from the repo:
`pip3 install -r requirements`

**Step 5 - Create your Nornir 3 inventory**

General usage of Nornir is required. Please see [nornir.tech](https://nornir.tech) or [nornir-automation](https://github.com/nornir-automation/nornir/). Example files have already been supplied at the _inventory_ folder. Alter these files to match your environment:
```
inventory/defaults.yaml
inventory/groups.yaml
inventory/hosts.yaml
```

**Step 6 - Filter your Nornir 3 inventory to your liking**

It's a good idea to start small and not run your script on your whole inventory. That's where Nornir 3's filtering comes into play. In case you would like to filter your hosts based on their parent group, you may want to add the following code after Nornir initialization (line _54_):
`nr = nr.filter(F(has_parent_group='afg'))`
For further Nornir 3 filtering examples and practice see [Nornir 3 Filtering Demo](https://developer.cisco.com/codeexchange/github/repo/writememe/nornir-filtering-demo).

**Step 7 - Change the jinja2 template to the configuration you want**

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
Your custom access port configuration should be inserted between `interface {{i['interface']}}` and `!`. If your have changed the template's name or are using a new jinja2 template file, alter the name at line _64_ next to _j2template=_:
`    result_gen_config = nr.run(task = generate_config, j2path= "templates/", j2template = "8021x_mon.j2")`

**Step 8 - Run the script**

You should be ready to run the script now. If you feel uncertain about the _push_config_ task, we advise you to comment out lines _68_ and _69_ by inserting _#_ at the beginning of the line before you run the script. To run the script you simply enter:
`python3 config_access_ports.py`
The script will prompt you for the user's (defined in _inventory/defaults.yaml_) password and will run the following tasks by default:
```
get_access_ports
generate_config
push_config
```

As always, **test** your changes to a demo host in a lab environment **before** actually using it in a production environment.
