# Forked version

We use a forked version mainly for `opsworks_ruby/templates/default/appserver.nginx.conf.erb` which has the following custom configurations:

1) Ability for the load balancer to perform health checks:

```
location /ping {
  access_log off;
  return 200;
}
```

2) Load balancer uses the SSL certificate to terminate the connection and then decrypt requests from clients before sending them to the instances (also known as SSL termination).

Which also means, anyone landing on the following domains, should be redirected to the https protocol. For example:

- http://www.domain-name.com to https://www.domain-name.com

To achieve this. I have:

```
server {
  listen <%= @out[:port] %>; # Typically listens on port 80
  ...
  location @<%= @name %> {
    ...
    # Any request that did not originally come in to the ELB over HTTPS gets redirected
    if ($http_x_forwarded_proto != "https") {
      rewrite ^(.*)$ https://$server_name$1 permanent;
    }
    ...
  }
  ...
}
```

3) To redirect the bare domain names to the https counterpart, i.e:

- http://domain-name.com to https://www.domain-name.com
- https://domain-name.com to https://www.domain-name.com

I have the following:

```
# If request is made to the bare domain (i.e. domain-name.com)
# Issue a redirect 301 response
server {
  listen 80;
  server_name domain-name.com;

  location /ping {
    access_log off;
    return 200;
  }

  location / {
    return 301 https://www.$server_name$request_uri;

    # Add HTTP Strict Transport Security for good measure
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains;";
  }
}
```

---

# opsworks_ruby Cookbook

[![Chef cookbook](https://img.shields.io/cookbook/v/opsworks_ruby.svg)](https://supermarket.chef.io/cookbooks/opsworks_ruby)
[![Build Status](https://travis-ci.org/ajgon/opsworks_ruby.svg?branch=master)](https://travis-ci.org/ajgon/opsworks_ruby)
[![Coverage Status](https://coveralls.io/repos/github/ajgon/opsworks_ruby/badge.svg?branch=master)](https://coveralls.io/github/ajgon/opsworks_ruby?branch=master)
[![Documentation Status](https://readthedocs.org/projects/opsworks-ruby/badge/?version=latest)](http://opsworks-ruby.readthedocs.io/en/latest/?badge=latest)
[![Commitizen friendly](https://img.shields.io/badge/commitizen-friendly-brightgreen.svg)](http://commitizen.github.io/cz-cli/)
[![license](https://img.shields.io/github/license/ajgon/opsworks_ruby.svg?maxAge=2592000)](https://opsworks-ruby.mit-license.org/)

A [chef](https://www.chef.io/) cookbook to deploy Ruby applications to Amazon OpsWorks.

## Quick Start

Refer to [Getting Started](http://opsworks-ruby.readthedocs.io/en/latest/getting_started.html)
guide in [documentation](http://opsworks-ruby.readthedocs.io/en/latest/index.html).

## Development

You can either install eveyrthing locally using [rvm](https://rvm.io/) and [pip](https://pypi.python.org/pypi/pip)
or use the Docker container which includes all necessary dependencies inside it.

### Build documentation

```
docker-compose run cookbook bash -c "cd docs && make html"
```

## Contributing

Please see [CONTRIBUTING](https://github.com/ajgon/opsworks_ruby/blob/master/CONTRIBUTING.md)
for details.

## Author and Contributors

Author: [Igor Rzegocki](https://www.rzegocki.pl/) ([@ajgon](https://github.com/ajgon))

### Contributors

* Nick Marden ([@nickmarden](https://github.com/nickmarden))
* Phong Si ([@phongsi](https://github.com/phongsi))
* Kevin Olbrich ([@olbrich](https://github.com/olbrich))
* Kevin Pheasey ([@kpheasey](https://github.com/kpheasey))
* Nathan Flood ([@npflood](https://github.com/npflood))
* Teruo Adachi ([@interu](https://github.com/interu))
* Marcos Beirigo ([@marcosbeirigo](https://github.com/marcosbeirigo))

## License

License: [MIT](http://opsworks-ruby.mit-license.org/)
