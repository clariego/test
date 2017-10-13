## Lab 16 -  Using Regex to parse device command output

In this lab, we will tackle the problem of gleaning relevant data from a blob of text - typically response data from CLI network devices using the `re` Python standard library.


### Task 1 - Collecting the IOS version from a CSR 

The goal of the first task is to collect the IOS version from **csr1** by parsing the output of the `show version` command.  We'll do this by using a Python script to gather the data and then we'll subsuqently parse this data looking for the OS version.

##### Step 1

Navigate to the `scripts` directory within your home directory.:

```
ntc@ntc:~$ cd scripts
ntc@ntc:~/scripts$
```

##### Step 2

Create a new file called `regex_version.py`:

```
ntc@ntc:~/scripts$ touch regex_version.py
ntc@ntc:~/scripts$
```

##### Step 3

Open this file in Sublime Text or any other text editor.

Write a `main` function that calls another function called `command_output`. We always want to use functions as much as possible to make our code modular.

The main function should provide the device details, such as username, password, IP address and device type.

The `command_output` function should take the command we would like to parse - `show version` - and the device details as inputs and return the device's response.



``` python
from netmiko import ConnectHandler

def command_output(device_details, command):
    """Collect and return command output from device"""
    print("\nConnecting to device {}...\n".format(device_details['ip']))
    device = ConnectHandler(**device_details)
    output = device.send_command(command)

    return output 

```

> We use the `netmiko` library to establish connectivity to the end device and send the command to it.

Here is the `main` function we'll need: 

``` python
def main():
    """Return structured data for show commands"""
    # Device credentials etc
    username = 'ntc'
    password = 'ntc123'
    device_type = 'cisco_ios'
    host = 'csr1'
    # Show command to parse
    command = 'show version'
    device_details = dict(username=username, password=password,
                          ip=host, device_type=device_type)
    # Collect the raw output
    raw_data = command_output(device_details, command)
    print(raw_data)
```

##### Step 4


Save, exit and execute this script to validate that the `show version` output is being collected from the device.


```
ntc@ntc:~/scripts$ python regex_version.py

Connecting to device csr1...

Cisco IOS XE Software, Version 16.03.01
Cisco IOS Software [Denali], CSR1000V Software (X86_64_LINUX_IOSD-UNIVERSALK9-M), Version 16.3.1, RELEASE SOFTWARE (fc3)
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 1986-2016 by Cisco Systems, Inc.
Compiled Tue 02-Aug-16 18:36 by mcpre


Cisco IOS-XE software, Copyright (c) 2005-2016 by cisco Systems, Inc.
All rights reserved.  Certain components of Cisco IOS-XE software are
licensed under the GNU General Public License ("GPL") Version 2.0.  The
software code licensed under GPL Version 2.0 is free software that comes
with ABSOLUTELY NO WARRANTY.  You can redistribute and/or modify such
GPL code under the terms of GPL Version 2.0.  For more details, see the
documentation or "License Notice" file accompanying the IOS-XE software,
or the applicable URL provided on the flyer accompanying the IOS-XE
software.


ROM: IOS-XE ROMMON

csr1 uptime is 3 hours, 6 minutes
Uptime for this control processor is 3 hours, 10 minutes
System returned to ROM by reload
System image file is "bootflash:packages.conf"
Last reload reason: reload



This product contains cryptographic features and is subject to United
States and local country laws governing import, export, transfer and
use. Delivery of Cisco cryptographic products does not imply
third-party authority to import, export, distribute or use encryption.
Importers, exporters, distributors and users are responsible for
compliance with U.S. and local country laws. By using this product you
agree to comply with applicable laws and regulations. If you are unable
to comply with U.S. and local laws, return this product immediately.

A summary of U.S. laws governing Cisco cryptographic products may be found at:
http://www.cisco.com/wwl/export/crypto/tool/stqrg.html

If you require further assistance please contact us by sending email to
export@cisco.com.

License Level: ax
License Type: Default. No valid license found.
Next reload license Level: ax

cisco CSR1000V (VXE) processor (revision VXE) with 2047392K/3075K bytes of memory.
Processor board ID 9KXI0D7TVFI
4 Gigabit Ethernet interfaces
32768K bytes of non-volatile configuration memory.
3984776K bytes of physical memory.
7774207K bytes of virtual hard disk at bootflash:.
0K bytes of  at webui:.

Configuration register is 0x2102

```


