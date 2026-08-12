# CCNA Automation Code Examples

A hands-on code reference for CCNA Automation study. Replace all placeholders like `<DEVICE_IP>`, `<USERNAME>`, `<PASSWORD>` with your own lab values. Never hardcode real credentials.

---

## 1. Python Fundamentals for Network Automation

### Variables

```python
hostname = "R1"
mgmt_ip = "192.168.1.1"
ssh_port = 22
is_reachable = True
uptime_days = 45.5
```

### Lists, Dictionaries, Tuples

```python
# List of device hostnames
devices = ["R1", "R2", "SW1", "SW2"]

# Dictionary describing a single device
device = {
    "hostname": "R1",
    "ip": "192.168.1.1",
    "device_type": "cisco_ios",
    "vlan": 10
}

# List of dictionaries = device inventory
inventory = [
    {"hostname": "R1", "ip": "192.168.1.1"},
    {"hostname": "SW1", "ip": "192.168.1.2"},
]

# Tuple - immutable, used for fixed pairs
interface_status = ("GigabitEthernet0/1", "up")
```

### Loops

```python
# Loop through device list
for dev in devices:
    print(f"Connecting to {dev}")

# Loop through inventory dictionaries
for dev in inventory:
    print(f"{dev['hostname']} -> {dev['ip']}")

# While loop - retry logic
attempts = 0
while attempts < 3:
    print(f"Attempt {attempts + 1}")
    attempts += 1
```

### Functions

```python
def build_vlan_config(vlan_id: int, vlan_name: str) -> list:
    """Return list of CLI commands to create a VLAN."""
    return [
        f"vlan {vlan_id}",
        f"name {vlan_name}",
        "exit"
    ]

commands = build_vlan_config(10, "SALES")
print(commands)
```

### Exception Handling

```python
def connect_device(ip: str):
    try:
        if not ip:
            raise ValueError("IP address cannot be empty")
        print(f"Connecting to {ip}...")
        # simulate a failure
        raise TimeoutError("Device did not respond")
    except ValueError as e:
        print(f"Input error: {e}")
    except TimeoutError as e:
        print(f"Connection error: {e}")
    except Exception as e:
        print(f"Unexpected error: {e}")
    finally:
        print("Connection attempt finished.")

connect_device("192.168.1.1")
```

### Reading/Writing Files

```python
# Write list of hostnames to a file
with open("hostnames.txt", "w") as f:
    for dev in devices:
        f.write(dev + "\n")

# Read them back
with open("hostnames.txt", "r") as f:
    for line in f:
        print(line.strip())
```

### JSON Handling

```python
import json

device = {"hostname": "R1", "ip": "192.168.1.1", "vlan": 10}

# Python dict -> JSON string
json_str = json.dumps(device, indent=2)
print(json_str)

# JSON string -> Python dict
parsed = json.loads(json_str)
print(parsed["hostname"])

# Write to file
with open("device.json", "w") as f:
    json.dump(device, f, indent=2)

# Read from file
with open("device.json", "r") as f:
    loaded = json.load(f)
print(loaded)
```

### YAML Handling

```bash
pip install pyyaml
```

```python
import yaml

inventory_yaml = """
devices:
  - hostname: R1
    ip: 192.168.1.1
    device_type: cisco_ios
  - hostname: SW1
    ip: 192.168.1.2
    device_type: cisco_ios
"""

data = yaml.safe_load(inventory_yaml)
print(data["devices"][0]["hostname"])

# Write YAML to file
with open("inventory.yml", "w") as f:
    yaml.dump(data, f, default_flow_style=False)

# Read YAML from file
with open("inventory.yml", "r") as f:
    loaded = yaml.safe_load(f)
print(loaded)
```

---

## 2. Python Networking

### IP Address Manipulation with `ipaddress`

```python
import ipaddress

network = ipaddress.ip_network("192.168.1.0/24")
print(network.num_addresses)
print(network.netmask)

for host in list(network.hosts())[:5]:
    print(host)

ip = ipaddress.ip_address("192.168.1.10")
print(ip in network)

# Subnetting
subnets = list(network.subnets(new_prefix=26))
for s in subnets:
    print(s)
```

### Sockets (basic reachability check)

```python
import socket

def check_port(ip: str, port: int, timeout: int = 3) -> bool:
    """Check if a TCP port (e.g. SSH 22) is open."""
    try:
        with socket.create_connection((ip, port), timeout=timeout):
            return True
    except (socket.timeout, ConnectionRefusedError, OSError):
        return False

if check_port("192.168.1.1", 22):
    print("SSH port open")
else:
    print("SSH port closed/unreachable")
```

### HTTP Requests with `requests`

```bash
pip install requests
```

```python
import requests

response = requests.get("https://api.github.com")
print(response.status_code)
print(response.json())
```

### GET / POST / PUT / DELETE Requests

```python
import requests

base_url = "https://api.example.com/devices"

# GET
r = requests.get(base_url)

# POST - create resource
r = requests.post(base_url, json={"hostname": "R1", "ip": "192.168.1.1"})

# PUT - full update
r = requests.put(f"{base_url}/1", json={"hostname": "R1", "ip": "192.168.1.99"})

# DELETE
r = requests.delete(f"{base_url}/1")
```

### Headers

