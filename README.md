![](https://github.com/carlmontanari/nornsible/workflows/build/badge.svg)
[![PyPI version](https://badge.fury.io/py/nornsible.svg)](https://badge.fury.io/py/nornsible)
[![Python 3.6](https://img.shields.io/badge/python-3.6-blue.svg)](https://www.python.org/downloads/release/python-360/)
[![Python 3.7](https://img.shields.io/badge/python-3.7-blue.svg)](https://www.python.org/downloads/release/python-370/)
[![Python 3.8](https://img.shields.io/badge/python-3.8-blue.svg)](https://www.python.org/downloads/release/python-380/)
[![Code Style](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/ambv/black)

Nornsible
=======

Bring some of the nice things about Ansible to Nornir!

The idea behind Nornsible is to bring some great features of Ansible to Nornir. Specifically tagging, host limiting, and dynamic inventory/multiple inventory sources. 

Nornsible provides the ability to -- via command line arguments -- filter Nornir inventories by hostname or group name, or both. There is also a handy flag to set the number of workers; quite useful for setting workers equal to 1 for troubleshooting purposes.

Nornsible also supports the concept of tags. Tags correspond to the name of *custom* tasks and operate very much like tags in Ansible. Provide a list of tags to execute, and Nornsible will ensure that Nornir only runs those tasks. Provide a list of tags to skip, and Nornsible will ensure that Nornir only runs those not in that list. Easy peasy.

Finally, Nornsible supports multiple inventory files as well as dynamic inventory files -- just like Ansible.

Of course you can simply do all of this yourself, but why not let nornsible handle it for you!


# How does Nornsible work?

Nornsible accepts an instantiated Nornir object as an argument and returns a slightly modified Nornir object. Nornsible sets the desired number of workers if applicable, and adds an attribute for "run_tags" and "skip_tags" based on your command line input.

To take advantage of the tags feature Nornsible provides a decorator that you can use to wrap your custom tasks. This decorator inspects the task being ran and checks the task name against the lists of run and skip tags. If the task is allowed, Nornsible simply allows the task to run as per normal, if it is *not* allowed, Nornsible will print a pretty message and move on.

Nornsible inventory can be used by simply installing Nornsible and setting the inventory plugin in your config file as follows:

```yaml
inventory:
  plugin: nornsible.inventory.AnsibleInventory
  options:
    inventory: inventory.yaml
```

Note that instead of `hostsfile` Nornsible inventory uses `inventory` -- this is intentional to make sure to differentiate between the standard Nornir Ansible support and nornsible.

# Caveats

Nornsible breaks some things! Most notably it breaks "normal" Nornir filtering *after* the Nornir object is "nornsible-ified". This can probably be fixed but at the moment it doesn't seem like that big a deal, so I'm not bothering!

If you want to do "normal" Nornir filtering -- do this *before* passing the nornir object to Nornsible.

Nornsible, at the moment, can only wrap custom tasks. This can probably be improved upon as well, but at the moment the decorator wrapping custom tasks solution seems to work pretty well.


# Installation

Installation via pip:

```
pip install nornsible
```

To install from this repository:

```
pip install git+https://github.com/carlmontanari/nornsible
```

To install from source:

```
git clone https://github.com/carlmontanari/nornsible
cd nornsible
python setup.py install
```


# Usage

Import stuff:

```
from nornsible import InitNornsible, nornsible_task
```

Decorate custom tasks with `nornsible_task` if desired:

```
@nornsible_task
def my_custom_task(task):
```

Create your Nornir object and then pass it through InitNornsible:

```
nr = InitNornir(config_file="config.yaml")
nr = InitNornsible(nr)
```

Run a custom task wrapped by `nornsible_task`:

```
nr.run(task=my_custom_task)
```

Run your script with any of the following command line switches:

| Purpose          | Short Flag    | Long Flag  | Allowed Options
| -----------------|---------------|------------|-------------------|
| set num_workers  | -w            | --workers  | integer           |
| limit host(s)    | -l            | --limit    | comma sep string  |
| limit group(s)   | -g            | --groups   | comma sep string  |
| run tag(s)       | -t            | --tags     | comma sep string  |
| skip tag(s)      | -s            | --skip     | comma sep string  |

To set number of workers to 1 for troubleshooting purposes:

```
python my_nornir_script.py -w 1
```

To limit to the "sea" group (from your Nornir inventory):

```
python my_nornir_script.py -g sea
```

To run only the tasks named "create_configs" and "deploy_configs" (assuming you've wrapped all of your tasks with `nornsible_task`!):

```
python my_nornir_script.py -t create_configs,deploy_configs
```

To run only the tasks named "create_configs" and "deploy_configs" against the "sea-eos-1" host:

```
python my_nornir_script.py -t create_configs,deploy_configs -l sea-eos-1
```


# FAQ

TBA, probably things though!

# Linting and Testing

## Linting

This project uses [black](https://github.com/psf/black) for auto-formatting. In addition to black, tox will execute [pylama](https://github.com/klen/pylama), and [pydocstyle](https://github.com/PyCQA/pydocstyle) for linting purposes. I've also added docstring linting with [darglint](https://github.com/terrencepreilly/darglint) which has been quite handy! Finally, I've been playing with type hints and have added mypy to the test/lint suite as well.

## Testing

I broke testing into two main categories -- unit and integration. Unit is what you would expect -- unit testing the code. Integration testing is for things that test more than one "unit" (generally function) at a time.


# To Do

- Add handling for "not" in group limit; i.e.: "-t !somegroup" to run against all groups *not* somegroup.
- Add wildcard "*" for host/group limit
- Allow for filtering on hosts and groups -- I think basically should be easy... just need to filter groups first then do hosts... removing any skip hosts as needed, but just re-calling filter on the inventory object should sort this... would just need decent testing!
- Add more examples for using nornsible inventory in different ways -- i.e. multiple inventory, dynamic inventory, hash_behavior settings, etc.
- Add more detailed readme info on inventory stuff.
- Add support for host/group vars in directories in the host/group vars parent directory...
    - i.e. `host_vars/myhost/somevar1.yaml` and `host_vars/myhost/somevar2.yaml`
- Add integration testing for inventory bits.
- Fix/add logging -- ensure inventory logs to nornir log as per usual, but also create a nornsible log for all nornsible "stuff".
- Investigate adding a `-C` flag for check mode -- would likely only be able to support netmiko/napalm tasks... not sure best way to do this but would be a nice feature!
- Investigate adding a retry wrapper. Would need to clear failed hosts before retrying tasks. Easily done in normal nornir stuff directly, but could be nice to have here.
- Parametrize the tests... they've gotten outa control for no real reason!