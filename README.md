# firewall_multi
<!-- DO NOT EDIT: This document was generated by rake docs -->

[![Build Status](https://img.shields.io/travis/alexharv074/puppet-firewall_multi.svg)](https://travis-ci.org/alexharv074/puppet-firewall_multi)

#### Table of contents

1. [Overview](#overview)
2. [Version compatibility](#version-compatibility)
3. [Setup](#setup)
    * [What firewall_multi affects](#what-firewall_multi-affects)
    * [Setup requirements](#setup-requirements)
    * [Beginning with firewall_multi](#beginning-with-firewall_multi)
    * [Upgrading](#upgrading)
4. [Reference](#reference)
    * [Defined types](#defined-types)
        - [firewall_multi](#firewall_multi)
    * [Functions](#functions)
        - [firewall_multi](#firewall_multi-2)
    * [Examples](#examples)
        - [Array of sources](#array-of-sources)
        - [Arrays of sources and destinations](#arrays-of-sources-and-destinations)
        - [Array of protocols](#array-of-protocols)
        - [Array of ICMP types](#array-of-icmp-types)
        - [Array of providers](#array-of-providers)
    * [Use with Hiera](#use-with-hiera)
    * [The alias lookup](#the-alias-lookup)
5. [Development](#development)
    * [Testing](#testing)
    * [Release](#release)
6. [Troubleshooting](#troubleshooting)
    * ["Error: no parameter named X"](#error-no-parameter-named-x)
    * [Resolution 1 - ensure you have the right version](#resolution-1---ensure-you-have-the-right-version)
    * [Resolution 2 - environment isolation](#resolution-2---environment-isolation)
7. [Donate](#donate)

## Overview

The `firewall_multi` module provides a defined type wrapper for spawning [puppetlabs/firewall](https://github.com/puppetlabs/puppetlabs-firewall) resources for arrays of certain inputs. This is useful at large sites that may have many networks, due to the puppetlabs-firewall module lacking functionality to allow arrays for certain inputs. The limitation is due to the underlying Linux iptables command, which also only allows arrays for certain inputs.

(For more information about the history and motivation for this project, see [MODULES-3066](https://tickets.puppetlabs.com/browse/MODULES-3066) in the Puppet Jira.)

At present the following inputs can be arrays:

* source
* destination
* proto
* icmp
* provider

## Version compatibility

Each release of the firewall_multi module is compatible with a specific release of puppetlabs-firewall, starting at firewall v1.8.0. Earlier versions of the firewall module are not supported.

firewall_multi|firewall
--------------|--------
earlier|1.8.0
1.7.0|1.8.0
1.7.0|1.8.1
1.8.0|1.8.2
1.9.0|1.9.0
1.10.1|1.10.0
1.10.1|1.11.0
1.10.1|1.12.0
1.10.1|1.13.0
1.10.1|1.14.0
1.11.0|1.10.0
1.11.0|1.11.0
1.11.0|1.12.0
1.11.0|1.13.0
1.11.0|1.14.0
1.12.0|1.15.0
1.12.0|1.15.1
1.13.0|1.15.2
1.13.0|1.15.3
1.13.1|1.15.3
1.13.2|1.15.3
1.14.0|2.0.0
1.14.1|2.0.0
1.15.0|2.1.0
1.16.0|2.2.0
1.17.0|2.3.0

Note that Puppet 3 support was dropped in version 1.11.0.

## Setup

### What firewall_multi affects

The scope is the same as with the firewall module.

### Setup requirements

The firewall_multi module's only dependency is the firewall module.

### Beginning with firewall_multi

It is expected that a standard set up for the firewall module is followed, in particular with respect to the purging of firewall resources. If, for instance, addresses are removed from an array of sources, the corresponding firewall resources would only be removed if purging is enabled. This might be surprising in a way that impacts security.

Otherwise, usage of the firewall_multi defined type is the same as with the firewall custom type, the only exceptions being that some parameters optionally accept arrays.

### Upgrading

Firstly, ensure you have read the version compatibility matrix section above before upgrading as versions of this module sometimes must be kept in sync with the firewall module.

To upgrade the module, use the puppet module tool as normal:

~~~ text
puppet module upgrade alexharvey/firewall_multi
~~~

## Reference

**Defined types**

* [`firewall_multi`](#firewall_multi): A defined type wrapper for spawning
[puppetlabs/firewall](https://github.com/puppetlabs/puppetlabs-firewall)
resources for arrays of certain inputs.

**Functions**

* [`firewall_multi`](#firewall_multi): Take a name and a hash and return a modified hash suitable
for input to create_resources().

### Defined types

#### firewall_multi

A defined type wrapper for spawning
[puppetlabs/firewall](https://github.com/puppetlabs/puppetlabs-firewall)
resources for arrays of certain inputs.

##### Parameters

The following parameters are available in the `firewall_multi` defined type.

###### `source`

Data type: `Array`

An array of source IPs or CIDRs.

Default value: `undef`

###### `destination`

Data type: `Array`

An array of destination IPs or CIDRs.

Default value: `undef`

###### `proto`

Data type: `Array`

An array of protocols.

Default value: `undef`

###### `icmp`

Data type: `Array`

An array of ICMP types.

Default value: `undef`

###### `provider`

Data type: `Array`

An array of providers.

Default value: `undef`

### Functions

#### firewall_multi

Type: Ruby 4.x API

Convert firewall_multi type data to firewall type data.

* **Note** Given this input:

```ruby
[
  {
    '00100 accept inbound ssh' => {
      'action' => 'accept',
      'source' => ['1.1.1.1/24', '2.2.2.2/24'],
      'dport'  => 22,
    },
  }
]
```

Return this:

```ruby
{
  '00100 accept inbound ssh from 1.1.1.1/24' => {
    'action' => 'accept',
    'source' => '1.1.1.1/24',
    'dport'  => 22,
  },
  '00100 accept inbound ssh from 2.2.2.2/24' => {
    'action' => 'accept',
    'source' => '2.2.2.2/24',
    'dport'  => 22,
  },
}
```

##### `firewall_multi(Hash $hash)`

Convert firewall_multi type data to firewall type data.

Returns: `Any` Modified Hash for firewall types to be passed to create_resources().

###### `hash`

Data type: `Hash`

The original resource params data.

### Examples

#### Array of sources

```puppet
firewall_multi { '100 allow http and https access':
  source => [
    '10.0.10.0/24',
    '10.0.12.0/24',
    '10.1.1.128',
  ],
  dport  => [80, 443],
  proto  => tcp,
  action => accept,
}
```

This will cause three resources to be created:

* Firewall['100 allow http and https access from 10.0.10.0/24']
* Firewall['100 allow http and https access from 10.0.12.0/24']
* Firewall['100 allow http and https access from 10.1.1.128']

#### Arrays of sources and destinations

```puppet
firewall_multi { '100 allow http and https access':
  source => [
    '10.0.10.0/24',
    '10.0.12.0/24',
  ],
  destination => [
    '10.2.0.0/24',
    '10.3.0.0/24',
  ],
  dport  => [80, 443],
  proto  => tcp,
  action => accept,
}
```

This will cause four resources to be created:

* Firewall['100 allow http and https access from 10.0.10.0/24 to 10.2.0.0/24']
* Firewall['100 allow http and https access from 10.0.10.0/24 to 10.3.0.0/24']
* Firewall['100 allow http and https access from 10.0.12.0/24 to 10.2.0.0/24']
* Firewall['100 allow http and https access from 10.0.12.0/24 to 10.3.0.0/24']

#### Array of protocols

```puppet
firewall_multi { '100 allow DNS lookups':
  dport  => 53,
  proto  => ['tcp', 'udp'],
  action => 'accept',
}
```

This will cause two resources to be created:

* Firewall['100 allow DNS lookups protocol tcp']
* Firewall['100 allow DNS lookups protocol udp']

#### Array of ICMP types

```puppet
firewall_multi { '100 accept icmp output':
  chain  => 'OUTPUT',
  proto  => 'icmp',
  action => 'accept',
  icmp   => [0, 8],
}
```

This will cause two resources to be created:

* Firewall['100 accept icmp output icmp type 0']
* Firewall['100 accept icmp output icmp type 8']

#### Array of providers

Open a firewall for IPv4 and IPv6 on a web server:

```puppet
firewall_multi { '100 allow http and https access':
  dport    => [80, 443],
  proto    => 'tcp',
  action   => 'accept',
  provider => ['ip6tables', 'iptables'],
}
```

This will cause two resources to be created:

* Firewall['100 allow http and https access using provider ip6tables']
* Firewall['100 allow http and https access using provider iptables']

### Use with Hiera

Some users may prefer to externalise firewall resources in Hiera. For example:

```yaml
---
myclass::firewall_multis:
  '00099 accept tcp port 22 for ssh':
    dport: '22'
    action: 'accept'
    proto: 'tcp'
    source:
      - 10.0.0.3/32
      - 10.10.0.0/26
```

Meanwhile, manifest code would look something like this:

```puppet
class myclass (
  Hash[String, Hash] $firewall_multis,
) {
  $firewall_multis.each |$name, $firewall_multi| {
    firewall_multi { $name:
      * => $firewall_multi
    }
  }
  ...
}
```

Or:

```puppet
class myclass (
  Hash[String, Hash] $firewall_multis,
) {
  create_resources(firewall_multi, $firewall_multis)
  ...
}
```

### The alias lookup

Those who externalise firewall resources in Hiera should be aware of the [alias lookup function](https://docs.puppet.com/hiera/3.0/variables.html#the-alias-lookup-function), which makes it possible to define networks as arrays in Hiera and then look these up from within the `firewall_multi` definitions.

The following examples show how to do that:

```yaml
---
mylocaldomains:
  - 10.0.0.3/32
  - 10.10.0.0/26
myotherdomains:
  - 172.0.1.0/26

myclass::firewall_multis:
  '00099 accept tcp port 22 for ssh':
    dport: '22'
    action: 'accept'
    proto: 'tcp'
    source: "%{alias('mylocaldomains')}"
  '00200 accept tcp port 80 for http':
    dport: '80'
    action: 'accept'
    proto: 'tcp'
    source: "%{alias('myotherdomains')}"
```

## Development

Please read CONTRIBUTING.md before contributing.

### Testing

Make sure you have:

* rake
* bundler

Install the necessary gems:

    bundle install

To run the tests from the root of the source code:

    bundle exec rake spec

To run the acceptance tests:

    BEAKER_set=centos-72-x64 bundle exec rspec spec/acceptance

### Release

This module uses Puppet Blacksmith to publish to the Puppet Forge.

Ensure you have these lines in `~/.bash_profile`:

    export BLACKSMITH_FORGE_URL=https://forgeapi.puppetlabs.com
    export BLACKSMITH_FORGE_USERNAME=alexharvey
    export BLACKSMITH_FORGE_PASSWORD=xxxxxxxxx

Build the module and push to Forge:

    bundle exec rake module:push

Clean the pkg dir (otherwise Blacksmith will try to push old copies to Forge next time you run it and it will fail):

    bundle exec rake module:clean

## Troubleshooting

### "Error: no parameter named X"

On [occasion](https://github.com/alexharv074/puppet-firewall_multi/issues/19), users of this module have reported confusing failure error messages like:

```text
Error: no parameter named 'ipvs'
```

Sometimes seen after upgrading.

### Resolution 1 - ensure you have the right version

The error message is probably not a bug in this module. Firstly, ensure that you have checked the [version compatibility](#version-compatibility) matrix above, and have installed a compatible combination of firewall/firewall_multi.

### Resolution 2 - environment isolation

While not a problem with this module, this module - due to its requirement to be pinned against a very specific version of puppetlabs/firewall - is a likely candidate for failing if you have not properly set up [environment isolation](https://puppet.com/docs/puppet/6.4/environment_isolation.html). Follow the docs in the previous link to resolve the issue. Also, be aware of how to use r10k to automatically [generate types](https://github.com/puppetlabs/r10k/blob/master/doc/faq.mkd#how-can-run-i-puppet-generate-types-for-each-changed-environment-during-deployment) during deployments.

## Donate

If you find that this code saved your project some significant time, consider donating:

[![paypal](https://www.paypalobjects.com/en_AU/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=6849RBYT6VYBQ)

Also, please add a star if you find it useful!