```python
headers = {
    "Content-Type": "application/json",
    "Accept": "application/json"
}

response = requests.get("https://api.example.com/devices", headers=headers)
```

### Authentication Examples

```python
import requests
import os

# Basic Auth
response = requests.get(
    "https://<DEVICE_IP>/restconf/data/",
    auth=("<USERNAME>", os.environ.get("DEVICE_PASSWORD")),
    verify=False
)

# Bearer Token Auth
token = os.environ.get("API_TOKEN")
headers = {"Authorization": f"Bearer {token}"}
response = requests.get("https://api.example.com/data", headers=headers)
```

---

## 3. REST APIs

### REST API GET

```python
import requests

r = requests.get("https://api.example.com/devices", timeout=10)
print(r.status_code)
print(r.json())
```

### REST API POST

```python
payload = {"hostname": "SW1", "ip": "192.168.1.2"}
r = requests.post("https://api.example.com/devices", json=payload)
print(r.status_code, r.json())
```

### REST API PUT

```python
payload = {"hostname": "SW1", "ip": "192.168.1.55"}
r = requests.put("https://api.example.com/devices/5", json=payload)
```

### REST API DELETE

```python
r = requests.delete("https://api.example.com/devices/5")
print(r.status_code)
```

### JSON Parsing

```python
data = r.json()
hostname = data.get("hostname", "unknown")
for item in data.get("devices", []):
    print(item["hostname"], item["ip"])
```

### API Authentication (Token Fetch Pattern)

```python
import requests
import os

auth_url = "https://api.example.com/auth/login"
creds = {"username": "<USERNAME>", "password": os.environ.get("API_PASSWORD")}

auth_resp = requests.post(auth_url, json=creds, verify=False)
token = auth_resp.json()["token"]

headers = {"Authorization": f"Bearer {token}"}
data_resp = requests.get("https://api.example.com/devices", headers=headers, verify=False)
```

### Error Handling

```python
import requests

try:
    r = requests.get("https://api.example.com/devices", timeout=5)
    r.raise_for_status()
    print(r.json())
except requests.exceptions.Timeout:
    print("Request timed out")
except requests.exceptions.ConnectionError:
    print("Could not connect to API")
except requests.exceptions.HTTPError as e:
    print(f"HTTP error: {e}")
except requests.exceptions.RequestException as e:
    print(f"General request error: {e}")
```

### Status Code Handling

```python
r = requests.get("https://api.example.com/devices")

if r.status_code == 200:
    print("Success:", r.json())
elif r.status_code == 401:
    print("Unauthorized - check credentials")
elif r.status_code == 404:
    print("Resource not found")
elif r.status_code == 500:
    print("Server error")
else:
    print(f"Unexpected status: {r.status_code}")
```

---

## 4. Cisco RESTCONF

### Enable RESTCONF on Cisco IOS XE (device CLI)

```
configure terminal
ip http secure-server
restconf
username <USERNAME> privilege 15 algorithm-type scrypt secret <PASSWORD>
end
```

### RESTCONF GET (list interfaces)

```python
import requests
import os

requests.packages.urllib3.disable_warnings()

device_ip = "<DEVICE_IP>"
url = f"https://{device_ip}/restconf/data/ietf-interfaces:interfaces"

headers = {
    "Accept": "application/yang-data+json"
}

r = requests.get(
    url,
    headers=headers,
    auth=("<USERNAME>", os.environ.get("DEVICE_PASSWORD")),
    verify=False
)
print(r.status_code)
print(r.json())
```

### RESTCONF POST (create a VLAN)

```python
url = f"https://{device_ip}/restconf/data/Cisco-IOS-XE-native:native/vlan"

headers = {
    "Content-Type": "application/yang-data+json",
    "Accept": "application/yang-data+json"
}

payload = {
    "Cisco-IOS-XE-native:vlan-list": [
        {"id": 20, "name": "MARKETING"}
    ]
}

r = requests.post(
    url,
    headers=headers,
    json=payload,
    auth=("<USERNAME>", os.environ.get("DEVICE_PASSWORD")),
    verify=False
)
print(r.status_code)
```

### RESTCONF PUT (replace interface config)

```python
url = f"https://{device_ip}/restconf/data/ietf-interfaces:interfaces/interface=GigabitEthernet2"

payload = {
    "ietf-interfaces:interface": {
        "name": "GigabitEthernet2",
        "description": "Configured via RESTCONF",
        "enabled": True,
        "ietf-ip:ipv4": {
            "address": [{"ip": "10.10.10.1", "netmask": "255.255.255.0"}]
        }
    }
}

r = requests.put(url, headers=headers, json=payload,
                  auth=("<USERNAME>", os.environ.get("DEVICE_PASSWORD")), verify=False)
```

### RESTCONF PATCH (partial update - change description only)

```python
url = f"https://{device_ip}/restconf/data/ietf-interfaces:interfaces/interface=GigabitEthernet2"

payload = {
    "ietf-interfaces:interface": {
        "name": "GigabitEthernet2",
        "description": "Updated via RESTCONF PATCH"
    }
}

headers["Content-Type"] = "application/yang-data+json"

r = requests.patch(url, headers=headers, json=payload,
                    auth=("<USERNAME>", os.environ.get("DEVICE_PASSWORD")), verify=False)
```

