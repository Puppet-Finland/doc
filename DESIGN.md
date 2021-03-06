Defining organizational policies
================================

Organization policies are handled primarily with class parameters. This approach has the advantage of working regardless of the node classifier being used:

  * Classic node definitions (site.pp)
  * Hiera
  * ENC (e.g. LDAP, Foreman)

Global variables are used to provide default values for commonly repeating class parameters. Examples:

  * $serveradmin = 'admin@domain.com'
  * $servermonitor = 'monit@domain.com'
  * $ldap_host = 'ldap.domain.com'

Managing operating system differences
=====================================

Using params subclass
---------------------

Most operating system differences can be abstracted away using a *module::params* subclass. The purpose of the class is to convert things like file paths, package names and service names into variables, which can then be used directly in the Puppet classes. All other classes in the module inherit the params class. A typical example looks like this:

    class sshd::params {
    
        case $::osfamily {
            'RedHat': {
                $package_name = 'openssh-server'
                $service_name = 'sshd'
                $service_start = "/sbin/service $service_name start"
                $service_stop = "/sbin/service $service_name stop"
            }
            'Debian': {
                $package_name = 'openssh-server'
                $service_name = 'ssh'
                $service_start = "/usr/sbin/service $service_name start"
                $service_stop = "/usr/sbin/service $service_name stop"
            }
            'FreeBSD': {
                $service_name = 'sshd'
                $service_command = "/usr/sbin/service $service_name"
                $service_start = "/etc/rc.d/$service_name start"
                $service_stop = "/etc/rc.d/$service_name stop"
            }
            default:
                fail("Unsupported OS: ${::osfamily}")
            }
        }
    }

The *modulenaname::params* approach has several benefits:

  * It isolates operating system -specific information into a single class, making the rest of the Puppet code much cleaner.
  * Trying to include the module on an unsupported operating system (see the "default" block) will result in clean, early failure instead of a misconfiguration or partial configuration.


Conditionals in entrypoint class
--------------------------------

Use of a params class is not always enough to abstract away operating system differences. In particular

  * Some operating systems can't (in practice) be configured as fully as others.
  * Some operatings system require additional configuration steps.

A real-life example is shown below:

    # Do not attempt to configure postfix on OpenSuSE because of YaST
    if $::operatingsystem == 'OpenSuSE' {
        include postfix::install
        include postfix::service
    
    # On SLES we have to contend ourselves with only installing the package
    elsif $::operatingsystem == 'SLES' {
        include postfix::install
    }
    
    # On all sane operating systems we can configure everything
    else {
        include postfix::install

        class { 'postfix::config':
            ...
        }
        
        include postfix::service
        
        # FreeBSD requires additional configuration
        if $::operatingsystem == 'FreeBSD' {
            include postfix::config::freebsd
        }
    }

The two-pronged approach to handling operating system differences - params class and conditional statements - has proven itself in practice.

Managing configuration complexity
=================================

Some applications are by their nature very large and flexible. Trying to cater for every possible use-case using a single Puppet module would result in huge mess due to astronomic number of configuration options and their combinations. Indeed, some of the more widely used Puppet modules suffer from this "API explosion" which causes many issues:

  * Following the logic of the module becomes very difficult
  * Changes to the code in one place can easily break some other (corner) use-case

While it is possible to counter these negative effects with strict testing and release procedures, the underlying problem does not go away and the module will eventually break for some people/use-cases.

The Puppet-Finland modules attempt to prevent API explosions using the three strategies described below.

Parameterization
----------------

Configuration options that tend to change from site to site like "Allow SSH root login" in sshd can be safely parametrized. As long as only the most common changes are parameterized the API size will remain reasonable. 

Defining multiple module entrypoints
------------------------------------

Typically a module's entrypoint (API) is it's main class, *init.pp*. There are, however, cases where a module serves several disparate use cases. A few typical examples below:

  1. *Client-server software* such as OpenVPN requires significant configuration at both ends. While these two use cases are similar, there are enough configuration differences to warrant separate entrypoint classes. For example, a client would not need all the parameters a server needs, and using parameterization would result in having to define useless parameters for both use cases. This problem can be easily solved by using two entrypoint classes, *openvpn::client* and *openvpn::server*, and making them reuse the parts of the module they need.
  1. *Automating backups* using cron requires adding parameters for the month, week, monthday, minute, hour, database name, dump tool options, etc. Having all of these parameters in the main class (init.pp) would make it's API much more messy. On top of this backups are often not even managed by Puppet. This means that having a separate *modulename::backup* class is a much cleaner approach.

Defining multiple module entrypoints also keep the Puppet code much cleaner from confusing conditional code.

Forking the module
------------------

Forking a module is a good option if the use-case for a module differs a lot from the originally envisioned one. Forking does not, however, mean that the modules go their own ways forever. Typically a module contains several subclasses which rarely change, and some which change often. In most cases *install*, *service*, *monit* and *packetfilter* subclasses are fairly static, whereas the *config* subclass is most prone to change. Forking a module typically means rewriting only the main class as well as the "config" subclass. Because every class is in it's own file, it is still possible to merge fixes and improvements to the common parts (e.g. *install.pp*, *service.pp* or *monit.pp*).

Module structure
================

Each class in a module should be placed into a separate file for two reasons:

  * Reusing parts of the code becomes much easier
  * The structure of the module can be trivially viewed with commands such as *tree* or *ls -lR*

Here is a typical example of a module that install and configures a daemon: 

    snmpd
    ├── LICENSE
    ├── manifests
    │   ├── config
    │   │   └── freebsd.pp
    │   ├── config.pp
    │   ├── init.pp
    │   ├── install.pp
    │   ├── monit.pp
    │   ├── packetfilter.pp
    │   ├── params.pp
    │   └── service.pp
    ├── README.md
    └── templates
        ├── snmpd.conf.erb
        └── snmpd.monit.erb

The main class (init.pp) is used as the entrypoint or an interface to the module. As such it should not do any configuration on it own: the actual hard lifting should be delegated to the subclasses.

Default values
==============

Sane default values should be provided for entrypoint class parameters. This simplifies the use of the module:

    class snmpd
    (
        $iface = 'eth0',
        $community='public',
        $allow_address_ipv4='127.0.0.1',
        $allow_netmask_ipv4='32',
        $allow_address_ipv6='::1',
        $allow_netmask_ipv6='128',
        $min_diskspace='300000',
        $max_load='12 10 5',
        $monitor_email = $::servermonitor
    )

The parameter values are passed on to the subclasses as necessary:

    class { 'snmpd::packetfilter':
        iface => $iface,
        allow_address_ipv4 => $allow_address_ipv4,
        allow_netmask_ipv4 => $allow_netmask_ipv4,
        allow_address_ipv6 => $allow_address_ipv6,
        allow_netmask_ipv6 => $allow_netmask_ipv6,
    }

The subclasses are responsible for the actual configurations.

While this approach results in having to pass around variables, it has the benefit that it is possible to use only a part of module's functionality without forking the entire module.
