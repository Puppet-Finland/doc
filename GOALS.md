Project goals
=============

The goal of the Puppet-Finland project is to produce a consistent, simple and 
reusable set of Puppet modules which can be used for most purposes without any 
modifications.

To meet the requirement for "simple" only the most common usecases are covered. 
Allowing for all possible use-case would produce modules that

* Have dozens of parameters 
* Are near-impossible to read or understand
* Break easily, even if elaborate testing environments are in place

A simple, easy to read module does not need heavy testing framework to the 
extent as a very complex module.

We encourage you to take our modules and modify them to suit your needs. If the 
modifications are such that many people could benefit from them then they 
probably will be accepted upstream. If the new functionality is seldomly needed 
then it will not be merged, and that's just fine. You can still provide patches 
to the generally useful parts such as install.pp, service.pp, monit.pp and such.
