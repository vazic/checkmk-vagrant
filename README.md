# Site-Administration
Use the `omd` command to create/configure/destroy/start/stop/restart OMD sites.

```
# sudo omd create <site-name>
```

A site with the name `ies` has already been created for you during provisioning.

Let's go ahead and configure this site. In order to configure a site it has to be stopped, first:

```
vagrant ssh
sudo omd stop ies
sudo omd config ies
```

Under `Basic` choose the `Icinga` core as it has some improvements over the classic `Nagios` core.

Don't forget to restart the site:

```
sudo omd start ies
```

Now that the site is configured and running, let's `su` into it:

```
sudo omd su ies
omd status
exit
```

# Installing the agent
Inside the checkmk-vagrant box, issue:
```
sudo dpkg -i /opt/omd/versions/1.2.6p12.cre/share/check_mk/agents/check-mk-agent_1.2.6p12-1_all.deb
```

Now navigate to the web-ui of the checkmk-vagrant box. It will have received an address from your local DHCP server.
`http://<ip>/ies/`

    note: the trailing directory resembling the site's name.

The default credentials for OMD are:
- username: omdadmin
- password: omd

In order to add a host:
- In the sidebar, go to the WATO `main menu`, click `hosts`, click `add a new host`.
- As a hostname choose `localhost`, as we will monitor the agent running on the vagrant box itself. You can leave all the other settings and host-tags as is.
- Click `Save and go to services`
- Click `save manual check configuration`
- Activate your changes!

# Command-line Interface
First, we need to navigate to the OMD site:

```
vagrant ssh
sudo omd su ies
```

To query localhost for all check results:

```
cmk -nv localhost
```

In order totake a look at the site's monitoring configuration (hosts, services, etc)

```
cmk -D
```

Let's directly query `localhost`'s agent:

```
cmk -d localhost
```

It is also possible to run an inventory check on `localhost`
The inventory check will query the agent for any newly discovered checks that may want to be inventorized.

```
cmk -I localhost
```

# Writing an own agent plugin
Extending the agent with plugins is a twofold process.
- The agent needs to be extended to output a new section containing all infos
- The server needs to be extended to be able to check and inventorize this new section.

## Scenario
Assume we have a random service daemon that writes the agent's coffee level to the file `/coffee` on every host.

In the first step, we extend the agent to output an additional section with the contents of the coffee-file.
Run the following code on the vagrant box, but not inside the OMD site:

```
sudo cat <<EOF > /usr/lib/check_mk_agent/plugins/coffee
#!/bin/bash

# Maintain a reference to the coffee-file.
coffee_file=/coffee

# Output the section header.
echo "<<<coffee>>>"

# Output the contents of the coffee-file or 0 if it doesn't exist.
if [[ -r "$coffee_file" ]]; then
        cat "$coffee_file"
else
        echo 0
fi
EOF

chmod +x /usr/lib/check_mk_agent/plugins/coffee
```

In the second step, we extend the server to inventorize and check the new `<<<coffee>>>` section.

```
sudo omd su ies
cat <<EOF > local/share/check_mk/checks/coffee
default_thresholds = 50, 25


def inventory_coffe(info):
    return [('Current Coffee Level', None)]


def check_coffee(item, params, coffee_level):
    try:
        warn, crit = params
    except:
        warn, crit = default_thresholds
    
    try:
        coffee_level = int(coffee_level[0][0])
    except:
        return (
            3,              # 3 equals state: unknown
            '#clueless'     # An additional description
        )

    if coffee_level < crit:
        return (
            2,              # 2 equals state: critical
            '#fixme',       # An additional description
        )

    if coffee_level < warn:
        return (
            1,              # 1 equals state: warning
            '#danger',      # An additional description
        )

    return (
        0,                  # 0 equals state: ok
        '#thereifxiedit'    # An additional description
    )

check_info["coffee.level"] = (
    check_coffee,           # Check function within this module
    "Current Coffee Level", # Service description
    0,                      # Comes with perf-data
    inventory_coffe,        # Inventory function
)

EOF

cmk -I localhost
cmk -R
cmk -nv localhost | grep coffee
```
