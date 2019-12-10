# Ansible Role nginx

Library of Ansible plugins and roles for deploying various services.
See [ansible-roles](https://github.com/davidfischer-ch/ansible-roles) for additional documentation.

This repository hosts the role nginx and may depend of other roles and plugins of the library.

## Dependencies

See [meta](meta/main.yml).

## Variables

TODO VARIABLES

## Example

Example for delivering some static content (e.g. a React Front-End Application after being built).

### Playbook

```
---

- hosts:
    - web
  roles:
    - nginx
  vars:
    # nginx_length_hiding_filter_module_enabled: yes
    # nginx_length_hiding_filter_module_version: 1.1.1  # 15/08/2018

    nginx_version: release-1.17.6  # 28/11/2019 Released 19/11/2019
    nginx_daemon_mode: systemd
    nginx_build_flags:
      # - --with-http_addition_module      # https://nginx.org/en/docs/http/ngx_http_addition_module.html
      # - --with-http_auth_request_module  # https://nginx.org/en/docs/http/ngx_http_auth_request_module.html
      # - --with-http_dav_module           # https://nginx.org/en/docs/http/ngx_http_dav_module.html
      # - --with-http_geoip_module         # https://nginx.org/en/docs/http/ngx_http_geoip_module.html
      - --with-http_gzip_static_module     # https://nginx.org/en/docs/http/ngx_http_gzip_static_module.html
      # - --with-http_image_filter_module  # https://nginx.org/en/docs/http/ngx_http_image_filter_module.html
      - --with-http_realip_module          # https://nginx.org/en/docs/http/ngx_http_realip_module.html
      - --with-http_ssl_module             # https://nginx.org/en/docs/http/ngx_http_ssl_module.html
      # - --with-http_stub_status_module   # https://nginx.org/en/docs/http/ngx_http_stub_status_module.html
      # - --with-http_sub_module           # https://nginx.org/en/docs/http/ngx_http_sub_module.html
      - --with-http_v2_module              # https://nginx.org/en/docs/http/ngx_http_v2_module.html
      # - --with-http_xslt_module          # https://nginx.org/en/docs/http/ngx_http_xslt_module.html
      - --with-ipv6                        # Enable IPv6 connectivity
      - --with-pcre-jit                    # http://nginx.org/en/docs/ngx_core_module.html#pcre_jit

      # OpenSSL source directory
      # - --with-cc-opt="-I/usr/local/ssl/include"
      # - --with-ld-opt="-L/usr/local/ssl/lib"

    nginx_extra_packages: []
    nginx_server_header_engine: iss
    nginx_worker_processes: '{{ 4 * ansible_processor_count }}'

    nginx_sites:
      site:
        name: my-react-app-dev
        config_file: /path/to/my/templates/react-app.conf.j2
        ssl_auto_signed:  # Generate an auto-signed certificate (this is a development environment)
        debug: yes        # An extra variable for react-app.conf.j2 available as item.value.debug
        directory: /path/to/your/app  # Available as item.value.with_dhparam in react-app.conf.j2
        domains:
          - '{{ ansible_fqdn }}'
        max_body_size: 0
        redirect_ssl: yes
        with_dhparam: yes
        with_http2: yes
        with_ssl: yes
```

### Config Template

```
{% include 'templates/site.reject-unkown.conf.j2' %}

{% if item.value.with_ssl|bool and item.value.redirect_ssl|bool %}
server {
    listen {{ nginx_port|int }};
    server_name {{ item.value.domains|join(' ') }};
    rewrite ^ https://$http_host$request_uri? permanent;
}
{% endif %}

server {
    {% if item.value.with_ssl|bool %}
    listen {{ nginx_port_ssl|int }} {{ item.value.with_http2|bool|ternary('ssl http2', '') }};
    {% include 'templates/site.ssl.conf.j2' %}
    {% else %}
    listen {{ nginx_port|int }};
    {% endif %}

    {% include 'templates/site.real_ip.conf.j2' %}

    {% if not item.value.with_ssl|bool and item.value.redirect_ssl|bool %}
    if ($http_x_forwarded_proto != 'https') {
        rewrite ^ https://$host$request_uri? permanent;
    }
    {% endif %}

    server_name {{ item.value.domains|join(' ') }};

    gzip off;  # Security - http://breachattack.com/resources/BREACH%20-%20SSL,%20gone%20in%2030%20seconds.pdf
    gzip_proxied any;
    gzip_types application/javascript application/json text/css text/javascript text/xml;

    client_max_body_size {{ item.value.max_body_size }};

    access_log /var/log/nginx/{{ item.value.name }}/access.log;
    error_log /var/log/nginx/{{ item.value.name }}/error.log warn;

    # location /robots.txt {
    #     alias /some/path/robots.txt;
    # }

    # location /favicon/ {
    #     alias /some/path/favicons/;
    # }

    location / {
        access_log  on;
        autoindex   on;
        {% if not item.value.debug|bool %}
        expires     max;
        {% endif %}
        gzip        on;
        sendfile    off;  # Avoid Nginx serving outdated static files
        alias {{ item.value.directory }}/;
    }
}
```

## License

See [LICENSE.rst](LICENSE.rst).

## Authors

See [AUTHORS](AUTHORS).

2014-2019 - David Fischer