### RESTCONF DELETE (remove a VLAN)

```python
url = f"https://{device_ip}/restconf/data/Cisco-IOS-XE-native:native/vlan/vlan-list=20"

r = requests.delete(url, headers=headers,
                     auth=("<USERNAME>", os.environ.get("DEVICE_PASSWORD")), verify=False)
print(r.status_code)
```

### RESTCONF Headers Reference

```
Accept: application/yang-data+json
Content-Type: application/yang-data+json
Accept: application/yang-data+xml
Content-Type: application/yang-data+xml
```

### JSON Payload Example (full interface object)

```json
{
  "ietf-interfaces:interface": {
    "name": "GigabitEthernet2",
    "description": "LAN uplink",
    "type": "iana-if-type:ethernetCsmacd",
    "enabled": true,
    "ietf-ip:ipv4": {
      "address": [
        { "ip": "10.0.0.1", "netmask": "255.255.255.0" }
      ]
    }
  }
}
```

### XML Payload Example (same interface)

```xml
<interface xmlns="urn:ietf:params:xml:ns:yang:ietf-interfaces">
  <name>GigabitEthernet2</name>
  <description>LAN uplink</description>
  <type xmlns:ianaift="urn:ietf:params:xml:ns:yang:iana-if-type">ianaift:ethernetCsmacd</type>
  <enabled>true</enabled>
  <ipv4 xmlns="urn:ietf:params:xml:ns:yang:ietf-ip">
    <address>
      <ip>10.0.0.1</ip>
      <netmask>255.255.255.0</netmask>
    </address>
  </ipv4>
</interface>
```

---

## 5. NETCONF

### Enable NETCONF on Cisco IOS XE (device CLI)

```
configure terminal
netconf-yang
end
```

### Install ncclient

```bash
pip install ncclient
```

### Connecting to a Cisco Device

```python
from ncclient import manager
import os

device = {
    "host": "<DEVICE_IP>",
    "port": 830,
    "username": "<USERNAME>",
    "password": os.environ.get("DEVICE_PASSWORD"),
    "hostkey_verify": False,
    "device_params": {"name": "iosxe"}
}

with manager.connect(**device) as m:
    print("Connected:", m.connected)
```

### Get Configuration

```python
from ncclient import manager

with manager.connect(**device) as m:
    config = m.get_config(source="running")
    print(config.data_xml)
```

### Get Operational State

```python
filter_str = """
<filter>
  <interfaces-state xmlns="urn:ietf:params:xml:ns:yang:ietf-interfaces"/>
</filter>
"""

with manager.connect(**device) as m:
    state = m.get(filter=filter_str)
    print(state.data_xml)
```

### Edit Configuration (create loopback interface)

```python
edit_data = """
<config>
  <native xmlns="http://cisco.com/ns/yang/Cisco-IOS-XE-native">
    <interface>
      <Loopback>
        <name>100</name>
        <description>Created via NETCONF</description>
        <ip>
          <address>
            <primary>
              <address>10.100.100.1</address>
              <mask>255.255.255.255</mask>
            </primary>
          </address>
        </ip>
      </Loopback>
    </interface>
  </native>
</config>
"""

with manager.connect(**device) as m:
    response = m.edit_config(target="running", config=edit_data)
    print(response.xml)
```

### Commit Configuration (for candidate datastore devices)

```python
with manager.connect(**device) as m:
    m.edit_config(target="candidate", config=edit_data)
    m.commit()
```

### Filtered NETCONF Query (get only interface names)

```python
filter_str = """
<filter>
  <interfaces xmlns="urn:ietf:params:xml:ns:yang:ietf-interfaces">
    <interface>
      <name/>
    </interface>
  </interfaces>
</filter>
"""

with manager.connect(**device) as m:
    result = m.get_config(source="running", filter=filter_str)
    print(result.data_xml)
```

---

## 6. YANG

### Basic YANG Model Structure

```yang
module example-vlan {
  namespace "urn:example:vlan";
  prefix "vlan";

  container vlans {
    list vlan {
      key "id";
      leaf id {
        type uint16;
      }
      leaf name {
        type string;
      }
      leaf status {
        type enumeration {
          enum active;
          enum suspended;
        }
      }
    }
  }
}
```

### Containers

```yang
container interface-config {
  leaf name { type string; }
  leaf enabled { type boolean; }
}
```

### Lists

```yang
list interface {
  key "name";
  leaf name { type string; }
  leaf description { type string; }
}
```

### Leaf Nodes

```yang
leaf hostname {
  type string;
  description "Device hostname";
}

leaf mtu {
  type uint16 {
    range "64..9216";
  }
}
```

### Data Types

```yang
leaf enabled { type boolean; }
leaf vlan-id { type uint16; }
leaf ip-address { type string; }   // often typedef'd to inet:ipv4-address
leaf bandwidth { type uint32; }
```

### Configuration vs Operational Data

```yang
// Configuration data (config true - default)
leaf description {
  type string;
  config true;
}

// Operational/state data (read only)
leaf oper-status {
  type string;
  config false;
}
```

### Example Cisco YANG Interaction (query interface oper-status via RESTCONF)

