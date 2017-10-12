## Lab 15 - Rendering configuration using Jinja2

In the preceding labs, we incrementally built upon previous concepts finally ending with a script that generated configuration and deployed it to an end device, based on user input. All along, we were using 2 functions - `generate_config_file` and `generate_commands` - to build our list of configuration commands. We were relying on the `format` string built-in method to correctly format our desired configurations. This can get quite unmanageable when we have to deal with large configurations.
In this lab, we will use the Jinja2 templating engine to generate the configuration, in lieu of the format functions.
### Task 1 - Collecting input using Python's raw_input method

##### Step 1

Navigate to the `scripts` directory within your home directory.:

```
ntc@ntc:~$ cd scripts
ntc@ntc:~/scripts$
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

Save, exit and navigate back to the `scripts` directory.

``` shell
ntc@ntc:~/templates$ cd ../scripts

```


##### Step 6

In order to visualize how the Jinja2 templating engine is rendering the configuration, we will explore it using the Python interpreter shell first.

From the command prompt type `python` to enter the Python interpreter shell

``` python
ntc@ntc:~/scripts$ python
Python 2.7.12 (default, Nov 19 2016, 06:48:10) 
[GCC 5.4.0 20160609] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> 
```

##### Step 7

The Jinja2 templating engine needs to know about the location of the template file(s). This is done by importing the `Enivronment` and `FileSystemloader` objects from the Jinja2 library.

``` python
>>> from jinja2 import Environment, FileSystemLoader

```

Create a variable called `ENV` to identify the directory containing the template file as a Jinja2 `Environment` object.

``` python
>>>ENV = Environment(loader=FileSystemLoader('./templates'))

```
Now, collect the template file into a variable called `template`

``` python
>>>template = ENV.get_template('interfaces.j2')

```

Then, collect the interface configuration data into a variable called interfaces

``` python
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

``` python
>>> interface_config = template.render(interfaces=interfaces)

```
The resulting output is contained in the variable interfaces_config.

``` python
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



##### Step 8


Ensure you are in the `scripts` directory and make a copy the python file you created in the previous lab.

```
ntc@ntc:~/scripts$ cp parse_user_input.py jinja2_generate_config.py
```


##### Step 9

Open this new file in Sublime Text or any other text editor.

We will be using the Jinja2 library to render the configuration. Go ahead and import the `Environment` and `FileSystemLoader` objects at the top of the script.

``` python
from jinja2 import Environment, FileSystemLoader

```

Create a new function called `render_config` that takes the configuration data as input. This function implements rendering the configuration using the Jinja2 template we did in the previous steps.

``` python
def render_config(interfaces):
    """Return the rendered configuration as string."""
    ENV = Environment(loader=FileSystemLoader('./templates'))
    template = ENV.get_template('interfaces.j2')
    interface_config = template.render(interfaces=interfaces)
    return(interface_config)

```



##### Step 10

Call this new function from `main()`. 

Add the following Python statements just above the call to the `generate_config_file` function:

```python
    # Call a function that returns the configuration
    config = render_config(interfaces_dict)
    # Deploy the configurations
    # push_config(config, device_details)
    commands_list = config.splitlines()

```

The new complete main function should look like this:

``` python
def main():
    """Generate and write interface configurations to a file
    """

    # Collect input from user
    user_input = user_input_parse()

    # Collect the file name
    file_name = user_input.pop('file_name')

    # Collect the device_details
    device_details = user_input

    # Collect the interface details from file
    interfaces_dict = get_interfaces_from_file(file_name)

    # Call a function that returns the configuration
    config = render_config(interfaces_dict)
    # Deploy the configurations
    # push_config(config, device_details)
    commands_list = config.splitlines()

    file_name = `/tmp/device.cfg'
    generate_config_file(commands_list, file_name)
    deploy_config(file_name, device_details)
    # End


```

The resulting configuration is stored in the `config` variable. From this data, we can generate a list of commands by invoking the inbuilt string method `splitlines`. From this point, we use the functions we used earlier to save the configuration lines into a file and then deploy it.

##### Step 11

The final, complete script should look like below:

``` python
#!/usr/bin/env python
""" Code for Lab 16, Task 2"""
import yaml
from netmiko import ConnectHandler
import argparse
from jinja2 import Environment, FileSystemLoader


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
    # Output the file details
    print("File {} has been generated...".format(file_name))


def get_interfaces_from_file(file_name):
    """ Read in YAML data of the interfaces and generate the dictionary"""
    with open(file_name) as yaml_file_handler:
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
    host = input("Please enter the hostname or IP: ")
    device_type = input("Please enter the device type: ")
    username = input("Please enter the username: ")
    password = input("Please enter the password: ")
    device_details = dict(device_type=device_type, ip=host,
                          username=username, password=password)
    return(device_details)


def user_input_parse():
    """Collect the device details using an unix-command-like menu"""
    parser = argparse.ArgumentParser(description='Collect device and data'
                                     ' file information to configure a device')
    parser.add_argument('-f', '--file_name',
                        help='Enter the full path to the YAML data file',
                        default='csr1.yaml')
    parser.add_argument('-i', '--ip',
                        help='Enter the IP address or hostname of the device',
                        required=True)
    parser.add_argument('-d', '--device_type', help='Enter the device type',
                        required=True)
    parser.add_argument('-u', '--username', help='Enter the username',
                        required=True)
    parser.add_argument('-p', '--password', help='Enter the password',
                        required=True)
    input_data = parser.parse_args()
    host = input_data.ip
    username = input_data.username
    password = input_data.password
    device_type = input_data.device_type
    file_name = input_data.file_name

    user_input = dict(device_type=device_type, ip=host,
                      username=username, password=password,
                      file_name=file_name)
    return(user_input)


def render_config(interfaces):
    """Return the rendered configuration as string."""
    ENV = Environment(loader=FileSystemLoader('./templates'))
    template = ENV.get_template('interfaces.j2')
    interface_config = template.render(interfaces=interfaces)
    return(interface_config)


def main():
    """Generate and write interface configurations to a file
    """

    # Collect input from user
    user_input = user_input_parse()

    # Collect the file name
    file_name = user_input.pop('file_name')

    # Collect the device_details
    device_details = user_input

    # Collect the interface details from file
    interfaces_dict = get_interfaces_from_file(file_name)

    # Call a function that returns the configuration
    config = render_config(interfaces_dict)
    # Deploy the configurations
    # push_config(config, device_details)
    commands_list = config.splitlines()

    file_name = `/tmp/device.cfg'
    generate_config_file(commands_list, file_name)
    deploy_config(file_name, device_details)
    # End

if __name__ == "__main__":
    main()

```



##### Step 12

Save and execute this script:


``` shell
ntc@ntc:~/scripts$ python jinja2_generate_config.py  -i csr1 -d cisco_ios -u ntc -p ntc123 -f csr1.yaml
Opening file /tmp/device.cfg to write...
File /tmp/device.cfg has been generated...
Connecting to the remote device csr1...

Sending the configuration from file /tmp/device.cfg...
Changes sent to device. Please log in and verify...


```

##### Step 13

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

You have now successfully rendered interface configurations using a Jinja2 template and data from a YAML file. The rendered configuration was then deployed to a network device!
