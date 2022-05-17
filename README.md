# Introduction
Laravel Octane supercharges your application's performance by serving your application using high-powered application servers, including Open Swoole and RoadRunner. Octane boots your application once, keeps it in memory, and then feeds it requests at supersonic speeds.

# Swoole or Roadrunner?
When you use Laravel Octane you need to decide if you're going to use the Swoole or Roadrunner web server to serve your application. For somebody who has never been down this path the decision can be overwhelming - after all, there are pros and cons to both and you don't want to leave useful features on the table by choosing the wrong web server. Below is a comparison between the two based on my own research.

# Roadrunner benefits ?
Roadrunner, unlike Swoole, is not a PHP extension but simply a replacement for PHP-FPM. This means it does not need to be compiled into the language itself, making it a lot easier for most people to deploy. In most cases you can simply swap your usage of PHP-FPM directly with Roadrunner and immediately see large performance improvements.

# Swoole drawbacks
Since Swoole is a PHP extension, you need to specifically compile it into the language during your build step. Depending on how you build and deploy your application this may introduce complexity for you or require you to learn how to build your own PHP installation. 
Swoole also does not work with Xdebug, the most popular debugging tool in the PHP ecosystem. You can read more about this in the GitHub issue.
Finally, there are a lot of reports of profiling and monitoring tools such as Datadog and New Relic not working reliably with Swoole due to its use of coroutines and technically running in a CLI environment.

# Installation
Octane may be installed via the Composer package manager:
```
composer require laravel/octane
```
After installing Octane, you may execute the octane:install Artisan command, which will install Octane's configuration file into your application:
```
php artisan octane:install
```
> Laravel Octane requires minimum PHP 8.0+ 

# RoadRunner
RoadRunner is powered by the RoadRunner binary, which is built using Go. The first time you start a RoadRunner based Octane server, Octane will offer to download and install the RoadRunner binary for you.
# Serving Your Application Via Nginx

In production environments, you should serve your Octane application behind a traditional web server such as a Nginx or Apache. 
In the Nginx configuration example below, Nginx will serve the site's static assets and proxy requests to the Octane server that is running on port 8000:
>change the nginx configration file in "** cd /etc/nginx/sites-avalible/<your-nginx-conf-file> **"

```
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}
 
server {
    listen 80;
    listen [::]:80;
    server_name domain.com;
    server_tokens off;
    root /home/forge/domain.com/public;
 
    index index.php;
 
    charset utf-8;
 
    location /index.php {
        try_files /not_exists @octane;
    }
 
    location / {
        try_files $uri $uri/ @octane;
    }
 
    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }
 
    access_log off;
    error_log  /var/log/nginx/domain.com-error.log error;
 
    error_page 404 /index.php;
 
    location @octane {
        set $suffix "";
 
        if ($uri = /index.php) {
            set $suffix ?$query_string;
        }
 
        proxy_http_version 1.1;
        proxy_set_header Host $http_host;
        proxy_set_header Scheme $scheme;
        proxy_set_header SERVER_PORT $server_port;
        proxy_set_header REMOTE_ADDR $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
 
        proxy_pass http://127.0.0.1:8000$suffix;
    }
}
  
```
Before using this feature, you should ensure that Node is installed within your local development environment. 
In addition, you should install the Chokidar file-watching library within your project:library:
```
npm install --save-dev chokidar
```
OR only run Npm (which downlod the all dependies):
```
npm install
```
# Serving Your Application
The Octane server can be started via the octane:start Artisan command. By default, this command will utilize the server specified by the server configuration option of your application's octane configuration file:
```
php artisan octane:start
```
By default, Octane will start the server on port 8000, so you may access your application in a web browser via http://localhost:8000.

# Watching For File Changes
Since your application is loaded in memory once when the Octane server starts, any changes to your application's files will not be reflected when you refresh your browser. For example, route definitions added to your routes/web.php file will not be reflected until the server is restarted. For convenience, you may use the --watch flag to instruct Octane to automatically restart the server on any file changes within your application:
```
php artisan octane:start --watch
```
# Stopping The Server
You may stop the Octane server using the octane:stop Artisan command:
```
php artisan octane:stop
```
# Checking The Server Status
You may check the current status of the Octane server using the octane:status Artisan command:
```
php artisan octane:status  
```
# Specifying The Worker Count
By default, Octane will start an application request worker for each CPU core provided by your machine. These workers will then be used to serve incoming HTTP requests as they enter your application. You may manually specify how many workers you would like to start using the --workers option when invoking the octane:start command:
```
php artisan octane:start --workers=4
```
If you are using the Swoole application server, you may also specify how many "task workers" you wish to start:
```
php artisan octane:start --workers=4 --task-workers=6
```
# Specifying The Max Request Count
To help prevent stray memory leaks, Octane can gracefully restart a worker once it has handled a given number of requests. To instruct Octane to do this, you may use the --max-requests option:
```
php artisan octane:start --max-requests=250
```
# Reloading The Workers
You may gracefully restart the Octane server's application workers using the octane:reload command. Typically, this should be done after deployment so that your newly deployed code is loaded into memory and is used to serve to subsequent requests:
```
php artisan octane:reload
```
# ** Serving Your Application Via HTTPS **
By default, applications running via Octane generate links prefixed with http://. The OCTANE_HTTPS environment variable, used within your application's config/octane.php configuration file, can be set to true when serving your application via HTTPS. When this configuration value is set to true, Octane will instruct Laravel to prefix all generated links with https://:
```
'https' => env('OCTANE_HTTPS', false),  
``` 
  
  
  
[LINK]https://www.youtube.com/watch?v=2vKnGX-bIu0
  
[LINK]https://laravel.com/docs/8.x/octane
  
[LINK]https://chriswhite.is/coding/swoole-vs-roadrunner-for-laravel-octane/#:~:text=Roadrunner%20benefits,for%20most%20people%20to%20deploy. 
