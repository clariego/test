## Lab 14 - Jinja2 Templating in Python

### Task 1 - Creating Configuration Templates

This task walks through how to convert network device configurations into Jinja2 templates and variables while working on the Python shell.

##### Step 1

Create a new sub-directory in the `scripts` directory called `templates`.

```
ntc@ntc:~/scripts$ mkdir templates
```

In the new sub-directory, create a new file called `snmp-1.j2`.

In this file, create a template that could be used instead of the following SNMP configuration.  

```
snmp-server contact JOHN SMITH
snmp-server location NEW YORK CITY
snmp-server community networktocode ro
snmp-server community netsecret123 rw
```

You should replace each value with a variable surrounded by double curly braces.

The new template (`/home/ntc/scripts/templates/snmp-1.j2`) should look like this:

```
snmp-server contact {{ contact }}
snmp-server location {{ location }}
snmp-server community {{ ro_string }} ro
snmp-server community {{ rw_string }} rw

```


##### Step 2

While you are in the `templates` directory, enter the Python shell.

```
ntc@ntc:~/templates$ python
Python 2.7.6 (default, Jun 22 2015, 17:58:13) 
[GCC 4.8.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

##### Step 3

Create four new variables while in the Python shell that are equal to the values that were replaced when creating the template.

```
>>> snmp_contact = 'JOHN SMITH'
>>> snmp_location = 'NEW YORK CITY'
>>> snmp_ro = 'networktocode'
>>> snmp_rw = 'netsecret123'
>>>
```

##### Step 4

Import the required Python modules to work with Jinja2 in Python and then load the template `snmp-1.j2` you created in Step 1.

```python
>>> from jinja2 import Environment, FileSystemLoader

>>> ENV = Environment(loader=FileSystemLoader('.'))

>>> template = ENV.get_template("snmp-1.j2")
>>> 
```

##### Step 5

Now render the template with the variables with the variables you created.

```python
>>> snmp_config = template.render(contact=snmp_contact, location=snmp_location, ro_string=snmp_ro, rw_string=snmp_rw)
>>> 
```

Print the `snmp_config` using the `print` command.

```python
>>> print snmp_config
snmp-server contact JOHN SMITH
snmp-server location NEW YORK CITY
snmp-server community networktocode ro
snmp-server community netsecret123 rw
>>> 
```

This config could now be sent to the device with netmiko (or equivalent).

##### Step 6

Create the following new variable in the Python shell.

```python
>>> snmp = dict(contact='JACK SMITH', location='NYC', ro='n2c', rw='priv123')
>>> 
>>> print snmp
{'location': 'NYC', 'rw': 'priv123', 'ro': 'n2c', 'contact': 'JACK SMITH'}
>>> 
```

##### Step 7 (Challenge)

Create a new template called `snmp-2.j2` that will work with the new variable just created, but still produce the same commands string.

Ensure that passing a single variable called `snmp` to the template will allow it to render.

Once the new template is created, you use it in the next step.

Note: the solution is provided below.

```

Scroll for the solution.


































```

**Solution:**

```
snmp-server contact {{ snmp.contact }}
snmp-server location {{ snmp.location }}
snmp-server community {{ snmp.ro }} ro
snmp-server community {{ snmp.rw }} rw
```

You could have also done:

```
snmp-server contact {{ snmp['contact'] }}
snmp-server location {{ snmp['location'] }}
snmp-server community {{ snmp['ro'] }} ro
snmp-server community {{ snmp['rw'] }} rw
```


##### Step 8

Now render the template with the variables with the variables you created.

```python
>>> template = ENV.get_template("snmp-2.j2")
>>> 
>>> new_snmp_config = template.render(snmp=snmp)
>>> 
```

Print the `snmp_config` using the `print` command.

```python
>>> print new_snmp_config
snmp-server contact JACK SMITH
snmp-server location NYC
snmp-server community n2c ro
snmp-server community priv123 rw
>>> 
```

Do not exit the Python shell.


### Task 2 - Loading Data from a YAML file and rendering it with a Jinja2 Template

##### Step 1

This task builds on the previous task, but rather than create variables in Python, you will read them in from a YAML file.

Create a **new** Jinja2 template called `acls.j2` that has the following text:

```
{% for rule in rules %}
id {{ rule.id }} {{ rule.action }} from {{ rule.source }} to {{ rule.dest }} {% if rule.log %} log {% endif %}
{%- endfor %}
```

> Note: In this template `rules` is a list of dictionaries and `rule` (each element in the list) is a dictionary.

##### Step 2

In the same directory as your templates, create a new YAML file that will be used as input data and rendered with the `acls` template.  

Save the YAML file as `security.yml`

The YAML file should have a single key called `rules` and it should be a list of dictionaries (key/value pairs).  Each rule entry should have five (5) key value pairs:

  1. `id`     - integer, or id, to identify rule
  2. `action` - permit or deny
  3. `source` - IP Address, subnet, or group object name (simulation)
  4. `dest`   - IP Address, subnet, or group object name (simulation)
  5. `log`    - set true

The solution is provided below.

```
Keep scrolling for the solution on what should be inside the YAML file.










































```


YAML File solution:



```yaml
---

rules:
  - id: 2
    action: permit
    source: 1.1.1.1/23
    dest: 4.4.4.4/32
    log: true
  - id: 5
    action: permit
    source: 1.1.1.1/23
    dest: 4.4.4.4/32
    log: true

```



##### Step 3

Import the `yaml` module while you are in the Python shell.

```python
>>> import yaml
>>>
```

Read in the `acls.j2` template.

```python
>>> template = ENV.get_template("acls.j2")
```


##### Step 4

Load the contents of the YAML file you created as a Python object and render the object with the template.

Note: `acl_entries` will look different and will be based upon what you have in your custom YAML file.

> The YAML file when are keys are defined imports as one large dictionary in Python.
> 

```python
>>> sec_rules = yaml.load(open('security.yml'))
>>> print sec_rules
# output omitted 
```

See how the YAML file looks as dictionary?

##### Step 5

Render the final configuration using the `render` function.

```python
>>> acl_entries = template.render(rules=sec_rules['rules'])
>>>
>>> print acl_entries

id 10 permit from inside to outside  log 
id 20 permit from 1.1.1.1/32 to outside  log 
id 30 permit from 10.0.0.0/8 to outside 
>>> 

```



