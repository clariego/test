## Lab 14 - Rendering configuration using Jinja2

In the preceding labs, we incrementally built upon previous concepts finally ending with a script that generated configuration and deployed it to an end device, based on user input. All along, we were using 2 functions - `generate_config_file` and `generate_commands` - to build our list of configuration commands. We were relying on the `format` string built-in method to correctly format our desired configurations. This can get quite unmanageable when we have to deal with large configurations.
In this lab, we will use the Jinja2 templating engine to generate the configuration, in lieu of the format functions.
### Task 1 - Collecting input using Python's raw_input method

##### Step 1

Navigate to the `scripts` directory within your home directory.:

```
ntc@ntc:~$ cd scripts
```

##### Step 2

Create a new directory here called `templates` and change to that directory. We will store our Jinja2 template here.

```
ntc@ntc:~$ mkdir templates
ntc@ntc:~$ cd templates
ntc@ntc:~/templates$

```

##### Step 3

Touch a new template file within here.

``` shell
ntc@ntc:~/templates$ touch interfaces.j2
```

##### Step 4

Open this file in Sublime Text or any other text editor.

Write the Jinja2 template configuration that represents the desired interface configuration.

``` python
{% for interface, config_params in interfaces.items() %}
  interface {{ interface }}
  {%- for feature, value in config_params.items() %}
    {{ feature }} {{ value }}
  {%- endfor %}
{%- endfor %}

```
> Note that the `-` symbols at the beginning of the Jinja2 controls are for stripping white space. Feel free to experiment with this, by removing the `-` and rendering the final configuration, during later steps of this lab.

##### Step 5

Navigate back to the `scripts` directory and make a copy the python file you created in the previous lab.

```
ntc@ntc:~/scripts$ cp parse_user_input.py jinja2_generate_config.py
```

##### Step 6

In order to visualize how the Jinja2 templating engine is rendering the configuration, we will explore it using the Python interpreter shell first.

From the command prompt type `python` to enter the Python interpreter shell

``` shell
ntc@ntc:~/scripts$ python
Python 2.7.12 (default, Nov 19 2016, 06:48:10) 
[GCC 5.4.0 20160609] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> 
```

##### Step 7

The Jinja2 templating engine needs to know about the location of the template file(s). This is done by importing the `Enivronment` and `FileSystemloader` objects from the Jinja2 library.

``` shell
>>> from jinja2 import Environment, FileSystemLoader

```

Create a variable called `ENV` to identify the directory containing the template file as a Jinja2 `Environment` object.

``` shell
>>>ENV = Environment(loader=FileSystemLoader('./templates'))

```
Now, collect the template file into a variable called `template`

``` shell
>>>template = ENV.get_template('interfaces.j2')

```

Then, collect the interface configuration data into a variable called interfaces

``` shell
>>> import yaml
>>> 
>>> 
>>> 
>>> with open('csr1.yaml') as yaml_file_handler:
...   interfaces = yaml.load(yaml_file_handler)
... 
>>> print interfaces
{'GigabitEthernet2': {'duplex': 'half', 'speed': 100, 'description': 'Configured_by_Python_GigabitEthernet2'}, 'GigabitEthernet1': {'duplex': 'full', 'speed': 1000, 'description': 'Configured_by_Python_GigabitEthernet1'}, 'Loopback101': {'description': 'Configured_by_Python_Loopback101'}, 'Loopback100': {'description': 'Configured_by_Python_Loopback100'}}

```

Finally pass the data to the template and render it.

``` shell
>>>interface_config = template.render(interfaces=interfaces)

```
The resulting output is contained in the variable interfaces_config.

``` shell
>>> print(interface_config)

  interface GigabitEthernet2
    duplex half
    speed 100
    description Configured_by_Python_GigabitEthernet2
  interface GigabitEthernet1
    duplex full
    speed 1000
    description Configured_by_Python_GigabitEthernet1
  interface Loopback101
    description Configured_by_Python_Loopback101
  interface Loopback100
    description Configured_by_Python_Loopback100
>>> 

```


##### Step 6

Open this new file in Sublime Text or any other text editor.

We will be using the Jinja2 library to render the configuration. Go ahead and import the `Environment` and `FileSystemLoader` objects at the top of the script.

``` python
from jinja2 import Environment, FileSystemLoader

```




##### Step 7

Define a new function called `render_config` that takes the configuration data as input.

``` python
def render_config(interfaces):

```


The functions that we already created in the previous lab can be reused. The new function needed to collect user input will use python's inbuilt `raw_input` function.

> The `raw_input` function is replaced with `input` in Python 3

Go ahead and add this new function, calling it `user_input_interactive`

``` python

def user_input_interactive():
    """Collect the device details from the user interactively"""
    host = raw_input("Please enter the hostname or IP: ")
    device_type = raw_input("Please enter the device type: ")
    username = raw_input("Please enter the username: ")
    password = raw_input("Please enter the password: ")
    device_details = dict(device_type=device_type, ip=host,
                          username=username, password=password)
    return(device_details)

```
> `raw_input` allows you to prompt for user input by displaying the string passed to it.

This function will collect the FQDN/IP address of the device, the device type and login credentials from the user, through an interactive prompt. 

##### Step 4

Call this new function from `main()`. The `device_details` dictionary will be used to store the login details and the device type, needed by `netmiko` to connect to `csr1`. The values are provided by calling the function we just defined.


