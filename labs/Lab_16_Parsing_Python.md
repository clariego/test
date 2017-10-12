## Lab 16 -  Using Regex to parse device command output

In this lab, we will tackle the problem of gleaning relevant data from a blob of text - typically response data from CLI network devices using the `regex` standard Python library.


### Task 1 - Collecting the IOS version from a CSR 

The goal of the first task is to collect the IOS version from `csr1` by parsing the output of the `show version` command.

##### Step 1

Navigate to the `scripts` directory within your home directory.:

```
ntc@ntc:~$ cd scripts
```

##### Step 2

Create a new file called `regex_version.py` by touching it.

```
ntc@ntc:~/scripts$ touch regex_version.py

```

##### Step 3

Open this file in Sublime Text or any other text editor.

Write a `main` function that calls another function called `command_output`. The main function should provide the device details, such as username, password, IP address and device type.
The `command_output` function should take the command we would like to parse - `show version` - and the device details as inputs and return the device's response.

``` python
from netmiko import ConnectHandler

def command_output(device_details, command):
    """Collect and return command output from device"""
    print("\nConnecting to device {}...\n".format(device_details['ip']))
    device = ConnectHandler(**device_details)
    return(device.send_command(command))

```
> We use the `netmiko` library to establish connectivity to the end device and send the command to it.

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



``` shell
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

Now rather than print the output of the command, send it a function called `show_version`. Create this function to use the `re` standard Python library. Go ahead an import `re` at the top of the file

``` python
import re
```

Create the function `show_version`:

``` python
def show_version(raw_data):

```
Create a variable called `version_pattern`. This is simply a string variable that holds the regular expression pattern for the data we would like to collect from the device output

``` python

    version_pattern = ".*Software\s.+\),\sVersion\s(.+),*\s+RELEASE.*"

```

Next, go over ever line in the raw data response from the device and look for a pattern match. This is accomplished using `re's` `search` method. The `search` method takes the pattern and the line of raw data as inputs.

``` python
    for line in raw_data.splitlines():
        # Collect the matching patterns
        match_result = re.search(version_pattern, line)
```

Then, test for a match. If successful collect and print the result.


``` python
        if match_result:
            version = match_result.groups()[0]
    print("The IOS version on csr1 is {}".format(version))


```

##### Step 6


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
    show_version(raw_data)


```




##### Step 7

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
    return(device.send_command(command))


def show_version(raw_data):
    """Print the Version from raw data"""
    # Setting up the patterns
    version_pattern = ".*Software\s.+\),\sVersion\s(.+),*\s+RELEASE.*"
    # Processing the patterns
    print("Printing results from raw data....\n")
    for line in raw_data.splitlines():
        # Collect the matching patterns
        match_result = re.search(version_pattern, line)
        if match_result:
            version = match_result.groups()[0]
    print("The IOS version on csr1 is {}".format(version))


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


if __name__ == '__main__':
    main()

```

##### Step 8

Save and exit. Then from the command prompt, execute this script.

``` shell
ntc@ntc:~/scripts$ python regex_version.py

Connecting to device csr1...

Printing results from raw data....

The IOS version on csr1 is 16.3.1,
```

### Task 2 - Collecting multiple data points from show version using regex

In the previous task, we learned how to collect the OS version from the CSR. In this task we will expand on this and learn how to collect multiple data points from different show command outputs.

##### Step 1

Navigate to the `scripts` directory within your home directory.:

```
ntc@ntc:~$ cd scripts
```

##### Step 2

Create a new file called `regex_version_data.py` by touching it.

```
ntc@ntc:~/scripts$ touch regex_version_data.py

```

##### Step 3

Open this file in Sublime Text or any other text editor.

Write a `main` function that calls another function called `command_output`. The main function should provide the device details, such as username, password, IP address and device type.
The `command_output` function should take the command we would like to parse - `show version` - and the device details as inputs and return the device's response.


``` python
from netmiko import ConnectHandler

def command_output(device_details, command):
    """Collect and return command output from device"""
    print("\nConnecting to device {}...\n".format(device_details['ip']))
    device = ConnectHandler(**device_details)
    return(device.send_command(command))

```

> We use the `netmiko` library to establish connectivity to the end device and send the command to it.

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
```

> This step is identical to Step 3 in Task 1
##### Step 4

Now send the raw output of the command, to a function called `show_version`. Create this function to use the `re` standard Python library. Go ahead an import `re` at the top of the file

``` python
import re
```

Create the function `show_version`:

``` python
def show_version(raw_data):

```

We are going to collect multiple pieces of data from the `show version` output. Initialize a list variable called `result` at the begining of the function:


``` python
def show_version(raw_data):
    """Print serial number, model, etc from raw data"""
    # Initialize a result set
    result = []

```

In addition to the OS version, we need to gather:
 - ROMMON info
 - Hostname
 - Image
 - Serial Number
 - Config Register Value
 
Create a variable for each of these data points that holds the regular expression pattern that matches.



``` python

    version_pattern = ".*Software\s.+\),\sVersion\s(.+),*\s+RELEASE.*"
    rom_pattern = "^ROM: (\S+)"
    hostname_pattern = "^\s*(\S+)\s+uptime\s+is\s+(.+)"
    image_pattern = "^[sS]ystem\s+image\s+file\s+is\s+\"(.*?):(\S+)"
    serial_pattern = "^[Pp]rocessor\s+board\s+ID\s+(\S+)"
    confreg_pattern = "^[Cc]onfiguration\s+register\s+is\s+(\S+)"

```

Collect these patterns into a list, making it easy to iterate over.

``` python
    patterns = [version_pattern, rom_pattern, hostname_pattern,
                image_pattern, serial_pattern, confreg_pattern]

