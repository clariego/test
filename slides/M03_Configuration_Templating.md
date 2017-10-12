layout: true

.footer-picture[![Network to Code Logo](slides/media/Footer2.PNG)]
.footnote-left[(C) 2015 Network to Code, LLC. All Rights Reserved. ]
.footnote-con[CONFIDENTIAL]

---

class: center, middle, title
.footer-picture[<img src="slides/media/Footer1.PNG" alt="Blue Logo" style="alight:middle;width:350px;height:60px;">]

# Configuration Templating

---



# Network Configuration Building

<br>
<br>

.center[
<img src="slides/media/templating.png" alt="Templating" style="align:middle;width:800px;height:205px;">
]

---


# Network Configuration Building

.center[
<img src="slides/media/ansible-templating.png" alt="Ansible Templating" style="alight:middle;width:600px;height:405px;">
]

---


# Templating

Two components are required:

  1.  The template
  2.  Variables (or data) to insert into the template

De-constructing configs/files into templates:

.left-column[

```bash
snmp-server community ro PUBLIC
```

```bash
Dear John,

Thanks for attending.
```
]

.right-column[

```bash
snmp-server community ro {{ ro_string }}
```

```bash
Dear {{ NAME }},

Thanks for attending.
```
]

---

# Templating 

Variables (or data) needed to render with the template.

Various options:

* User input
* Structured Data Files
* Variables in a script
* Databases

---


# Tools & Technologies

- Python - our language of choice
- Jinja2 - templating engine for Python (others are available)
- YAML   - structured data format (superset of JSON)

Note: we review a more scalable way of templating in the Ansible section.

---

class: ubuntu

# Jinja2 in Python

- Python module called `jinja2`
  + Installation: `sudo pip install jinja2`
- Two objects required to create an Environment to read and render templates:
  - `Environment` 
  - `FileSystemLoader`
  - Pass directory that has templates as a parameter to `FileSystemLoader`


```
>>> from jinja2 import Environment, FileSystemLoader
>>> 
>>> ENV = Environment(loader=FileSystemLoader('.'))
>>>
```

---


# Variable Substitution

Variables are denoted by double curly braces

```bash
# filename: snmp.j2

snmp-server community ro {{ ro_string }}
```

Key methods:
* `get_template`
* `render`

.ubuntu[
```
>>> from jinja2 import Environment, FileSystemLoader
>>> ENV = Environment(loader=FileSystemLoader('.'))
>>> 
>>> template = ENV.get_template("snmp.j2")
>>>
```
]
.ubuntu[
```
>>> config = template.render(ro_string='networktocode')
>>> 
>>> print config
snmp-server community ro networktocode
>>> 
```
]
---

# For loops in Jinja2

Example: `list` in Python and for loop in Jinja2

```bash
# filename: vlans.j2

{% for vlan in vlans %}
vlan {{ vlan }}
{%- endfor %}
```

.ubuntu[
```
>>> vlans_list = [1, 2, 3, 10, 20]
>>> 
>>> template = ENV.get_template("vlans.j2")
>>> 
>>> config = template.render(vlans=vlans_list)
>>> 
>>> print config

vlan 1
vlan 2
vlan 3
vlan 10
vlan 20
```
]

---

# For loops in Jinja2 (cont'd)

Example: List of dictionaries (in YAML) and for loop in Jinja2

Need to model data and understand how to access it in Jinja2

```bash
# acls.j2
{% for rule in policies %}
id {{ rule.id }} {{ rule.action }} from {{ rule.source }} to {{ rule.dest }} {% if rule.log %} log {% endif %}
{%- endfor %}
```

.left-column[
```yaml
---

# policies.yml
access_policies:
  - id: 10
    action: permit
    source: inside
    dest: outside
    log: true
  - id: 20
    action: permit
    source: 1.1.1.1/32
    dest: outside
    log: true
  - { id: 30, action: permit, source: 10.0.0.0/8, dest: outside, log: false }

```
]

--

.right-column[



.ubuntu[
```
>>> template = ENV.get_template("acls.j2")
>>> 
>>> acls = yaml.load(open('policies.yml'))
>>> 
>>> config = template.render(policies=acls['access_policies'])
>>> 
>>> print config

id 10 permit from inside to outside  log 
id 20 permit from  1.1.1.1/32 to outside  log 
id 30 permit from 10.0.0.0/8 to outside
```
]

]

---


# Summary

- More smaller templates scale better than fewer monotlithic templates
- Significant value in de-coupling the data from the CLI
- Version control your templates, scripts, and inputs (data, variables)
- Templates will be used thoroughly within Ansible 

---

# Lab

Lab 13 - Jinja2 Templating
  - Creating a Jinja2 Template
  - Working with Jinja2 templates in Python
  - Reading in data from YAML and inserting into a template