##### Step 5

Now rather than print the output of the command, send the raw output to a function called `show_version`. This function will use the `re` Python standard library. 

Go ahead an import `re` at the top of the file

``` python
import re
```

##### Step 6

Create the function `parse_show_version`:

``` python
def parse_show_version(raw_data):

```

##### Step 7

Create a variable called `pattern_to_find`. This is string variable that holds the regular expression pattern for the data we would like to collect from the device output.

``` python

    pattern_to_find = ".*Software\s.+\),\sVersion\s(.+),*\s+RELEASE.*"

```

##### Step 8

Next, loop over every line in the raw data response from the device and look for a pattern match. This is accomplished using `re`'s' `search` method. The `search` method takes the pattern and the line of raw data as inputs.

``` python
    for line in raw_data.splitlines():
        # Collect the matching patterns
        match_result = re.search(pattern_to_find, line)
```

> Note: the search function returns an actual object like this: `<_sre.SRE_Match object at 0xb728bcd0>`

Then, test for a match also within the for loop. If successful collect and print the result.

Since `search` returns an object, we need to use a method of that object to extract that data.  It actually returns a tuple (you can think of this as a list) of the match objects and they are accessed using the `groups` method.  Since we know there is only ever going to be one match, we access the `0` element.


``` python
        if match_result:
            print(line)
            os_version = match_result.groups()[0]
            print("The IOS version on csr1 is {}".format(os_version))
            # Once the version is found, we are exiting the function
            return
    # this pront statment is only executed if there is no match object
    print("Unknown device version")

```


##### Step 9


Call this function from `main()`.

``` python
def main():
    """Return structured data for show commands"""
    # Device credentials etc
    username = 'ntc'
    password = 'ntc123'
    device_type = 'cisco_ios'
    host = 'csr1'
    # Show command to parse
    command = 'show version'
    device_details = dict(username=username, password=password,
                          ip=host, device_type=device_type)
    # Collect the raw output
    raw_data = command_output(device_details, command)
    # Generate structured data from raw output
    parse_show_version(raw_data)


```




##### Step 10

The final script should look similar to :

``` python
#! /usr/bin/env python
"""Code for Lab 17 Task 1- regex search"""
from netmiko import ConnectHandler
import re


def command_output(device_details, command):
    """Collect and return command output from device"""
    print("\nConnecting to device {}...\n".format(device_details['ip']))
    device = ConnectHandler(**device_details)
    output = device.send_command(command)

    return output 


def parse_show_version(raw_data):

    pattern_to_find = ".*Software\s.+\),\sVersion\s(.+),*\s+RELEASE.*"

    for line in raw_data.splitlines():
        # Collect the matching patterns
        match_result = re.search(pattern_to_find, line)
        if match_result:
            print(line)
            os_version = match_result.groups()[0]
            print("The IOS version on csr1 is {}".format(os_version))
            return 
    print("Unknown device version")


def main():
    """Return structured data for show commands"""
    # Device credentials etc
    username = 'ntc'
    password = 'ntc123'
    device_type = 'cisco_ios'
    host = 'csr1'
    # Show command to parse
    command = 'show version'
    device_details = dict(username=username, password=password,
                          ip=host, device_type=device_type)
    # Collect the raw output
    raw_data = command_output(device_details, command)
    # Generate structured data from raw output
    parse_show_version(raw_data)


if __name__ == '__main__':
    main()

```

##### Step 11

Save and exit. Then from the command prompt, execute this script.

``` shell
ntc@ntc:~/scripts$ python regex_version.py

Connecting to device csr1...

Printing results from raw data....

Cisco IOS Software, CSR1000V Software (X86_64_LINUX_IOSD-UNIVERSALK9-M), Version 15.5(3)S2, RELEASE SOFTWARE (fc2)
The IOS version on csr1 is 16.3.1,
```