```

Next, go over ever line in the raw data response from the device. For each line, validate whether we have a match against any of our patterns. 

``` python
    for line in raw_data.splitlines():
        # Collect the matching patterns
        for pattern in patterns:
            match_result = re.findall(pattern, line)

```

If a match occurs, collect the result into the `result[]` list and then print this list.

``` python
            if match_result:
                result.append(match_result)
    # Print the final result
      for data in result:
          print(data)
```

>We use the `re.findall` method for matching multiple search groups against a line of text.


##### Step 5

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
    show_version(raw_data)

```



##### Step 6

The final script should look similar to :

``` python
#! /usr/bin/env python
"""Code for Lab 17 - regex"""
from netmiko import ConnectHandler
import re


def command_output(device_details, command):
    """Collect and return command output from device"""
    print("\nConnecting to device {}...\n".format(device_details['ip']))
    device = ConnectHandler(**device_details)
    return(device.send_command(command))


def show_version(raw_data):
    """Print serial number, model, etc from raw data"""
    # Initialize a result set
    result = []
    # Setting up the patterns
    version_pattern = ".*Software\s.+\),\sVersion\s(.+),*\s+RELEASE.*"
    rom_pattern = "^ROM: (\S+)"
    hostname_pattern = "^\s*(\S+)\s+uptime\s+is\s+(.+)"
    image_pattern = "^[sS]ystem\s+image\s+file\s+is\s+\"(.*?):(\S+)"
    serial_pattern = "^[Pp]rocessor\s+board\s+ID\s+(\S+)"
    confreg_pattern = "^[Cc]onfiguration\s+register\s+is\s+(\S+)"
    # A list of patterns to iterate over
    patterns = [version_pattern, rom_pattern, hostname_pattern,
                image_pattern, serial_pattern, confreg_pattern]
    # Processing the patterns
    print("Printing results from raw data....\n")
    for line in raw_data.splitlines():
        # Collect the matching patterns
        for pattern in patterns:
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
    show_version(raw_data)

if __name__ == '__main__':
    main()

```

##### Step 7

Save and exit. Then from the command prompt, execute this script.

``` shell
ntc@ntc:~/scripts$ python regex_version.py

Connecting to device csr1...

Printing results from raw data....

[u'16.3.1,']
[u'IOS-XE']
[(u'csr1', u'4 hours, 33 minutes')]
[(u'bootflash', u'packages.conf"')]
[u'9KXI0D7TVFI']
[u'0x2102']

```

### Task 3 - Collecting interface status using regex

The objective of Task 3 is to extend the previous script to collect interface status from the CSR.

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
    show_version(raw_data)
    # Repeat for show IP int bri
    command = 'show ip interface brief'
    raw_data = command_output(device_details, command)

```

##### Step 4

Create a new function called `show_ip_int_brief` that takes this raw data as input.

``` python
def show_ip_int_brief(raw_data):

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
    show_ip_int_brief(raw_data)

```


##### Step 9

The final script should look similar to:

``` python
#! /usr/bin/env python
"""Code for Lab 17 - regex"""
from netmiko import ConnectHandler
import re


def command_output(device_details, command):
    """Collect and return command output from device"""
    print("\nConnecting to device {}...\n".format(device_details['ip']))
    device = ConnectHandler(**device_details)
    return(device.send_command(command))


def show_ip_int_brief(raw_data):
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

def show_version(raw_data):
    """Print serial number, model, etc from raw data"""
    # Initialize a result set
    result = []
    # Setting up the patterns
    version_pattern = ".*Software\s.+\),\sVersion\s(.+),*\s+RELEASE.*"
    rom_pattern = "^ROM: (\S+)"
    hostname_pattern = "^\s*(\S+)\s+uptime\s+is\s+(.+)"
    image_pattern = "^[sS]ystem\s+image\s+file\s+is\s+\"(.*?):(\S+)"
    serial_pattern = "^[Pp]rocessor\s+board\s+ID\s+(\S+)"
    confreg_pattern = "^[Cc]onfiguration\s+register\s+is\s+(\S+)"
    # A list of patterns to iterate over
    patterns = [version_pattern, rom_pattern, hostname_pattern,
                image_pattern, serial_pattern, confreg_pattern]
    # Processing the patterns
    print("Printing results from raw data....\n")
    for line in raw_data.splitlines():
        # Collect the matching patterns
        for pattern in patterns:
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
    show_version(raw_data)
    # Repeat for show IP int bri
    command = 'show ip interface brief'
    raw_data = command_output(device_details, command)
    show_ip_int_brief(raw_data)


if __name__ == '__main__':
    main()

```

##### Step 10

Save and exit. Now execute the script from the command line. The output should contain the data points from the `show version` regex parse as well as the `show ip interface brief`

``` shell
ntc@ntc:~/scripts$ python regex_show_ip_intf.py
Connecting to device csr1...

Printing results from raw data....

[u'16.3.1,']
[u'IOS-XE']
[(u'csr1', u'4 hours, 39 minutes')]
[(u'bootflash', u'packages.conf"')]
[u'9KXI0D7TVFI']
[u'0x2102']

Connecting to device csr1...

Printing results from raw data....

[(u'GigabitEthernet1', u'10.0.0.51', u'up', u'up')]
[(u'GigabitEthernet2', u'unassigned', u'administratively down', u'down')]
[(u'GigabitEthernet3', u'unassigned', u'administratively down', u'down')]
[(u'GigabitEthernet4', u'unassigned', u'administratively down', u'down')]
[(u'Loopback100', u'unassigned', u'up', u'up')]
[(u'Loopback101', u'unassigned', u'up', u'up')]

```