``` python
def main():
    """Generate and write interface configurations to a file
    """

    # Collect device details from user
    device_details = user_input_interactive()

    interfaces_dict = get_interfaces_from_file()
    # Call a function that returns the configuration
    commands_list = get_commands_list(interfaces_dict)

    # Call a function that writes configs to a file
    file_name = write_config(commands_list)

    # Output the file details
    print("File {} has been generated...".format(file_name))

    # Deploy the configurations
    deploy_config(file_name, device_details)

    # End


```

    
##### Step 5

The final, complete script should look like below:

``` python
#!/usr/bin/env python
""" Code for Lab 15, Task 1"""
import yaml
from netmiko import ConnectHandler


def generate_commands(config_params):
    """Generate specific feature commands using feature name & value."""

    cmd_list = []
    for feature, value in config_params.items():
        command = " {} {}".format(feature, value)
        cmd_list.append(command)
    return cmd_list


def get_commands_list(interfaces):
    """Return a list of interface configuration commands."""

    # Iterate over the dictionary and generate configuration.

    commands_list = []
    for interface, config_params in interfaces.items():
        interface_command = "interface {}".format(interface)
        commands_list.append(interface_command)
        feature_commands = generate_commands(config_params)
        commands_list.extend(feature_commands)

    return commands_list


def print_config(commands_list):
    """Print the commands as a list and config."""
    # Print the results as a list
    print("Commands as a List:")
    print(commands_list)
    print("--------------------")
    # Print the results as config
    print("Commands Simulating Config File:")
    for command in commands_list:
        print(command)


def generate_config_file(commands_list, file_name):
    """Write interface configs to a file"""
    print("Opening file {} to write...".format(file_name))

    with open(file_name, "w") as file_handler:
        for command in commands_list:
            file_handler.write("{}\n".format(command))


def write_config(commands_list):
    """Returns the file name of generated configurations."""
    file_path = '/tmp/device.cfg'
    print("Opening file {} to write...".format(file_path))
    with open(file_path, "w") as file_handler:
        for command in commands_list:
            file_handler.write("{}\n".format(command))
    return(file_path)


def get_interfaces_from_file():
    """ Read in YAML data of the interfaces and generate the dictionary"""
    with open('csr1.yaml') as yaml_file_handler:
        interfaces = yaml.load(yaml_file_handler)
    return interfaces


def get_interfaces():
    """ Return a dictionary of interfaces containing attributes"""
    interfaces = {
        "GigabitEthernet1": {
            "duplex": "full",
            "speed": 1000,
            "description": "Configured_by_Python_GigabitEthernet1"
        },
        "GigabitEthernet2": {
            "duplex": "half",
            "speed": 100,
            "description": "Configured_by_Python_GigabitEthernet2"
        },
        "Loopback101": {
            "description": "Configured_by_Python_Looback101"
        },
        "Loopback100": {
            "description": "Configured_by_Python_Loopback100"
        }
    }

    return interfaces


def deploy_config(file_name, device_details):
    """Connects to the device and deploys the configuration
    """

    print("Connecting to the remote device {}...\n"
          .format(device_details['ip']))
    # Invoke netmiko ConnectHandler and pass it the device details
    device = ConnectHandler(**device_details)
    # Send the config file
    print("Sending the configuration from file {}...".format(file_name))
    device.send_config_from_file(config_file=file_name)
    device.disconnect()
    print("Changes sent to device. Please log in and verify...")

    return


def user_input_interactive():
    """Collect the device details from the user interactively"""
    host = raw_input("Please enter the hostname or IP: ")
    device_type = raw_input("Please enter the device type: ")
    username = raw_input("Please enter the username: ")
    password = raw_input("Please enter the password: ")
    device_details = dict(device_type=device_type, ip=host,
                          username=username, password=password)
    return(device_details)


def main():
    """Generate and write interface configurations to a file
    """

    # Collect device details from user
    device_details = user_input_interactive()

    interfaces_dict = get_interfaces_from_file()
    # Call a function that returns the configuration
    commands_list = get_commands_list(interfaces_dict)

    # Call a function that writes configs to a file
    file_name = write_config(commands_list)

    # Output the file details
    print("File {} has been generated...".format(file_name))

    # Deploy the configurations
    deploy_config(file_name, device_details)

    # End


if __name__ == "__main__":
    main()

```


##### Step 6

Save and execute this script:


``` shell
ntc@ntc:~/scripts$ python raw_user_input.py
Please enter the hostname or IP: csr1
Please enter the device type: cisco_ios
Please enter the username: ntc
Please enter the password: ntc123
Opening file /tmp/device.cfg to write...
File /tmp/device.cfg has been generated...
Connecting to the remote device csr1...

Sending the configuration from file /tmp/device.cfg...
Changes sent to device. Please log in and verify...

```

##### Step 7

Finally, log into device `csr1` and ensure that the changes were pushed to the device.


``` shell
csr1#show interfaces description 
Interface                      Status         Protocol Description
Gi1                            up             up       Configured_by_Python_GigabitEthernet1
Gi2                            admin down     down     Configured_by_Python_GigabitEthernet2
Gi3                            admin down     down     
Gi4                            admin down     down     
Lo100                          up             up       Configured_by_Python_Loopback100
Lo101                          up             up       Configured_by_Python_Loopback101
csr1#
```

You have now successfully created a modular python script that reads in configration data from a YAML encoded file, generates device interface configurations and deploys the configurations to the end device!
