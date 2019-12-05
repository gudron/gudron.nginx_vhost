gudron.nginx_vhost
=========

Role-generator of nginx virtual host config files

Role Variables
--------------

### General variables

  * `vhost_destintion_path: string`
    Virtual host config directory path. In this directory will be rendered virtual host config file. Path like that - /any/path/to/vhost/config/files/directory/

  * `vhost_headers_params: dict`
    Dictionary of virtual host **default** http headers parameters. You can check default parameters in [defaults/main.yml](defaults/main.yml) file.

  * `host_params: dict`
    Dictionary of virtual host parameters. Like `charset` or `status` location path.

  * `vhost_fastcgi_params: dict`
    Dictionary of virtual hosts **default** fastcgi params. Used only for `fastcgi` virtual hosts type.

  * `vhost_proxy_params: dict`
    Dictionary of virtual hosts **default** proxy params. Used only for `proxy` virtual hosts type.
 Currently supported parameters: `address`, `port`.
  * `include_params: list of string`
    This is **default** list of nginx config files path to include in vhost `server` section.

  * `vhosts_params: dict`
    Dict of virtual hosts parameters. Like `type`, `domain`, `chartset` and etc.

#### Virtual hosts variables

  * `type: string`
    Type of virtual host. Supported `proxy`, `fastcgi`, `static`.

  * `domain: string | list of string`
    Domain name of virtual host. This is `server_name` nginx config file directive - https://nginx.org/en/docs/http/server_names.html

  * `chartset: string`
    Set charset for virtual host. This is `charset` nginx config file directive - https://nginx.org/en/docs/http/ngx_http_charset_module.html

    Default: 'utf-8'

  * `root: string`
    Set root directory path for virtual host. This is `root` nginx config file directive - http://nginx.org/en/docs/http/ngx_http_core_module.html#root

  * `status_path: string`
    Set path for virtual host status location. This is path in location section for virtual host status - http://nginx.org/ru/docs/http/ngx_http_status_module.html#status

    Default: ngxstatus

  * `cert_file_path: string`
    Set path for ssl public cert path. This is `ssl_certificate_key` config file directive - https://nginx.org/ru/docs/http/ngx_http_ssl_module.html#ssl_certificate

  * `private_key_file_path: string`
    Set path for ssl private key path. This is `ssl_certificate` config file directive - https://nginx.org/ru/docs/http/ngx_http_ssl_module.html#ssl_certificate_key

  * `hpkp: string`
    Set hpkp header for virtual host. This is `Public-Key-Pins` header - https://developer.mozilla.org/en-US/docs/Web/HTTP/Public_Key_Pinning

    Default: This parameter will auto calculated if you set `cert_file_path` in vhost parameter.

  * `headers: dict`
    Set http headers for virtual host. This is dict of `add_header` directives - https://nginx.org/ru/docs/http/ngx_http_headers_module.html#add_header

    Default: Passed data in virtual hosts parameters will merged with **General variables** - `vhost_headers_params`.

  * `proxy_params: dict`
    Set proxy params directives in main location. This is dict of `proxy_set_header` directives - https://nginx.org/ru/docs/http/ngx_http_proxy_module.html#proxy_set_header
 Currently supported parameters: `address`, `port`.
    Default: This parameter will auto calculated if you set `cert_file_path` in vhost parameter.

  * `backends: list of dict`
    Set servers for upstream module. This is list of `server` directives - https://nginx.org/ru/docs/http/ngx_http_upstream_module.html#server . Currently supported parameters: `address`, `port`.

  * `include_params: list of string`
    This is list of nginx config files path to include in vhost `server` section.

    Default: Passed data in virtual hosts parameters will merged with **General variables** - `include_params`.

  * `fastcgi_params: dict`
    Set fastcgi params directives in bootstrap location. This is dict of `fastcgi_param` directives - http://nginx.org/ru/docs/http/ngx_http_fastcgi_module.html#fastcgi_param

    Default: Passed data in virtual hosts parameters will merged with **General variables** - `vhost_fastcgi_params`.

  * `bootstrap: string`
    Path in bootstrap location, like `index.php`. This is variable of `index` - https://nginx.org/ru/docs/http/ngx_http_index_module.html#index and `try_files` - http://nginx.org/ru/docs/http/ngx_http_core_module.html#try_files directives.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: example_project:&example_project_stage
      any_errors_fatal: "{{ any_errors_fatal | default(true) }}"
      gather_facts: false
      vars:
        ansible_connection: local
        ansible_become: no
        ansible_distribution: Debian
            
      roles:
        - name: gudron.nginx_vhost
          vars: 
            vhost_destintion_path: /example/project/nginx/conf.d/sites-available/
            vhost_proxy_params:
              Host: $http_host
              X-Forwarded-Proto: $scheme
            vhosts_params:
              api:
                port: 80
                domain: api.example.com
                type: proxy
                root: /example/project/app/static/dir/
                status_path: ngxstatus
                headers:
                  X-Content-Type-Options: nosniff
                  Strict-Transport-Security: max-age=31536000; includeSubDomains
                  X-Frame-Options: SAMEORIGIN
                proxy_params:
                  X-Real-IP: $remote_addr
                include_params:
                  - /etc/nginx/conf.d/include_file.conf
                backends: 
                  - address: api.example.internal
                    port: 8080

        - name: gudron.nginx_vhost
          vars: 
            vhost_destintion_path: /example/project/nginx/conf.d/sites-available/
            vhosts_params:
              api:
                address: api.example.internal
                port: 8080
                domain: api.example.com
                type: fastcgi
                root: /example/project/app/public/
                status_path: ngxstatus
                backends: 
                  - address: api-worker-pool-1.example.internal
                    port: 1010
                  - address: api-worker-pool-2.example.internal
                    port: 1010
                bootstrap: index.php

License
-------

BSD