### Task 2 - Collecting multiple data points from show version using regex

In the previous task, we learned how to collect the OS version from the CSR. In this task we will expand on this and learn how to collect multiple data points from different show command outputs.

##### Step 1

Navigate to the `scripts` directory within your home directory.:

```
ntc@ntc:~$ cd scripts
ntc@ntc:~/scripts$
```

##### Step 2

Copy the script from the last task into `regex_version_data.py`.  This will be the foundation for this task.

```
ntc@ntc:~/scripts$ cp regex_version.py regex_version_data.py
ntc@ntc:~/scripts$

```

##### Step 3

Open this file in Sublime Text or any other text editor.


##### Step 4

We are going to collect multiple pieces of data from the `show version` output. 

Initialize a list variable called `result` at the begining of the function.  This will be used to store all of the data points we'll be collecting.  In addition to creating this variable, make changes to ensure the existing script looks like the following:


``` python
def parse_show_version(raw_data):
    """Print serial number, model, etc from raw data"""
    # Initialize a result set

    result = [] # this is a new line

    pattern_to_find = ".*Software\s.+\),\sVersion\s(.+),*\s+RELEASE.*"

    for line in raw_data.splitlines():
        # Collect the matching patterns
        match_result = re.search(pattern_to_find, line)
        if match_result:
            result.append(match_result)

    for item in result:
        print item.groups()[0]

```


##### Step 5

Save and execute the script.  You should see the following output:

```
ntc@ntc:testing$ python regex_version_data.py 

Connecting to device 34.237.3.13...

16.3.1,
```


##### Step 6

In addition to the OS version, we also need to gather:
 - ROMMON info
 - Hostname
 - Image
 - Serial Number
 - Config Register Value
 
Create a variable for each of these data points that holds the regular expression pattern that matches while also changing the variable `pattern_to_find` to `version_pattern`.


``` python

    version_pattern = ".*Software\s.+\),\sVersion\s(.+),*\s+RELEASE.*"
    rom_pattern = "^ROM: (\S+)"
    hostname_pattern = "^\s*(\S+)\s+uptime\s+is\s+(.+)"
    image_pattern = "^[sS]ystem\s+image\s+file\s+is\s+\"(.*?):(\S+)"
    serial_pattern = "^[Pp]rocessor\s+board\s+ID\s+(\S+)"
    confreg_pattern = "^[Cc]onfiguration\s+register\s+is\s+(\S+)"

```

##### Step 7

Save these patterns into a list, making it easy to iterate over.

``` python
    patterns = [version_pattern, rom_pattern, hostname_pattern,
                image_pattern, serial_pattern, confreg_pattern]

```

##### Step 8

Update the for loop to iterate over each pattern now:

``` python
def parse_show_version(raw_data):
    """Print serial number, model, etc from raw data"""
    # Initialize a result set

    result = [] # this is a new line

    version_pattern = ".*Software\s.+\),\sVersion\s(.+),*\s+RELEASE.*"
    rom_pattern = "^ROM: (\S+)"
    hostname_pattern = "^\s*(\S+)\s+uptime\s+is\s+(.+)"
    image_pattern = "^[sS]ystem\s+image\s+file\s+is\s+\"(.*?):(\S+)"
    serial_pattern = "^[Pp]rocessor\s+board\s+ID\s+(\S+)"
    confreg_pattern = "^[Cc]onfiguration\s+register\s+is\s+(\S+)"

    patterns = [version_pattern, rom_pattern, hostname_pattern,
                image_pattern, serial_pattern, confreg_pattern]

    for line in raw_data.splitlines():
        # Collect the matching patterns
        for pattern in patterns:
            match_result = re.search(pattern, line)
            if match_result:
                result.append(match_result)

    for item in result:
        print item.groups()[0]

```

Now the result list will contain all of the results found.

