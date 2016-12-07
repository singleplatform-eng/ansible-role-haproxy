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
     
    haproxy_frontend_name: 'hafrontend'
    haproxy_frontend_bind_address: '*'
    haproxy_frontend_port: 80
    haproxy_frontend_mode: 'http'

HAProxy frontend configuration directives.

    haproxy_backend_name: 'habackend'
    haproxy_backend_mode: 'http'
    haproxy_backend_balance_method: 'roundrobin'
    haproxy_backend_httpchk: 'HEAD / HTTP/1.1\r\nHost:localhost'

HAProxy backend configuration directives.

    haproxy_backend_acls:
      - name: '{{ your_acl_name }}'
        var: 'path'
        module: 'reg'
        query: '.+?.+'

A list of acls for the backend context to use. Requires haproxy_version 1.6+

    haproxy_backend_conditions:
      - action: 'http-request set-query %[query]&thing=added'
        operator: 'if'
        acl: '{{ your_acl_name }}'

A list of conditionals used for backend. The acl must already be declared or haproxy will fail the restart. Requires haproxy_version 1.6+

    haproxy_backend_servers:
      - name: app1
        address: 192.168.0.1:80
      - name: app2
        address: 192.168.0.2:80

A list of backend servers (name and address) to which HAProxy will distribute requests.

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
        - { role: geerlingguy.haproxy }

## License

MIT / BSD

## Author Information

This role was created in 2015 by [Jeff Geerling](http://www.jeffgeerling.com/), author of [Ansible for DevOps](https://www.ansiblefordevops.com/). It has been extended to support parts of the 1.6+ haproxy release. 
