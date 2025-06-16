# Deployment

Congratulations on reaching the deployment stage of your application! But we still have some important steps to cover
before we can celebrate a successful launch. This guide outlines the best practices and necessary configurations for a
successful production deployment.

## System requirements

See the [system requirements](/docs/framework/latest/installation#system-requirements) for a complete list of
requirements to run ZAPHYR in production.

## Environment configuration

#### Disable debug mode

The `APP_DEBUG` variable in the `.env` file controls whether detailed error messages are shown. While useful in
development, debug mode **must be disabled** in production to prevent sensitive information from being exposed.

```bash
APP_DEBUG=false
```

With debug mode off, users will see clean, generic error pages instead of full stack traces.

#### Set environment to production

Set the `APP_ENV` variable to `production` to activate production-specific optimizations within the framework:

```bash
APP_ENV=production
```

## Server configuration

To run ZAPHYR in production, you need a web server that supports PHP. The most common options are Apache, Nginx, and
FrankenPHP. Each server has its own configuration requirements, but the general principles remain the same.

### Apache configuration

When using [Apache](https://httpd.apache.org), you should point the document root to the `public/` directory of your
application. You’ll also need to enable `.htaccess` files or include the rules directly in your Apache virtual host
configuration to support URL rewriting.

A basic `.htaccess` configuration with Apache might look like this:

```apacheconf
<IfModule mod_rewrite.c>
    <IfModule mod_negotiation.c>
        Options -MultiViews -Indexes
    </IfModule>

    RewriteEngine On

    # Redirect to HTTPS
    RewriteCond %{HTTPS} off
    RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]

    # Redirect www to non-www
    RewriteCond %{HTTP_HOST} ^www\.(.*)$ [NC]
    RewriteRule ^(.*)$ https://%1/$1 [L,R=301]

    # Redirect trailing slashes if not a directory
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_URI} (.+)/$
    RewriteRule ^ %1 [L,R=301]

    # Send requests to index.php
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [L]
</IfModule>
```

### nginx configuration

If you're using [nginx](https://nginx.org), configure your server block to serve the application from the `public/`
directory. You’ll also need to forward requests to `index.php` for routing.

A basic `nginx.conf` configuration for ZAPHYR with nginx might look like this:

```nginx
server {
    listen 80 default_server;
    listen 443 ssl default_server;
    
    root /var/www/html/public;
    
    ssl_certificate /etc/ssl/certs/master.crt;
    ssl_certificate_key /etc/ssl/certs/master.key;
    
    index index.php;
    
    location / {
        absolute_redirect off;
        try_files $uri $uri/ /index.php?$query_string;
    }
    
    location ~ \.php {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/run/php-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param SCRIPT_NAME $fastcgi_script_name;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_hide_header X-Powered-By;
    }
    
    location ~* /\.(?!well-known\/) {
        deny all;
    }
}
```

### FrankenPHP configuration

[FrankenPHP](https://frankenphp.dev/) is a modern PHP application server written in Go. For ZAPHYR, configure FrankenPHP
to serve from the `public/` directory and ensure it is executing `index.php` as the front controller.

To serve your application with FrankenPHP, you may run the following command in your project root:

```bash
frankenphp php-server -r public/
```

### Caddy configuration

[Caddy](https://caddyserver.com/docs/) is a powerful web server with automatic HTTPS support. Use it to serve the
`public/` directory and route requests to `index.php`.

For a basic Caddy configuration, create a `/etc/caddy/Caddyfile` in your project with the following content:

```ruby
:80 {
    php_fastcgi unix//var/run/php/php-fpm.sock
    root * /var/www/html/public
}
```

If you want to serve your application over HTTPS, you can use the following configuration:

```ruby
:443 {
    tls internal
    php_fastcgi unix//var/run/php/php-fpm.sock
    root * /var/www/html/public
}
```

### Document root

Regardless of which server you use, always set the document root to the `public/` directory of your ZAPHYR application.
This ensures that only publicly accessible files are exposed and your application’s core logic remains secure.

### Directory permissions

Ensure that your web server user (e.g., `www-data`, `apache`, or `nginx`) has read and write permissions for the
`storage/` directory. This is critical for logging, caching, sessions, and other runtime features. Improper permissions
can lead to errors, failed writes, or inaccessible features.

You can set the correct permissions using:

```bash
chmod -R 775 storage
chown -R www-data:www-data storage
```

Adjust the user/group according to your server setup.

## Deploy application

There are several ways to deploy your ZAPHYR application to a production server. The best method depends on your
infrastructure, team size, and deployment frequency. Below are some common approaches:

### Manual file transfer

The most straightforward, but also the least recommended, way to deploy an application is by manually copying files to
the server using tools like FTP, SCP, or similar file transfer methods. While this approach may seem simple, it comes
with several significant drawbacks.

Manual deployments lack automation and consistency, making them error-prone and difficult to reproduce reliably. You
also have limited visibility and control over what changes are being made on the server during the deployment process,
which increases the risk of downtime, broken configurations, or missing dependencies.

### Deployment tools

If you control your own server, you should use a deployment tool to automate the deployment process. Here are some
popular options:

- [DeployBot](https://deploybot.com) – A user-friendly deployment service that integrates with Git and supports
  automatic deployments.
- [Deployer](https://deployer.org) – A PHP-based deployment tool with zero-downtime deployment support and flexible
  configuration.
- [PHing](https://www.phing.info) – A build system based on Apache Ant, tailored for PHP projects with powerful task
  automation features.
- [Ansistrano](https://ansistrano.com) – A deployment tool built on top of Ansible, ideal for advanced and customizable
  server automation.

## Performance optimizations

To improve performance, ZAPHYR provides several caching mechanisms. Run the following command to optimize your
application before deploying:

```bash
php bin/zaphyr cache:optimize
```

This will compile and cache:

- Console commands
- Configuration files
- Event listeners
- Service providers
- Route controllers
- Middleware

These components will be loaded faster and more efficiently in production.

#### Clear cache

If you make changes to your configuration, service providers, routes, or any other cached component, clear the cache to
reflect updates:

```bash
php bin/zaphyr cache:clear
```

To remove **all** cached files and contents from the entire `storage/cache/` directory, use the `--all` option:

```bash
php bin/zaphyr cache:clear --all
```