```
ntc@ntc:testing$ python regex_version_data.py

Connecting to device 34.237.3.13...

15.5(3)S2,
IOS-XE
csr1
bootflash
9HGHHQFKWCA
0x2102
```


##### Step 9

Instead of storing all of the regex strings in a list, make them a dictionary, with the key being what we are parsing in human terms as such:

```
patterns_to_find = dict(os_version=version_pattern, rommon=rom_pattern, hostname=hostname_pattern, image=image_pattern, serial_number=serial_pattern, confreg=confreg_pattern)
```

Update the `parse_show_version` so it looks like this:

```python
def parse_show_version(raw_data):
    """Print serial number, model, etc from raw data"""
    # Initialize a result set

    version_pattern = ".*Software\s.+\),\sVersion\s(.+),*\s+RELEASE.*"
    rom_pattern = "^ROM: (\S+)"
    hostname_pattern = "^\s*(\S+)\s+uptime\s+is\s+(.+)"
    image_pattern = "^[sS]ystem\s+image\s+file\s+is\s+\"(.*?):(\S+)"
    serial_pattern = "^[Pp]rocessor\s+board\s+ID\s+(\S+)"
    confreg_pattern = "^[Cc]onfiguration\s+register\s+is\s+(\S+)"

    # new patterns dictionary
    patterns_to_find = dict(os_version=version_pattern, rommon=rom_pattern, hostname=hostname_pattern, image=image_pattern, serial_number=serial_pattern, confreg=confreg_pattern)

    # this is where we'll store the match objects
    match_dict = {}

    for line in raw_data.splitlines():
        # loop over the patterns to find dict
        for feature, pattern in patterns_to_find.items():
            match_result = re.search(pattern, line)
            if match_result:
                # instead of appending to a list, we're appending to a dictionary
                match_dict[feature] = match_result

    # this new loop gives us context in the print statement
    # the actual `match.groups` statement could have been done in the previous for loop too
    for feature, match in match_dict.items():
        print ("Feature {} --- Value {} ").format(feature, match.groups()[0])


```



##### Step 6

```
ntc@ntc:testing$ python regex_version_data.py 

Connecting to device 34.237.3.13...

Feature image --- Value bootflash 
Feature hostname --- Value csr1 
Feature os_version --- Value 16.3.1
Feature serial_number --- Value 9HGHHQFKWCA 
Feature confreg --- Value 0x2102 
Feature rommon --- Value IOS-XE
```



The final script should look similar to :

``` python
#! /usr/bin/env python


from netmiko import ConnectHandler
import re

def command_output(device_details, command):
    """Collect and return command output from device"""
    print("\nConnecting to device {}...\n".format(device_details['ip']))
    device = ConnectHandler(**device_details)
    output = device.send_command(command)

    return output 

def parse_show_version(raw_data):
    """Print serial number, model, etc from raw data"""
    # Initialize a result set

    version_pattern = ".*Software\s.+\),\sVersion\s(.+),*\s+RELEASE.*"
    rom_pattern = "^ROM: (\S+)"
    hostname_pattern = "^\s*(\S+)\s+uptime\s+is\s+(.+)"
    image_pattern = "^[sS]ystem\s+image\s+file\s+is\s+\"(.*?):(\S+)"
    serial_pattern = "^[Pp]rocessor\s+board\s+ID\s+(\S+)"
    confreg_pattern = "^[Cc]onfiguration\s+register\s+is\s+(\S+)"

    patterns_to_find = dict(os_version=version_pattern, rommon=rom_pattern, hostname=hostname_pattern, image=image_pattern, serial_number=serial_pattern, confreg=confreg_pattern)

    match_dict = {}

    for line in raw_data.splitlines():
        # Collect the matching patterns
        for feature, pattern in patterns_to_find.items():
            match_result = re.search(pattern, line)
            if match_result:
                match_dict[feature] = match_result

    for feature, match in match_dict.items():
        print ("Feature {} --- Value {} ").format(feature, match.groups()[0])

def main():
    """Return structured data for show commands"""
    # Device credentials etc
    username = 'ntc'
    password = 'ntc123'
    device_type = 'cisco_ios'
    host = 'csr1'
    # Show command to parse
    command = 'show version'
    device_details = dict(username=username, password=password,
                          ip=host, device_type=device_type)
    # Collect the raw output
    raw_data = command_output(device_details, command)
    # Generate structured data from raw output
    parse_show_version(raw_data)

if __name__ == '__main__':
    main()

```



