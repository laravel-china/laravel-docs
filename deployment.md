# Laravel 的部署

- [简介](#introduction)
- [服务器配置](#server-configuration)
    - [Nginx](#nginx)
- [优化](#optimization)
    - [优化自动加载](#autoloader-optimization)
    - [优化配置加载](#optimizing-configuration-loading)
    - [优化路由加载](#optimizing-route-loading)
- [Forge 部署](#deploying-with-forge)

<a name="introduction"></a>
## 简介

当你准备好将 Laravel 应用部署到生产环境时，你可以执行一些操作来确保应用程序尽可能高效地运行。在本文档中介绍一些能确保 Laravel 应用被正确部署。

<a name="server-configuration"></a>
## 服务器配置

<a name="nginx"></a>
### Nginx

如果你将应用程序部署到运行 Nginx 的服务器，可以使用下面的内容来配置 Web 服务器。这个文件可能需要根据你的服务器配置进行自定义。你可以考虑使用 [Laravel Forge](https://forge.laravel.com) 等服务协助管理你的服务器：

    server {
        listen 80;
        server_name example.com;
        root /example.com/public;

        add_header X-Frame-Options "SAMEORIGIN";
        add_header X-XSS-Protection "1; mode=block";
        add_header X-Content-Type-Options "nosniff";

        index index.html index.htm index.php;

        charset utf-8;

        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }

        location = /favicon.ico { access_log off; log_not_found off; }
        location = /robots.txt  { access_log off; log_not_found off; }

        error_page 404 /index.php;

        location ~ \.php$ {
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_pass unix:/var/run/php/php7.1-fpm.sock;
            fastcgi_index index.php;
            include fastcgi_params;
        }

        location ~ /\.(?!well-known).* {
            deny all;
        }
    }

<a name="optimization"></a>
## 优化

<a name="autoloader-optimization"></a>
### 优化自动加载

部署项目到生产环境时，请确保你优化了 Composer 类的自动加载映射，以便 Composer 可以快速找到正确文件为给定类加载：

    composer install --optimize-autoloader

> {tip} 除了优化自动加载之外，还应该确保项目的源代码管理库中包含了 `composer.lock` 文件。因为当 `composer.lock` 文件存在时，项目的依赖项可以被更快地安装。

<a name="optimizing-configuration-loading"></a>
### 优化配置加载

将应用部署到生产环境时，记得在部署过程中运行 Artisan 命令 `config:cache`：

    php artisan config:cache
这个命令可以将所有 Laravel 的配置文件合并到单个文件中缓存，此举能大大减少框架在加载配置值时必须执行的系统文件的数量。

<a name="optimizing-route-loading"></a>
### 优化路由加载

如果你构建的是具有许多路由的大型应用程序，那你应该在部署过程中运行 Artisan 命令 `route:cache`：

    php artisan route:cache
这个命令可以将所有路由注册减少为缓存文件中的单个方法调用，以达到当应用程序在注册数百条路由时，提高路由注册的性能。

> {note} 由于此功能使用 PHP 序列化，而 PHP 无法序列化闭包，因此只能缓存应用程序中基于控制器的路由。

<a name="deploying-with-forge"></a>
## Forge 部署

如果你还没有准备好管理自己的服务器配置，或者你的服务器没有配置 Laravel 应用程序所需的各种服务，[Laravel Forge](https://forge.laravel.com) 是一个不错的选择。

Laravel Forge 可以在各种基础设施提供商（如 DigitalOcean、Linode、AWS 等）上创建服务器。此外，Forge 还能安装和管理构建 Laravel 应用程序所需的所有工具，如 Nginx、MySQL、Redis、Memcached、Beanstalk 等。

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
| --- | --- | --- | --- |
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  | 翻译 | Stay Hungry. Stay Foolish. |

---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