```python
url = f"https://{device_ip}/restconf/data/ietf-interfaces:interfaces-state/interface=GigabitEthernet1"

r = requests.get(url, headers={"Accept": "application/yang-data+json"},
                  auth=("<USERNAME>", os.environ.get("DEVICE_PASSWORD")), verify=False)
print(r.json())
```

```bash
# Discover supported YANG models on a device
pip install pyang
# Or query via NETCONF capabilities exchange (ncclient)
```

```python
with manager.connect(**device) as m:
    for capability in m.server_capabilities:
        print(capability)
```

---

## 7. Cisco APIs and Programmability

### Cisco IOS XE RESTCONF Example (get device hostname)

```python
url = f"https://{device_ip}/restconf/data/Cisco-IOS-XE-native:native/hostname"
r = requests.get(url, headers={"Accept": "application/yang-data+json"},
                  auth=("<USERNAME>", os.environ.get("DEVICE_PASSWORD")), verify=False)
print(r.json())
```

### Cisco DNA Center (Catalyst Center) API Example

```python
import requests
import os

requests.packages.urllib3.disable_warnings()

dnac_ip = "<DNAC_IP>"

# Step 1: Authenticate and get token
auth_url = f"https://{dnac_ip}/dna/system/api/v1/auth/token"
r = requests.post(auth_url, auth=("<USERNAME>", os.environ.get("DNAC_PASSWORD")), verify=False)
token = r.json()["Token"]

# Step 2: Use token to get device list
headers = {"X-Auth-Token": token}
devices_url = f"https://{dnac_ip}/dna/intent/api/v1/network-device"
r = requests.get(devices_url, headers=headers, verify=False)

for device in r.json()["response"]:
    print(device["hostname"], device["managementIpAddress"])
```

### Cisco Meraki API Example

```python
import requests
import os

api_key = os.environ.get("MERAKI_API_KEY")
headers = {"Authorization": f"Bearer {api_key}"}

# Get organizations
r = requests.get("https://api.meraki.com/api/v1/organizations", headers=headers)
orgs = r.json()

org_id = orgs[0]["id"]

# Get networks in the organization
r = requests.get(f"https://api.meraki.com/api/v1/organizations/{org_id}/networks", headers=headers)
for net in r.json():
    print(net["name"], net["id"])

# Get devices in a network
network_id = "<NETWORK_ID>"
r = requests.get(f"https://api.meraki.com/api/v1/networks/{network_id}/devices", headers=headers)
print(r.json())
```

### Cisco Webex API Example (send a notification message)

```python
import requests
import os

bot_token = os.environ.get("WEBEX_BOT_TOKEN")
headers = {
    "Authorization": f"Bearer {bot_token}",
    "Content-Type": "application/json"
}

payload = {
    "roomId": "<ROOM_ID>",
    "text": "Automation Alert: Interface Gi0/1 is DOWN on R1"
}

r = requests.post("https://webexapis.com/v1/messages", headers=headers, json=payload)
print(r.status_code)
```

### Authentication/Token Summary

```python
# DNA Center: Basic Auth -> returns Token -> use X-Auth-Token header
# Meraki: static API key -> Authorization: Bearer <key>
# Webex: static Bot token -> Authorization: Bearer <token>
# IOS XE RESTCONF: Basic Auth on every request (username/password)
# IOS XE NETCONF: SSH username/password on port 830
```

---

## 8. Network Device Automation

### Install Paramiko / Netmiko

```bash
pip install paramiko netmiko
```

### SSH with Paramiko (raw SSH)

```python
import paramiko
import os
import time

client = paramiko.SSHClient()
client.set_missing_host_key_policy(paramiko.AutoAddPolicy())

client.connect(
    hostname="<DEVICE_IP>",
    username="<USERNAME>",
    password=os.environ.get("DEVICE_PASSWORD"),
    look_for_keys=False,
    allow_agent=False
)

shell = client.invoke_shell()
shell.send("terminal length 0\n")
time.sleep(1)
shell.send("show version\n")
time.sleep(2)
output = shell.recv(65535).decode()
print(output)

client.close()
```

### Execute Show Commands with Netmiko

```python
from netmiko import ConnectHandler
import os

device = {
    "device_type": "cisco_ios",
    "host": "<DEVICE_IP>",
    "username": "<USERNAME>",
    "password": os.environ.get("DEVICE_PASSWORD"),
    "secret": os.environ.get("DEVICE_ENABLE_SECRET"),
}

conn = ConnectHandler(**device)
conn.enable()

output = conn.send_command("show ip interface brief")
print(output)

output_version = conn.send_command("show version")
print(output_version)

conn.disconnect()
```

### Configure Cisco Devices with Netmiko

```python
from netmiko import ConnectHandler
import os

device = {
    "device_type": "cisco_ios",
    "host": "<DEVICE_IP>",
    "username": "<USERNAME>",
    "password": os.environ.get("DEVICE_PASSWORD"),
    "secret": os.environ.get("DEVICE_ENABLE_SECRET"),
}

config_commands = [
    "interface GigabitEthernet0/1",
    "description Configured by Netmiko",
    "no shutdown"
]

conn = ConnectHandler(**device)
conn.enable()
output = conn.send_config_set(config_commands)
print(output)

# Save config
conn.save_config()
conn.disconnect()
```

### Multiple-Device Automation

```python
from netmiko import ConnectHandler
import os

devices = [
    {"device_type": "cisco_ios", "host": "192.168.1.1", "username": "<USERNAME>",
     "password": os.environ.get("DEVICE_PASSWORD")},
    {"device_type": "cisco_ios", "host": "192.168.1.2", "username": "<USERNAME>",
     "password": os.environ.get("DEVICE_PASSWORD")},
]

for dev in devices:
    try:
        conn = ConnectHandler(**dev)
        output = conn.send_command("show version | include uptime")
        print(f"--- {dev['host']} ---")
        print(output)
        conn.disconnect()
    except Exception as e:
        print(f"Failed to connect to {dev['host']}: {e}")
```

### Device Inventory Using YAML/JSON

```yaml
# inventory.yml
devices:
  - hostname: R1
    ip: 192.168.1.1
    device_type: cisco_ios
  - hostname: SW1
    ip: 192.168.1.2
    device_type: cisco_ios
```

```python
import yaml
from netmiko import ConnectHandler
import os

with open("inventory.yml") as f:
    inventory = yaml.safe_load(f)

for dev in inventory["devices"]:
    conn = ConnectHandler(
        device_type=dev["device_type"],
        host=dev["ip"],
        username="<USERNAME>",
        password=os.environ.get("DEVICE_PASSWORD"),
    )
    print(conn.send_command("show ip interface brief"))
    conn.disconnect()
```

### Configuration Backup

```python
from netmiko import ConnectHandler
import os
from datetime import datetime

device = {
    "device_type": "cisco_ios",
    "host": "<DEVICE_IP>",
    "username": "<USERNAME>",
    "password": os.environ.get("DEVICE_PASSWORD"),
}

conn = ConnectHandler(**device)
running_config = conn.send_command("show running-config")

timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
filename = f"backup_{device['host']}_{timestamp}.cfg"

with open(filename, "w") as f:
    f.write(running_config)

print(f"Backup saved to {filename}")
conn.disconnect()
```

### Configuration Deployment (push config from a file)

```python
from netmiko import ConnectHandler
import os

device = {
    "device_type": "cisco_ios",
    "host": "<DEVICE_IP>",
    "username": "<USERNAME>",
    "password": os.environ.get("DEVICE_PASSWORD"),
}

conn = ConnectHandler(**device)
output = conn.send_config_from_file("vlan_config.txt")
print(output)
conn.disconnect()
```

---

## 9. Ansible Network Automation

### Install Ansible

```bash
pip install ansible
```

### Inventory

```ini
# inventory.ini
[routers]
R1 ansible_host=192.168.1.1

[switches]
SW1 ansible_host=192.168.1.2

[cisco:children]
routers
switches

[cisco:vars]
ansible_network_os=ios
ansible_connection=network_cli
ansible_user=<USERNAME>
ansible_password="{{ vault_password }}"
ansible_become=yes
ansible_become_method=enable
```

### Variables (group_vars)

```yaml
# group_vars/cisco.yml
ansible_network_os: ios
ansible_connection: network_cli
ansible_user: "<USERNAME>"
ansible_password: "{{ vault_password }}"
```

### Cisco IOS Playbook - Show Commands

```yaml
# show_version.yml
---
- name: Gather show command output
  hosts: cisco
  gather_facts: no
  tasks:
    - name: Run show version
      cisco.ios.ios_command:
        commands:
          - show version
      register: output

    - name: Print output
      debug:
        var: output.stdout_lines
```

### Configuration Changes

```yaml
# config_hostname.yml
---
- name: Set hostname
  hosts: cisco
  gather_facts: no
  tasks:
    - name: Configure hostname
      cisco.ios.ios_config:
        lines:
          - hostname R1-AUTOMATED
```

### VLAN Configuration

```yaml
# vlan_config.yml
---
- name: Configure VLANs
  hosts: switches
  gather_facts: no
  tasks:
    - name: Create VLAN 10
      cisco.ios.ios_vlans:
        config:
          - vlan_id: 10
            name: SALES
          - vlan_id: 20
            name: MARKETING
        state: merged
```

### Interface Configuration

```yaml
# interface_config.yml
---
- name: Configure interfaces
  hosts: switches
  gather_facts: no
  tasks:
    - name: Set interface description and enable
      cisco.ios.ios_interfaces:
        config:
          - name: GigabitEthernet0/1
            description: "Configured by Ansible"
            enabled: true
        state: merged
```

### Static Route Configuration

```yaml
# static_route.yml
---
- name: Configure static route
  hosts: routers
  gather_facts: no
  tasks:
    - name: Add static route
      cisco.ios.ios_static_route:
        config:
          - address_families:
              - afi: ipv4
                routes:
                  - dest: 10.0.0.0/24
                    next_hops:
                      - forward_router_address: 192.168.1.254
        state: merged
```

### IOS Backup

```yaml
# backup_config.yml
---
- name: Backup running config
  hosts: cisco
  gather_facts: no
  tasks:
    - name: Backup
      cisco.ios.ios_config:
        backup: yes
        backup_options:
          filename: "{{ inventory_hostname }}_backup.cfg"
          dir_path: ./backups
```

### Jinja2 Templates

```jinja
{# vlan_template.j2 #}
{% for vlan in vlans %}
vlan {{ vlan.id }}
 name {{ vlan.name }}
{% endfor %}
```

```yaml
# deploy_template.yml
---
- name: Deploy VLANs from template
  hosts: switches
  gather_facts: no
  vars:
    vlans:
      - id: 10
        name: SALES
      - id: 20
        name: MARKETING
  tasks:
    - name: Render config
      template:
        src: vlan_template.j2
        dest: ./generated_vlans.cfg

    - name: Push config
      cisco.ios.ios_config:
        src: ./generated_vlans.cfg
```

### Loops

```yaml
- name: Create multiple VLANs with a loop
  cisco.ios.ios_vlans:
    config:
      - vlan_id: "{{ item.id }}"
        name: "{{ item.name }}"
    state: merged
  loop:
    - { id: 10, name: SALES }
    - { id: 20, name: MARKETING }
```

### Conditionals

```yaml
- name: Configure only if hostname matches
  cisco.ios.ios_config:
    lines:
      - "logging host 10.0.0.5"
  when: inventory_hostname == "R1"
```

### Handlers

```yaml
# playbook with handler
---
- name: Update config and reload if changed
  hosts: cisco
  gather_facts: no
  tasks:
    - name: Change banner
      cisco.ios.ios_config:
        lines:
          - "banner motd ^Automated Access Only^"
      notify: Save config

  handlers:
    - name: Save config
      cisco.ios.ios_config:
        save_when: modified
```

---

## 10. Nornir

### Installation

```bash
pip install nornir nornir_netmiko nornir_utils
```

### Inventory

```yaml
# inventory/hosts.yaml
R1:
  hostname: 192.168.1.1
  platform: ios
  groups:
    - cisco_devices

SW1:
  hostname: 192.168.1.2
  platform: ios
  groups:
    - cisco_devices
```

```yaml
# inventory/groups.yaml
cisco_devices:
  username: "<USERNAME>"
  password: "{{ DEVICE_PASSWORD }}"
```

```yaml
# config.yaml
inventory:
  plugin: SimpleInventory
  options:
    host_file: "inventory/hosts.yaml"
    group_file: "inventory/groups.yaml"
runner:
  plugin: threaded
  options:
    num_workers: 10
```

### Connecting to Devices

```python
from nornir import InitNornir

nr = InitNornir(config_file="config.yaml")
print(nr.inventory.hosts.keys())
```

### Running Commands

```python
from nornir import InitNornir
from nornir_netmiko.tasks import netmiko_send_command
from nornir_utils.plugins.functions import print_result

nr = InitNornir(config_file="config.yaml")

result = nr.run(
    task=netmiko_send_command,
    command_string="show ip interface brief"
)

print_result(result)
```

### Configuration Deployment

```python
from nornir import InitNornir
from nornir_netmiko.tasks import netmiko_send_config
from nornir_utils.plugins.functions import print_result

nr = InitNornir(config_file="config.yaml")

config_commands = [
    "interface Loopback99",
    "description Configured by Nornir",
]

result = nr.run(
    task=netmiko_send_config,
    config_commands=config_commands
)

print_result(result)
```

### Multiple-Device Automation (filtering)

```python
from nornir import InitNornir
from nornir_netmiko.tasks import netmiko_send_command
from nornir_utils.plugins.functions import print_result

nr = InitNornir(config_file="config.yaml")

# Only run against devices in the cisco_devices group
switches = nr.filter(platform="ios")

result = switches.run(
    task=netmiko_send_command,
    command_string="show vlan brief"
)

print_result(result)
```

---

## 11. Git and Automation

```bash
# Initialize a new repo for your automation project
git init

# Clone an existing repo
git clone https://github.com/<USERNAME>/network-automation.git

# Stage files
git add inventory.yml backup_script.py

# Commit changes
git commit -m "Add device inventory and backup script"

# Create/switch branch
git branch feature/vlan-automation
git checkout feature/vlan-automation
# or in one step
git checkout -b feature/vlan-automation

# Pull latest changes
git pull origin main

# Push changes
git push origin feature/vlan-automation
```

### Example Network Automation Project Structure

```
network-automation/
├── inventory/
│   ├── hosts.yaml
│   └── groups.yaml
├── scripts/
│   ├── backup_configs.py
│   ├── vlan_automation.py
│   └── device_facts.py
├── playbooks/
│   ├── vlan_config.yml
│   └── interface_config.yml
├── templates/
│   └── vlan_template.j2
├── backups/
├── tests/
│   └── test_backup.py
├── requirements.txt
├── .env.example
├── .gitignore
└── README.md
```

```gitignore
# .gitignore
.env
*.cfg
backups/
__pycache__/
*.pyc
.venv/
```

---

## 12. Docker for Network Automation

### Dockerfile

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "backup_configs.py"]
```

### requirements.txt

```
netmiko==4.3.0
ncclient==0.6.15
requests==2.31.0
pyyaml==6.0.1
ansible==9.2.0
```

### Build Image

```bash
docker build -t network-automation:latest .
```

### Run Container

```bash
docker run --rm network-automation:latest
```

### Environment Variables

```bash
docker run --rm \
  -e DEVICE_PASSWORD=<PASSWORD> \
  -e DEVICE_ENABLE_SECRET=<SECRET> \
  network-automation:latest
```

### Docker Compose Example

```yaml
# docker-compose.yml
version: "3.9"
services:
  automation:
    build: .
    env_file:
      - .env
    volumes:
      - ./backups:/app/backups
      - ./inventory:/app/inventory
```

```bash
docker compose up --build
```

---

## 13. CI/CD for Network Automation

### Basic GitHub Actions Workflow

```yaml
# .github/workflows/ci.yml
name: Network Automation CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Run linter
        run: pip install flake8 && flake8 scripts/

      - name: Run tests
        run: pytest tests/
```

### Python Testing (part of pipeline)

```yaml
- name: Run pytest with coverage
  run: |
    pip install pytest pytest-cov
    pytest --cov=scripts tests/
```

### Linting

```bash
pip install flake8 black
flake8 scripts/
black --check scripts/
```

### Automated Network Configuration Validation

```yaml
- name: Validate YAML inventory
  run: |
    pip install yamllint
    yamllint inventory/

- name: Validate config templates render correctly
  run: python scripts/validate_templates.py
```

---

## 14. Testing and Security

### pytest Examples

```python
# tests/test_vlan_builder.py
import pytest
from scripts.vlan_automation import build_vlan_config

def test_build_vlan_config_returns_list():
    result = build_vlan_config(10, "SALES")
    assert isinstance(result, list)

def test_build_vlan_config_contains_vlan_id():
    result = build_vlan_config(10, "SALES")
    assert "vlan 10" in result

def test_build_vlan_config_invalid_id_raises():
    with pytest.raises(ValueError):
        build_vlan_config(-1, "BAD")
```

```bash
pytest tests/ -v
```

### API Testing

```python
import requests
from unittest.mock import patch

def test_get_devices_success():
    with patch("requests.get") as mock_get:
        mock_get.return_value.status_code = 200
        mock_get.return_value.json.return_value = {"devices": []}

        response = requests.get("https://api.example.com/devices")
        assert response.status_code == 200
        assert "devices" in response.json()
```

### Input Validation

```python
import ipaddress

def validate_ip(ip: str) -> bool:
    try:
        ipaddress.ip_address(ip)
        return True
    except ValueError:
        return False

def validate_vlan_id(vlan_id: int) -> bool:
    return 1 <= vlan_id <= 4094

assert validate_ip("192.168.1.1") is True
assert validate_ip("999.999.999.999") is False
assert validate_vlan_id(10) is True
assert validate_vlan_id(5000) is False
```

### Environment Variables for Secrets

```bash
pip install python-dotenv
```

```python
from dotenv import load_dotenv
import os

load_dotenv()  # loads variables from .env file

username = os.environ.get("DEVICE_USERNAME")
password = os.environ.get("DEVICE_PASSWORD")

if not password:
    raise EnvironmentError("DEVICE_PASSWORD not set")
```

### .env Example

```
# .env  (NEVER commit this file — add to .gitignore)
DEVICE_USERNAME=automation_user
DEVICE_PASSWORD=changeme123
DEVICE_ENABLE_SECRET=changeme456
MERAKI_API_KEY=your_meraki_key_here
DNAC_PASSWORD=changeme789
WEBEX_BOT_TOKEN=your_webex_token_here
```

### Secure Credential Handling

```python
import os
import getpass

# Prefer environment variables
password = os.environ.get("DEVICE_PASSWORD")

# Fallback: prompt securely without echoing input
if not password:
    password = getpass.getpass("Enter device password: ")

# Never do this:
# password = "cisco123"   # BAD - hardcoded secret
```

### Basic Logging

```python
import logging

logging.basicConfig(
    filename="automation.log",
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s"
)

logger = logging.getLogger(__name__)

logger.info("Starting backup job")
try:
    raise ConnectionError("Simulated failure")
except ConnectionError as e:
    logger.error(f"Backup failed: {e}")
```

---

## 15. Complete CCNA Automation Projects

### Project A: Python Cisco Device Automation (Netmiko multi-device facts)

```python
#!/usr/bin/env python3
import os
import yaml
from netmiko import ConnectHandler
from datetime import datetime

def load_inventory(path="inventory.yml"):
    with open(path) as f:
        return yaml.safe_load(f)["devices"]

def gather_facts(device):
    conn = ConnectHandler(
        device_type=device["device_type"],
        host=device["ip"],
        username=os.environ["DEVICE_USERNAME"],
        password=os.environ["DEVICE_PASSWORD"],
    )
    facts = {
        "hostname": device["hostname"],
        "version": conn.send_command("show version | include Version"),
        "interfaces": conn.send_command("show ip interface brief"),
        "timestamp": datetime.now().isoformat(),
    }
    conn.disconnect()
    return facts

if __name__ == "__main__":
    devices = load_inventory()
    for dev in devices:
        try:
            facts = gather_facts(dev)
            print(facts)
        except Exception as e:
            print(f"Error on {dev['hostname']}: {e}")
```

### Project B: RESTCONF VLAN Automation

```python
#!/usr/bin/env python3
import requests
import os

requests.packages.urllib3.disable_warnings()