### Task 3 - Collecting interface status using regex

The objective of Task 3 is to extend the previous script to collect interface status from the CSR, but this time show how to use the `finall` function when there are multiples in a given show command such as interfaces, VLANs, neighbors, etc.

A typical `show ip interface brief` output looks similar to :

``` shell
csr1# show ip interface brief

Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.0.0.51       YES NVRAM  up                    up      
GigabitEthernet2       unassigned      YES NVRAM  administratively down down    
GigabitEthernet3       unassigned      YES NVRAM  administratively down down    
GigabitEthernet4       unassigned      YES NVRAM  administratively down down    
Loopback100            unassigned      YES unset  up                    up      
Loopback101            unassigned      YES unset  up                    up      

```

##### Step 1

Navigate to the `scripts` directory within your home directory.:

```
ntc@ntc:~$ cd scripts
```

##### Step 2

Copy the script we created in the previous task, into a new file called `regex_show_ip_intf.py`

```
ntc@ntc:~/scripts$ cp regex_version_data.py regex_show_ip_intf.py

```

##### Step 3
Now open this file in Sublime Text or any other editor. Update the `main()` function to pass the command `show ip interface brief` to the device and collect the raw device response back.

``` python
def main():
    """Return structured data for show commands"""
    # Device credentials etc
    username = 'ntc'
    password = 'ntc123'
    device_type = 'cisco_ios'
    host = 'csr1'
    # Show command to parse
    command = 'show version'
    device_details = dict(username=username, password=password,
                          ip=host, device_type=device_type)
    # Collect the raw output
    raw_data = command_output(device_details, command)
    # Generate structured data from raw output
    parse_show_version(raw_data)
    # Repeat for show IP int bri
    command = 'show ip interface brief'
    raw_data = command_output(device_details, command)

```

##### Step 4

Create a new function called `show_ip_int_brief` that takes this raw data as input.

``` python
def parse_show_ip_int_brief(raw_data):

```


##### Step 5

Initialize the result list as an empty variable

``` python
    result = []
```

##### Step 6


Store the regex pattern that will match data points of interest as a variable called `pattern`

``` python
    pattern = "^(\S+)\s+(\S+)\s+\w+\s+\w+\s+(up|down|administratively down)\s+(up|down)"

```

> This will collect the regex groupings corresponding to name of the interface, protocol status, admin status etc


##### Step 7

Now iterate over every line of the raw device output and check for a match. If a match is identified, append the matched data to the result list. Finally, print this list.


``` python
    for line in raw_data.splitlines():
        match_result = re.findall(pattern, line)
        if match_result:
            result.append(match_result)
    # Print the final result
    for data in result:
        print(data)
```

The updated function should look like this:

```python
def parse_show_ip_int_brief(raw_data):
    """Print interface name, ip & status from raw data"""
    # Initialize a result set
    result = []
    # Setting up the patterns
    pattern = "^(\S+)\s+(\S+)\s+\w+\s+\w+\s+(up|down|administratively down)\s+(up|down)"
    print("Printing results from raw data....\n")
    for line in raw_data.splitlines():
        match_result = re.findall(pattern, line)
        if match_result:
            result.append(match_result)
    # Print the final result
    for data in result:
        print(data)
```


##### Step 8

Call this function from within `main()`.

``` python
def main():
    """Return structured data for show commands"""
    # Device credentials etc
    username = 'ntc'
    password = 'ntc123'
    device_type = 'cisco_ios'
    host = 'csr1'
    # Show command to parse
    command = 'show version'
    device_details = dict(username=username, password=password,
                          ip=host, device_type=device_type)
    # Collect the raw output
    raw_data = command_output(device_details, command)
    # Generate structured data from raw output
    show_version(raw_data)
    # Repeat for show IP int bri
    command = 'show ip interface brief'
    raw_data = command_output(device_details, command)
    parse_show_ip_int_brief(raw_data)

```


