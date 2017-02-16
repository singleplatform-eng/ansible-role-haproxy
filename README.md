# Ansible Role: HAProxy

Installs HAProxy on RedHat/CentOS and Debian/Ubuntu Linux servers.

## Requirements

None.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

    haproxy_version: 1.6

This will default to the platform default if you don't set it. 1.6+ enables additional config options.

    haproxy_socket: /var/lib/haproxy/stats

The socket through which HAProxy can communicate (for admin purposes or statistics). To disable/remove this directive, set `haproxy_socket: ''` (an empty string).

    haproxy_chroot: /var/lib/haproxy

The jail directory where chroot() will be performed before dropping privileges. To disable/remove this directive, set `haproxy_chroot: ''` (an empty string). Only change this if you know what you're doing!

    haproxy_user: haproxy
    haproxy_group: haproxy

The user and group under which HAProxy should run. Only change this if you know what you're doing!


    haproxy_admin_bind_address
    haproxy_admin_port
    haproxy_stats_bind_address
    haproxy_stats_port
    haproxy_stats_uri

Enables stats/admin panels. Set the port numbers and uri to enable. Doesn't currently support auth options. Defaults to local host. 
     
    haproxy_frontends:
      - name: front
        bind_address: '*'
	port: '9999'
	mode: http
	acls:
	  - name: valid-ua
	    criterion: 'hdr(user-agent)'
	    patterns:
	      - '-f exact-ua.lst'
	      - '-i -f generic-ua/lst'
	      - test
	conditions:
	  - action: use_backend backend_a
	    operator: if
	    acls:
	      - valid-ua
	default: backend_b

    haproxy_frontends:
      - "{{ write_frontend }}"
      - "{{ read_frontend }}" 

HAProxy frontend configuration structure. Takes a list of dictionaries representing a frontend configuration. This lets you make multiple front ends, acls, and conditions using those acls. Flags for acl patterns should be included with the pattern. Condition logical operators should be included in the subsequent acls or as a standalone entry in the conditions['acls'] list. Frontends can also be declared elsewhere and included in the list. 

    haproxy_read_backend:
      name: read
      mode: http
      balance_method: leastconn
      httpchk: GET /health-check 
      servers:
        - name: read-a
          address: read-a.example.com:80
          options: cookie read-a check maxconn 1000
        - name: read-b
          address: read-b.example.com:80
          options: cookie read-b check maxconn 1000

HAProxy backend configuration structure. Takes a list of dictionaries representing a backend configuration. This allows for multiple backends, e.g. read write splits. The acls and conditions structure matches the frontend configuration. Servers are described in the server section. 

    acl:
      name: your_acl_name
      criterion: 'path'
      patterns:
        - '-m reg .+?.+'

An example acl dictionary. Modules and flags should be included in the pattern or listed as their own pattern list item. Flags will carry through to the next entry if no flags exist for the entry. Requires haproxy_version 1.6+

    condition:
      action: 'http-request set-query %[query]&thing=added'
      operator: 'if'
      acls:
        - this_acl
        - AND other_acl

A conditional dictionary. The acl must already be declared or haproxy will fail the restart. Requires haproxy_version 1.6+

    server:
      name: app1
      address: 192.168.0.1:80
      options: check maxconn 1000

A server dictionary.

    haproxy_global_vars:
      - 'ssl-default-bind-ciphers ABCD+KLMJ:...'
      - 'ssl-default-bind-options no-sslv3'

A list of extra global variables to add to the global configuration section inside `haproxy.cfg`.

## Dependencies

None.

## Example Playbook

    - hosts: balancer
      sudo: yes
      roles:
        - { role: haproxy }

## License

MIT / BSD

## Author Information

This role was created in 2015 by [Jeff Geerling](http://www.jeffgeerling.com/), author of [Ansible for DevOps](https://www.ansiblefordevops.com/). It has been extended to support parts of the 1.6+ haproxy release. 
