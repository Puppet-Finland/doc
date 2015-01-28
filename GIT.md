Working with Git
================

Git configuration
-----------------

Git options *user.name* and *user.email* should be configured, so that a Signed-Off-By line can be added with *-s* switch to git-commit.

The commit message should be of the following format:

    Summary of the change in 80 chars or less

    Optional detailed, multi-line description, each line 80 chars or less  

    Signed-off-by: User Name <user@domain.com>

A real-life example:

    Added the possibility to allow traffic from an additional IPv4 address/subnet

    Previously only the IPv4 addresses reported by the Filedaemons were allowed to
    contact the Storagedaemon. Sometimes this does not work, but the new
    $allow_additional_ipv4_address parameter can defined to work around this issue.

    Signed-off-by: Samuli Seppänen <samuli@openvpn.net>

The purpose of the Signed-Off-By lines is to identify the author(s) of the changes.

Using Git submodules
--------------------

Each Puppet module should be a Git submodule for a number of reasons:

* Puppet modules are typically in separate Git repositories anyways
* Git submodules can be managed efficiently with built-in Git tools, unlike separate Git repositories
* Adding, removing and replacing individual modules is easy
* Eech site can include only those modules which it really needs

Using external Puppet modules
-----------------------------

If you're using external Puppet modules it's advisable to fork them first and 
then use the fork. This ensures that external changes do not accidentally break 
your nodes.

Symbolic links
--------------

Oftentimes the Git repository name is different fron the Puppet module name. For 
example "puppetlabs-git" contains a Puppet module called "git". To prevent 
namespace clashes it's useful to clone the Git repository as is, then make a 
symbolic link to the directory so that Puppet module autoloader can find it:

    $ git submodule add https://github.com/Puppet-Finland/puppetlabs-git.git
    $ ln -s puppetlabs-git git

If you want to switch to another Git module later on you can simply replace the 
symbolic link.

Rebasing modules
----------------

The goal of the Puppet-Finland project is to produce a consistent and reusable 
set of Puppet modules which can be used for most purposes without any 
modifications. What this means is that all sites/environments will be using the 
same set of modules and that modules on all sites will be kept updated with 
upstream. In this case upstream means the status of modules in Puppet-Finland 
GitHub project.

Because the same modules are used on all sites the modules need to be rebased 
from Puppet-Finland on a regular basis. If the Puppet modules are Git submodules 
as recommended above, reviewing the changes is fairly easy.

    $ git submodule foreach "git fetch"
    $ git submodule foreach "git diff origin/master"

If the changes look innocent you can rebase the modules with upstream:

    $ git submodule foreach "git pull --rebase origin"