##### Step 9

The final script should look similar to:

```python

from netmiko import ConnectHandler
import re

def command_output(device_details, command):
    """Collect and return command output from device"""
    print("\nConnecting to device {}...\n".format(device_details['ip']))
    device = ConnectHandler(**device_details)
    output = device.send_command(command)

    return output 

def parse_show_version(raw_data):
    """Print serial number, model, etc from raw data"""
    # Initialize a result set

    version_pattern = ".*Software\s.+\),\sVersion\s(.+),*\s+RELEASE.*"
    rom_pattern = "^ROM: (\S+)"
    hostname_pattern = "^\s*(\S+)\s+uptime\s+is\s+(.+)"
    image_pattern = "^[sS]ystem\s+image\s+file\s+is\s+\"(.*?):(\S+)"
    serial_pattern = "^[Pp]rocessor\s+board\s+ID\s+(\S+)"
    confreg_pattern = "^[Cc]onfiguration\s+register\s+is\s+(\S+)"

    patterns_to_find = dict(os_version=version_pattern, rommon=rom_pattern, hostname=hostname_pattern, image=image_pattern, serial_number=serial_pattern, confreg=confreg_pattern)

    match_dict = {}

    for line in raw_data.splitlines():
        # Collect the matching patterns
        for feature, pattern in patterns_to_find.items():
            match_result = re.search(pattern, line)
            if match_result:
                match_dict[feature] = match_result

    for feature, match in match_dict.items():
        print ("Feature {} --- Value {} ").format(feature, match.groups()[0])

def parse_show_ip_int_brief(raw_data):
    """Print interface name, ip & status from raw data"""
    # Initialize a result set
    result = []
    # Setting up the patterns
    pattern = "^(\S+)\s+(\S+)\s+\w+\s+\w+\s+(up|down|administratively down)\s+(up|down)"
    print("Printing results from raw data....\n")
    for line in raw_data.splitlines():
        match_result = re.findall(pattern, line)
        if match_result:
            result.append(match_result)
    # Print the final result
    for data in result:
        print(data)


def main():
    """Return structured data for show commands"""
    # Device credentials etc
    username = 'ntc'
    password = 'ntc123'
    device_type = 'cisco_ios'
    host = 'csr1'
    # Show command to parse
    command = 'show version'
    device_details = dict(username=username, password=password,
                          ip=host, device_type=device_type)
    # Collect the raw output
    raw_data = command_output(device_details, command)
    # Generate structured data from raw output
    parse_show_version(raw_data)

    command = 'show ip interface brief'
    raw_data = command_output(device_details, command)
    parse_show_ip_int_brief(raw_data)


if __name__ == '__main__':
    main()
```


##### Step 10

Save and exit. Now execute the script from the command line. The output should contain the data points from the `show version` regex parse as well as the `show ip interface brief`

``` shell
ntc@ntc:~/scripts$ python regex_show_ip_intf.py

Connecting to device csr1...

Feature image --- Value bootflash 
Feature hostname --- Value csr1 
Feature os_version --- Value 16.3.1, 
Feature serial_number --- Value 9HGHHQFKWCA 
Feature confreg --- Value 0x2102 
Feature rommon --- Value IOS-XE 

Connecting to device csr1...

Printing results from raw data....

[(u'GigabitEthernet1', u'10.0.0.51', u'up', u'up')]
[(u'GigabitEthernet2', u'unassigned', u'administratively down', u'down')]
[(u'GigabitEthernet3', u'unassigned', u'administratively down', u'down')]
[(u'GigabitEthernet4', u'unassigned', u'administratively down', u'down')]
[(u'Loopback100', u'unassigned', u'up', u'up')]
[(u'Loopback101', u'unassigned', u'up', u'up')]

```


Note: this script can also be optimized so it connects only once to the router.
