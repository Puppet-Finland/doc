Coding style
============

Indenting
---------

Basic indentation is 4 spaces. No tabs, please.

Use of quotes
-------------

Double quotes are only used when they are required, i.e. when a string has variables in it. In all other cases single quotes are used:

    package { 'module-package':
        name => "${::module::params::package_name}",
        ensure => present,
    }

Paths in File resources
-----------------------

The resource title is not used to define the target path. Instead, the path is defined explicitly using the "name" parameter:

    file { 'module-file':
        name => "${::module::params::config_file_name}"
        ensure => present,
    }

Naming resources
----------------

To prevent naming conflicts each resource name consists of two parts:

  * *Module name:* name of the module in which resource resides 
  * *Resource name:* (base)name of the resource

When dealing with defined resources an identifier is added to the name. Here are a few examples:

    file { 'ntp-ntp.conf': ... }
    package { 'ntp-ntp': ... }
    service { 'ntp-ntp': ...
    module::define { "module-$id": ... }

Managing resource dependencies
------------------------------

Resources should depend on classes instead of depending directly on the resources in those classes. The only exceptions are dependencies between resources in the same class. This approach allows changing the internal structure of the classes without having to worry about other classes breaking because some resource was renamed or removed. A typical module that configures a daemon has the following dependency chain:

    daemon::install -> daemon::config -> daemon::service

The daemon::install subclass is applied first, as it does not depend on any other class. All *resources* in *daemon::config* class depend on *daemon::install* class. If there are dependencies between the resources in *daemon::config* class, then additional dependencies pointing to those resources are added. Finally the resources in *daemon::service* class are set to depend on *daemon::config*.
