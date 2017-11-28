OpenStack Platform Administrator
============================================

The purpose of this role is to define and manage any administrator-controlled 
OpenStack resources (i.e. domains, projects, groups, admin users, quotas, fips, et cetera)
within an OpenStack Platform installation utilizing either Keystone v2 or v3.

Requirements
------------

- Ansible 2.4+
- Python Shade library 1.9+
- python-openstackclient
- Common GNU utils (bash, wc, grep, touch)

Role Variables
--------------

/path/to/your/playbooks/host_vars/localhost:

    ---
    # openstack-platform-related variable definitions
    osp_admin:
      # This dictionary should include the names of globally-defined roles.
      # Role definitions are in policy.json/yaml files for each OSP project.
      roles:
        - "admin"
        - "_member_"
     # The admin-defined public network that all tenants use for external access
      global_public_network:
        cidr: 10.16.0.0/16
        dns_nameservers:
          - 10.16.0.1
      # Default quotas for each project unless they are overwritten on a per-project basis.
      default_quotas:
        # Quota management will be included in the next release.
      # This dictionary defines domains, projects, and groups (as needed).
      # Keystone v2 only supports projects, so the domains and groups are ignored.
      # Even if you are using Keystone v2, the role expects a uniform dictionary structure, regardless.
      # As a result, when using v2, you must define a single, arbitrary domain containing all projects.
      # If you use v2, you can forego the "groups" dictionary altogether though.
      domains:                                        # Dictionary containing list of domains
      - name: default                                 # Name of one of the domains
        description: "the default domain"             # Description of the domain
        projects:                                     # Dicitionary containing list of domain projects
        - name: admin
          description: "admin project"
          admin:
            name: admin
            password: secret
            update_password: on_create
            email: admin@domain.net
            #default_project: leave_commented         # The global admin has no "default_project"
          private_network:
            cidr: 192.168.0.0/24
          public_network:
            minimum_fips: 38
        - name: test1                                 # Name of one of the projects list's projects
          description: "test1 project"                # Description of the project
          admin:                                      # Dictionary containing admin user details
            name: test1admin                          # Name of the project's admin user
            password: secret                          # Initial password for user (change immediately)
            update_password: on_create                # Password update policy; see module docs
            email: test1admin@domain.net              # Admin user's email address
            default_project: test1                    # The user's primary/default project
          private_network:                            # Variables related to the default private/tenant net
            cidr: 192.168.0.0/24                      # CIDR for the project's private/tenant network
          public_network:                             # Variables related to the global-public-network
            minimum_fips: 35                          # Minimum floating IPs project has on global-public-network
        - name: test2
          description: "test2 project"
          admin:
            name: test2admin
            password: secret
            update_password: on_create
            email: test2admin@domain.net
            default_project: test2
          private_network:
            cidr: 192.168.0.0/24
          public_network:
            minimum_fips: 35
        groups:                                       # Dictionary containing list of domain groups
        - name: "group1"                              # Name of one of the groups
          description: "group1 group"                 # Description of the group
    ...

Role Dependencies
------------

None

Example Playbook
----------------

    ---
    - hosts: localhost
      connection: local
      become: false
      gather_facts: false
    
      roles:
        - osp-admin
    ...


Notes
-----
When you are creating host_vars, keep in mind that the structure is nearly identical for both Keystone v2 and v3 even though v2 will only use a subset of the variables defined in the host_vars. An example of this is the "domains" dictionary, which is required even if you are not using Keystone v3. This ensures a consist host_vars structure regardless of the the API version, and allows you to seemlessly transition between v2 and v3 with very little refactoring of the group_vars. 

You may have already noticed that there are no group_vars required for this role; however, there is a requirement for all of the authentication credentials to be sourced in the same shell that is used to execute the playbook. Prior to running the playbook, you should be able to run `env | grep "OS_"` and see all the environment variables you'll need defined. Then, from that same terminal prompt, you will execute the playbook. Trying to execute the `source openrc` (or similar) command inside of the playbook itself does not work, because each task inside of the playbook is a child process of the shell used to execute the playbook. So, once each task completes, any env vars that it set that were specific to that child process, are dumped. These do not flow back up to the parent process that launched it.

The whole reason these variables are even required, is because the `os_` series of Ansible modules requires authentication credentials in order to auth to the OpenStack installation it is managing. The modules are relatively uniform in their authentication options, and each of them offers three different ways to authenticate. The first is `cloud-client-config`. This python library reads some variables from a location on the filesystem (a cloud.yml) file. The file is setup to define multiple clouds in a single place and call them by a nickname. The modules use the `cloud` parameter to reference this nickname, and look for these files in specific directories (e.g. /etc/openstack/). This method appears to have some limitations related to how it defines and stores some of the Keystone/Identity v3 parameters though. 

A similar authentication method used by these modules is the `auth` series of parameters that can be defined. These, by and large, have a one-to-one equivalency to the bash env vars defined when you `source openrc` the OpenStack provide credentials that you can download from Horizon. There are; however, some exceptions to that, which make it impractical for use. 

Additionally, I have found it extremely advantageous to use the same exact authentication method that the `python-openstackclient` uses when it authenticates to Keystone to perform API calls. As a result, sourcing an OpenStack provided `openrc` file is the authentication method of choice for this role. You'll notice a test of the env vars is performed at the front end of the playbook, and you are afforded the opportunity to review the env vars before the playbook proceeds with its run. 

The review time period is displayed when it reaches the appropriate step in the playbook. You can skip the review process by hitting <CTRL>+<C> and then <C> (Continue) or you can abort by hitting <CTRL>+<C> and then <A> (Abort) when prompted.



Usage
------- 
Executing the playbook is relatively simple. Since it is executed on the localhost, there is no need for an inventory file. Ansible will throw a little warning, but its default behavior is to execute on localhost when you don't provide a user-defined group for it to run the playbook against. You'll just need to ensure you've sourced your OpenStack credentials file prior to running the playbook. Also make sure you have the `shade` library installed and accessible in your path, or else the Openstack `os_` series Ansible modules cannot execute. For FIP-related tasks, you'll also need the `python-openstackclient` installed via package manager or pip, since none of the OS modules allow for the creation of FIPs. Here is an example of the commands:


```
$ pip freeze   #ensure the "shade" module is listed in the output
$ source ./openrc   #or whatever the name of your credentials file is...
$ ansible-playbook osp-admin.yml
```


License
-------

MIT

Author Information
------------------

The Development Range Engineering, Architecture, and Modernization (DREAM) Team.
