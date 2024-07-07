# Setup

## SilverStripe Installer

If using php 8.1+
```sh
composer create-project silverstripe/installer docker-ss
```

If using lower php version, in my case 7.3
```sh
composer create-project silverstripe/installer:5.2 docker-ss --ignore-platform-reqs
```

## Composer Platform Config

```json
"config": {
    "platform": {
      "php": "8.3.9",
      "ext-intl": "8.3.9"
    }
}
```

## NGINX conf template

Notably for the `fastcgi_pass   php:9000;`

```
server {
  listen 80;
  server_name ${NGINX_HOST};
  root /var/www/html/public;

  index index.php;

  location / {
    try_files $uri $uri/ /index.php?$args;
  }

  error_page 404 /assets/error-404.html;
  error_page 500 /assets/error-500.html;

  access_log off;
  sendfile off;

  location ~ [^/]\.php(/|$) {
    fastcgi_split_path_info ^(.+?\.php)(/.*)$;
    if (!-f $document_root$fastcgi_script_name) {
      return 404;
    }

    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_param PATH_INFO       $fastcgi_path_info;
    fastcgi_param PATH_TRANSLATED $document_root$fastcgi_path_info;

    fastcgi_pass   php:9000;

    fastcgi_connect_timeout 1d;
    fastcgi_read_timeout 1d;
    fastcgi_send_timeout 1d;

    fastcgi_buffer_size 32k;
    fastcgi_busy_buffers_size 64k;
    fastcgi_buffers 4 32k;
  }

}
```

## Composer vendor-expose

The composer container has issues copying resources to the public directory. They must be exposed by the host using the `copy` method.

### Remove public/\_resources

Delete the `public/_resources` directory in bash
```sh
rm -rf public/_resources
```

If you are on Windows, and are getting a permission error, use Poweshell as an admin.
- Shortcut: (Windows Key + X), then press "A" key
- Navigate to the project directory
```
rmdir .\public\_resources\
```
Hit enter for "yes"

### Expose from the host

Expose resources from the host using the copy method.
```sh
composer vendor-expose copy
```
