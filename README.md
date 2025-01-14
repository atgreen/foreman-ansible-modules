# Foreman Ansible Modules [![Build Status](https://travis-ci.org/theforeman/foreman-ansible-modules.svg?branch=master)](https://travis-ci.org/theforeman/foreman-ansible-modules)

This repository contains Ansible modules for interacting with a Foreman server API and various plugin APIs such as Katello.

## Goals

The intent of this repository is to be a place that community members can develop or contribute modules. The goals of this repository are:

  * centralized location for community modules
  * a single repository to clone for interacting with Foreman & plugins
  * an intermediate landing place for modules before pushing them to Ansible community
  * repository maintainers will be working to push the modules into Ansible proper [https://github.com/ansible/ansible/tree/devel/lib/ansible/modules/remote_management/foreman](https://github.com/ansible/ansible/tree/devel/lib/ansible/modules/remote_management/foreman)

## Branches

* `master` - current development branch, using both `nailgun` and `apypie` libraries
* `nailgun` - the state of the repository before the switch to the `apypie` library started, `nailgun` is the only dependency

## How To Use The Repository

The following is an example of how you could use this repository in your own environment. Let's assume you have a directory of playbooks and roles in a git repository for your infrastructure named `infra`:

```
infra/
├── playbooks
└── roles
```

First, clone this repository into `infra/`:

```
cd infra/
git clone https://github.com/theforeman/foreman-ansible-modules.git
```

Note the `ansible.cfg` file cloned with foreman-ansible-modules. The ansible.cfg
needs to be in your current directory when you run `ansible` or
`ansible-playbook`. You can copy it to another location or add it to your
current ansible configuration; make sure to update the relative paths to the
foreman-ansible-module `modules` and `module_utils` if you do so.

Now your playbooks and roles should have access to the `modules` and `module_utils`
contained in the repository for use, testing, or development of new modules.

## How to write modules in this repository

First of all, please have a look at the [Ansible module development](https://docs.ansible.com/ansible/latest/dev_guide/developing_modules_general.html) guide and get familiar with the general Ansible module layout.

When looking at actual modules in this repository ([`foreman_domain`](plugins/modules/foreman_domain.py) is a nice short example), you will notice a few differences to a "regular" Ansible module:

* Instead of `AnsibleModule`, we use `ForemanEntityApypieAnsibleModule` (and a few others, see [`plugins/module_utils/foreman_helper.py`](plugins/module_utils/foreman_helper.py)) which provides an abstraction layer for talking with the Foreman API
* We provide a `name_map` that translates between Ansible module parameters and Foreman API parameters, as nobody wants to write `organization_ids` in their playbook when they can write `organizations`

The rest of the module is usually very minimalistic:

* Connect to the API (`module.connect()`)
* Find the entity if it already exists (`entity = module.find_resource_by_name(…)`)
* Adjust the data of the entity if desired
* Ensure the entity state and details (`changed = module.ensure_resource_state(…)`)

Please note: we currently have modules that use `apypie` and `nailgun` as the backend libraries to talk to the API, but we would prefer not to add any new modules using `nailgun` and focus on migrating everything to `apypie`.

## How to test modules in this repository

To test, you need a running instance of Foreman, probably with Katello (use [forklift](https://github.com/theforeman/forklift) if unsure).
Also you need to run `make test-setup` and update `test/test_playbooks/vars/server.yml`:

```sh
make test-setup
vi test/test_playbooks/vars/server.yml # point to your Foreman instance
```

To run the tests using the `foreman_global_parameter` module as an example:

```sh
make test # all tests
make test_global_parameter  # single test
make test TEST="-k 'organzation or global_parameter'"  # select tests by expression (see `pytest -h`)
```

The tests are run against prerecorded server-responses.
You can (re-)record the cassettes for a specific test with

```sh
make record_global_parameter
```

See also [Guidedeline to writing tests](test/README.md).

## How to debug modules in this repository

Set up debugging using ansible's test-module

```sh
make debug-setup
```

Debug with ansible's test-module

```sh
make debug MODULE=<module name>

# Example: debug the katello_content_view module
$ make debug MODULE=katello_content_view
./.tmp/ansible/hacking/test-module -m modules/katello_content_view.py -a @test/data/content-view.json -D /usr/lib64/python2.7/pdb.py
...
```

You can set a number of environment variables besides `MODULE` to configure make. Check the [Makefile](https://github.com/theforeman/foreman-ansible-modules/blob/master/Makefile) for more configuration options.

## Modules List

This is a list of modules currently in the repository (please add to the list if adding a module).

#### Entity Modules

 * foreman_compute_attribute: create and maintain compute attributes
 * foreman_compute_resource: create and maintain compute resources
 * foreman_domain: create and maintain domains
 * foreman_environment: create and maintain environments (puppet)
 * foreman_global_parameter: create and maintain global parameters
 * foreman_hostgroup: create and maintain hostgroups
 * foreman_job_template: create and maintain job templates and associated template inputs
 * foreman_location: create and maintain locations
 * foreman_operating_system: create and maintain operating systems
 * foreman_organization: create and maintain organizations
 * foreman_os_default_template: create and maintain the association of default templates to operating systems
 * foreman_provisioning_template: create and maintain provisioning templates
 * foreman_ptable: create and maintain partition templates
 * foreman_role: create and maintain user roles
 * foreman_setting: set and reset settings
 * foreman_subnet: create and maintain subnets
 * katello_activation_key: create and maintain activation keys
 * katello_content_credential: create and maintain content credentials
 * katello_content_view: create and maintain content views
 * katello_product: create and maintain products
 * katello_repository: create and maintain repositories
 * katello_sync_plan: create and maintain sync plans
 * redhat_manifest: create and maintain manifests

#### Action Modules

 * katello_sync: sync Katello repositories and products
 * katello_upload: upload files, rpms, etc. to repositories. Note, rpms & files are idempotent.
 * katello_content_view_publish: publish Katello content views
 * katello_manifest: upload and Manage Katello manifests

## Nailgun Versions

Below is listed the correct Nailgun branch for your Server

Server | Nailgun branch
------------ | -------------
Katello | master
Satellite 6.3 | 6.3.z
Satellite 6.2 | 6.2.z

## Ansible Version

Please note that you need Ansible >= 2.3 to use these modules.
As we're using Ansible's [documentation fragment](https://docs.ansible.com/ansible/devel/dev_guide/developing_modules_documenting.html#documentation-fragments) feature, that was introduced in Ansible 2.8, `ansible-doc` prior to 2.8 won't be able to display the module documentation, but the modules will still run fine with `ansible` and `ansible-playbook`.

## Python Version

Starting with Ansible 2.7, Ansible only supports Python 2.7 and 3.5 (and higher). These are also the only Python versions we develop and test the modules against.
