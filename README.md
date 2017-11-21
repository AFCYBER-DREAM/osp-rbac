OpenStack Platform Role-Based Access Control
============================================

The purpose of this role is to manage RBAC setting within an OpenStack Platform installation.

Requirements
------------

- Ansible 2.4+
- Python Shade library 1.9+

Role Variables
--------------

/path/to/your/group_vars/${your_group_name}:

    ---
    # osp-rbac role variable definitions
    osp_rbac:
      # These api-related  variables are required for OpenStack authentication.
      api:                                            # Contains api-related variables
        auth:                                         # Mirrors parameters of OSP modules
          auth_url: "http://192.168.0.1:35357/v2.0"   # URL for Keystone adminURL
          username: "admin"                           # "admin" or other global admin user
          password: "{{ keystone_admin_password }}"   # Define on command-line via "-e" flag
          project_name: "admin"                       # name of user's primary project
        auth_type: "password"                         # Should be "password"
        endpoint_type: "admin"                        # Should be "admin"
        region_name: "RegionOne"                      # Set to adminURL's region (or omit)
        keystone_version: "2"                         # Set to either "2" or "3"; see adminURL
      # This dictionary should include the names of globally-defined roles.
      # Role definitions are in policy.json/yaml files for each OSP project.
      roles:
        - "admin"
        - "_member_"
      # This dictionary defines domains, projects, and groups (as needed).
      # Keystone v2 only supports projects, so the domains and groups are ignored.
      # Even if you use v2, you must define a single, arbitrarily-named domain.
      # If you use v2, you can forego the "groups" dictionary altogether.
      domains:                                        # Dictionary containing list of domains
      - name: default                                 # Name of one of the domains
        projects:                                     # Dicitionary containing list of domain projects
        - name: test1                                 # Name of one of the projects list's projects
          description: "test1 project"                # Description of the project
          admin:                                      # Dictionary containing admin user details
            name: test1admin                          # Name of the project's admin user
            password: secret                          # Initial password for user (change immediately)
            update_password: on_create                # Password update policy; see module docs
            email: test1admin@domain.net              # Admin user's email address
        - name: test2                                 # Name of another project, followed by it's details
          description: "test2 project"
          admin:
            name: test2admin
            password: secret
            update_password: on_create
            email: test2admin@domain.net
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
    - name: Validating RBAC settings of an OSP installation
      hosts: your_host_group_here
      become: true
      become_user: root
      become_method: sudo

      roles:
        - osp-rbac
    ...

Notes
-----
When you are creating group_vars, keep in mind that the structure is nearly identical for both Keystone v2 and v3 even though v2 will only use a subset of the variables defined in the group_vars. An example of this is the "domains" dictionary, which is required even if you are not using Keystone v3. This ensures a consist group_var structure regardless of the the API version, and allows you to seemlessly transition between v2 and v3 with very little refactoring of the group_vars. 

You may have already noticed that there are no host_vars required for this role; however, there is a requirement for the `keystone_admin_password` to be set on the command line like so:

```
$ ansible-playbook -i your_inventory_filename_here -e "keystone_admin_password=your_admin_password_here" osp-rbac.yml
```

You can add an additional layer of security like so:

```
$ read -sp "Enter keystone admin password:" keystone_admin_password_variable
Enter keystone admin password: <Enter_your_password_when_prompted>
$ ansible-playbook -i your_inventory_filename_here -e "keystone_admin_password=${keystone_admin_password_variable}" osp-rbac.yml
```

License
-------

MIT

Author Information
------------------

The Development Range Engineering, Architecture, and Modernization (DREAM) Team.
