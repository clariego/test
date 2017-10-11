## Lab 15 - Jinja2 Templating in Python

In the preceding labs, we incrementally built upon previous concepts finally ending with a script that generated configuration and deployed it to an end device, based on user input. All along, we were using 2 functions - `generate_config_file` and `generate_commands` - to build our list of configuration commands. We were relying on the `format` string built-in method to correctly format our desired configurations. This can get quite unmanageable when we have to deal with large configurations.
In this lab, we will use the Jinja2 templating engine to generate the configuration, in lieu of the format functions.

### Task 1 - Collecting input using Python's raw_input method

##### Step 1

Navigate to the `templates` sub-directory within the `scripts` directory.:

```
ntc@ntc:~$ cd scripts/templates
```

##### Step 2

Touch a new template file within here.

``` shell
ntc@ntc:~/templates$ touch interfaces.j2
```

##### Step 3

Open this file in Sublime Text or any other text editor.

Write the Jinja2 template configuration that represents the desired interface configuration.  This template is going to loop over the interfaces dictionary that has been used in previous labs.

As a reminder, this is the data structure we need to loop over in Jinja:

```
{'GigabitEthernet2': {'duplex': 'half', 'speed': 100, 'description': 'Configured_by_Python_GigabitEthernet2'}, 'GigabitEthernet1': {'duplex': 'full', 'speed': 1000, 'description': 'Configured_by_Python_GigabitEthernet1'}, 'Loopback101': {'description': 'Configured_by_Python_Loopback101'}, 'Loopback100': {'description': 'Configured_by_Python_Loopback100'}}
```

This is the Jinja template that needs to go into `interfaces.j2`:

``` python
{% for interface, config_params in interfaces.items() %}
  interface {{ interface }}
  {%- for feature, value in config_params.items() %}
    {{ feature }} {{ value }}
  {%- endfor %}
{%- endfor %}

```

> Note that the `-` symbols at the beginning of the Jinja2 controls are for stripping white space. 
> Feel free to experiment with this, by removing the `-` and rendering the final configuration, during later steps of this lab.

When the dictionary, or _data_, is rendered with the template, we will end up with a valid configuration.


##### Step 4

Navigate back to the `scripts` directory and make a copy the Python file you created in the previous lab.

```
ntc@ntc:~/scripts$ cp parse_user_input.py jinja2_generate_config.py
```

##### Step 5

In order to visualize how the Jinja2 templating engine is rendering the configuration, we will explore it using the Python interpreter shell first.

From the command prompt type `python` to enter the Python interpreter shell

``` 
ntc@ntc:~/scripts$ python
Python 2.7.12 (default, Nov 19 2016, 06:48:10) 
[GCC 5.4.0 20160609] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> 
```

##### Step 6

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

The resulting output is contained in the variable `interfaces_config`.

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


##### Step 7

Open this new file in Sublime Text or any other text editor.

We will be using the Jinja2 library to render the configuration. Go ahead and import the `Environment` and `FileSystemLoader` objects at the top of the script.

``` python
from jinja2 import Environment, FileSystemLoader

```

##### Step 8

Define a new function called `render_config` that takes the configuration data as input.

``` python
def render_config(interfaces):

```


    
##### Step 9

The final, complete script should look like below:

``` python

```


##### Step 10

Save and execute this script:

