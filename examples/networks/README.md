# Networks
The 'kvm_networks' variable contains a list of networks to create and/or modify.

The network configuration is based on the [Libvirt XML configuration](https://libvirt.org/formatnetwork.html).

## Variables

Following Libvirt XML elements are supported:

```yaml
kvm_networks:
  - name: string    # Name of the network
    title: string   # Title for the network (OPTIONAL)
    description: string   # Description for network (OPTIONAL)
    mtu: int        # Maximum Transmission Unit for network (OPTIONAL)
    ipv6: string    # Enable or disable IPv6 networking, 'yes' or 'no' (OPTIONAL)
    port: dict()    # Configure isolated ports (isolate traffic between guests) (OPTIONAL)
    bridge: dict()  # Configuration of bridge device for network (OPTIONAL)
    forward: dict() # Configuration of physical LAN connection (OPTIONAL)
    ip: list()      # IPv4 and/or IPv6 networks to define on network
    route: list()   # Static route configuration (OPTIONAL)
    bandwidth: dict() # Configure Quality Of Service (OPTIONAL)
    domain: dict()  # Defines the DHCP DNS domain (OPTIONAL)
    dns: dict()     # DNS Server setup for network, enabled by default! (OPTIONAL)
```

The [network.xml.j2](templates/network.xml.j2) is used to convert each network configuration to an XML file. 

## Examples

Below examples provide an overview of the common options for each supported element.
The element configurations are not limited to what is shown here.

The [Libvirt XML documentation](https://libvirt.org/formatnetwork.html) contains an overview 
of all available options for the supported elements.

**Note:** Real life examples can be found in the YAML files located in this directory. 

### Port

```yaml
kvm_networks:
  - name: default
    title: Default Network
    description: Default network
    mtu: 1500
    ipv6: 'yes'
    ip:
      - family: ipv4
        address: 192.168.122.1
        netmask: 255.255.255.0
      - family: ipv6
        address: 2001:db8:ca2:2::1
        prefix: 64
    port:
      isolated: 'yes'
```

### Bridge

```yaml
kvm_networks:
  - name: default
    ip:
      - family: ipv4
        address: 192.168.122.1
        netmask: 255.255.255.0
    bridge:
      name: virbr0
      stp: 'yes'
      delay: 5
      zone: libvirt
      macTableManager: libvirt
```

### Forward

```yaml
kvm_networks:
  # NAT Forwarding
  - name: nat
    ip:
      - family: ipv4
        address: 192.168.122.1
        netmask: 255.255.255.0
    forward:
      mode: nat
      dev: eth0
      nat:
        address:
          start: 1.2.3.4
          end: 1.2.3.10
        port:
          start: 500
          end: 1000
        ipv6: 'yes'

  # Routed Forwarding
  - name: route
    ip:
      - family: ipv4
        address: 192.168.123.1
        netmask: 255.255.255.0
    forward:
      mode: route
      dev: eth1

  # Open Forwarding
  - name: open
    ip:
      - family: ipv4
        address: 192.168.124.1
        netmask: 255.255.255.0
    forward:
      mode: open
```

**Note:** If no forward element is specified the network will be isolated.

### IP

```yaml
kvm_networks:
  - name: default
    ip:
      - family: ipv4
        address: 192.168.122.1
        netmask: 255.255.255.0
        local_ptr: 'yes'
        tftp: /tftp
        dhcp:
          range:
            start: 192.168.122.100
            end: 192.168.122.200
            lease: 60 # Minutes
          host:
            - mac: 00:16:3e:77:2e:ed
              name: foobar.example.com
              ip: 192.168.122.10
              lease: 30 # Minutes
          bootp:
            file: efi/BOOTX64.efi
            server: 192.168.1.100
      - family: ipv6
        address: 2001:db8:ca2:2::1
        prefix: 64
        local_ptr: 'yes'
        dhcp:
          range:
            start: 2001:db8:ca2:2::10
            end: 2001:db8:ca2:2::ff
            lease: 60 # Minutes
          host:
            - id: 0:1:0:1:18:aa:62:fe:0:16:3e:44:55:66
              name: paul
              ip: 2001:db8:ca2:2:3::2
              lease: 0 # Infinite
```

### Route

```yaml
kvm_networks:
  - name: default
    ipv6: 'yes'
    ip:
      - family: ipv4
        address: 192.168.122.1
        netmask: 255.255.255.0
      - family: ipv6
        address: 2001:db8:ca2:2::1
        prefix: 64
    route:
      - family: ipv4
        address: 192.168.222.0
        netmask: 255.255.255.0
        gateway: 192.168.122.2
        metric: 2
      - family: ipv6
        address: '2001:db8:ca2:3::'
        prefix: 64
        gateway: 2001:db8:ca2:2::2
        metric: 2
```

### Bandwith

```yaml
kvm_networks:
  - name: default
    ip:
      - family: ipv4
        address: 192.168.122.1
        netmask: 255.255.255.0
    bandwidth:
      inbound:
        avarage: 1000
        peak: 5000
        burst: 5120
        floor: 256
      outbound:
        avarage: 128
        peak: 256
        burst: 256
```

### Domain

```yaml
kvm_networks:
  - name: default
    ip:
      - family: ipv4
        address: 192.168.122.1
        netmask: 255.255.255.0
    domain:
      name: example.com
      localOnly: 'yes'
      register: 'yes'
```

### DNS

```yaml
kvm_networks:
  - name: default
    ip:
      - family: ipv4
        address: 192.168.122.1
        netmask: 255.255.255.0
    dns:
      enable: 'yes'
      forward_plain_names: 'no'
      forwarder:
        - addr: 8.8.8.8
          domain: example.com
      txt:
        - name: example
          value: example value
      hosts:
        - ip: 192.1468.122.2
          hostname:
            - myhost
            - myhostalias
      srv:
        - service: name
          protocol: tcp
          target: '.'
          port: 1024
          priority: 10
          weight: 10
          domain: test-domain-name
```

**Note:** The libvirt DNS server is started by default!