def create_vlan(device_ip, vlan_id, vlan_name):
    url = f"https://{device_ip}/restconf/data/Cisco-IOS-XE-native:native/vlan"
    headers = {
        "Content-Type": "application/yang-data+json",
        "Accept": "application/yang-data+json"
    }
    payload = {"Cisco-IOS-XE-native:vlan-list": [{"id": vlan_id, "name": vlan_name}]}

    r = requests.post(
        url, headers=headers, json=payload,
        auth=(os.environ["DEVICE_USERNAME"], os.environ["DEVICE_PASSWORD"]),
        verify=False
    )
    r.raise_for_status()
    return r.status_code

if __name__ == "__main__":
    status = create_vlan("<DEVICE_IP>", 30, "GUEST")
    print(f"VLAN creation status: {status}")
```

### Project C: NETCONF Configuration Automation

```python
#!/usr/bin/env python3
from ncclient import manager
import os

def configure_loopback(device_ip, loopback_id, ip_addr, mask):
    device = {
        "host": device_ip,
        "port": 830,
        "username": os.environ["DEVICE_USERNAME"],
        "password": os.environ["DEVICE_PASSWORD"],
        "hostkey_verify": False,
        "device_params": {"name": "iosxe"},
    }

    config_xml = f"""
    <config>
      <native xmlns="http://cisco.com/ns/yang/Cisco-IOS-XE-native">
        <interface>
          <Loopback>
            <name>{loopback_id}</name>
            <ip>
              <address>
                <primary>
                  <address>{ip_addr}</address>
                  <mask>{mask}</mask>
                </primary>
              </address>
            </ip>
          </Loopback>
        </interface>
      </native>
    </config>
    """

    with manager.connect(**device) as m:
        response = m.edit_config(target="running", config=config_xml)
        return response.xml

if __name__ == "__main__":
    result = configure_loopback("<DEVICE_IP>", 200, "10.200.200.1", "255.255.255.255")
    print(result)
```

### Project D: Ansible Multi-Device Configuration

```yaml
# site.yml
---
- name: Configure all Cisco devices
  hosts: cisco
  gather_facts: no
  tasks:
    - name: Set VLANs on switches
      cisco.ios.ios_vlans:
        config:
          - vlan_id: 10
            name: SALES
        state: merged
      when: "'switches' in group_names"

    - name: Configure interfaces
      cisco.ios.ios_interfaces:
        config:
          - name: GigabitEthernet0/1
            description: "Managed by Ansible"
            enabled: true
        state: merged

    - name: Save running config
      cisco.ios.ios_config:
        save_when: modified
```

### Project E: Automated Configuration Backup (multi-device, timestamped)

```python
#!/usr/bin/env python3
import os
import yaml
from netmiko import ConnectHandler
from datetime import datetime
import logging

logging.basicConfig(level=logging.INFO, format="%(asctime)s %(message)s")
logger = logging.getLogger(__name__)

def backup_device(device):
    conn = ConnectHandler(
        device_type=device["device_type"],
        host=device["ip"],
        username=os.environ["DEVICE_USERNAME"],
        password=os.environ["DEVICE_PASSWORD"],
    )
    config = conn.send_command("show running-config")
    conn.disconnect()

    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    filename = f"backups/{device['hostname']}_{timestamp}.cfg"
    os.makedirs("backups", exist_ok=True)

    with open(filename, "w") as f:
        f.write(config)

    logger.info(f"Backed up {device['hostname']} -> {filename}")

if __name__ == "__main__":
    with open("inventory.yml") as f:
        devices = yaml.safe_load(f)["devices"]

    for dev in devices:
        try:
            backup_device(dev)
        except Exception as e:
            logger.error(f"Failed to backup {dev['hostname']}: {e}")
```

### Project F: API-Driven Network Monitoring

```python
#!/usr/bin/env python3
import requests
import os
import time
import logging

requests.packages.urllib3.disable_warnings()
logging.basicConfig(level=logging.INFO, format="%(asctime)s %(message)s")
logger = logging.getLogger(__name__)

def get_interface_status(device_ip, interface_name):
    url = f"https://{device_ip}/restconf/data/ietf-interfaces:interfaces-state/interface={interface_name}"
    headers = {"Accept": "application/yang-data+json"}

    r = requests.get(
        url, headers=headers,
        auth=(os.environ["DEVICE_USERNAME"], os.environ["DEVICE_PASSWORD"]),
        verify=False, timeout=5
    )
    r.raise_for_status()
    data = r.json()
    return data["ietf-interfaces:interface"]["oper-status"]

def monitor(device_ip, interface_name, interval=30):
    while True:
        try:
            status = get_interface_status(device_ip, interface_name)
            logger.info(f"{interface_name} status: {status}")
            if status != "up":
                logger.warning(f"ALERT: {interface_name} is {status}")
        except Exception as e:
            logger.error(f"Monitoring error: {e}")
        time.sleep(interval)

if __name__ == "__main__":
    monitor("<DEVICE_IP>", "GigabitEthernet1", interval=30)
```

---

## Quick Reference: Required Cisco IOS XE Enablement Commands

```
! Enable HTTPS server (required for RESTCONF)
ip http secure-server

! Enable RESTCONF
restconf

! Enable NETCONF over SSH
netconf-yang

! Create automation user with full privileges
username <USERNAME> privilege 15 algorithm-type scrypt secret <PASSWORD>

! Ensure SSH is configured
ip domain-name lab.local
crypto key generate rsa modulus 2048
line vty 0 4
 transport input ssh
 login local
```
