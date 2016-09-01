# 文件系统与云存储

- [简介](#introduction)
- [设置](#configuration)
- [基本用法](#basic-usage)
    - [获取磁盘实例](#obtaining-disk-instances)
    - [提取文件](#retrieving-files)
    - [保存文件](#storing-files)
    - [文件可见性](#file-visibility)
    - [删除文件](#deleting-files)
    - [目录](#directories)
- [自定义文件系统](#custom-filesystems)

<a name="introduction"></a>
## 简介

Laravel 强大的文件抽象层得力于 Frank de Jonge 的 [Flysystem](https://github.com/thephpleague/flysystem) 扩展包。Laravel 的 flysystem 集成提供了可给本地端磁盘系统、Amazon S3、以及 Rackspace 云存储使用的各种驱动（driver）。并且能像使用 API 一样，轻易的切换这些存储方式来应对各式系统。

<a name="configuration"></a>
## 设置

文件系统配置文件位于 `config/filesystems.php`。该文件能让你设置所有的「磁盘（disk）」。每个磁盘代表一个唯一的存储驱动以及存储位置。各种支持驱动的例子已包含其中，仅需要简单的根据你的偏好配置及凭证设置进行修改即可。

当然，你也可以设置多组磁盘，甚至在多个磁盘使用相同的驱动。

<a name="the-public-disk"></a>
#### 公开磁盘

公开磁盘意味着你的所有文件将可以公开访问，默认的 `public` 磁盘使用 `local` 驱动，并且存储文件至 `storage/app/public` 文件夹中。为了能公开访问，你需要创建 `public/storage` 文件夹，然后作为符号链接到 `storage/app/public` 文件夹。这个约定能使你在使用 [Envoyer](https://envoyer.io) 类似的部署器的时候保证文件夹配置的统一。

当符号链接创建完毕以后，你就可以使用 `asset` 辅助函数创建共用的 URL：

    echo asset('storage/file.txt');

#### 本地端驱动

当使用 `local` 驱动时，所有的操作都是相对于配置文件的 `root` 目录设置进行的。该目录默认是 `storage/app`。因此下列方法将把文件保存在 `storage/app/file.txt`：

    Storage::disk('local')->put('file.txt', 'Contents');

#### 其它驱动的预先需求

在使用 S3 或 Rackspace 驱动之前，你需要通过 Composer 安装适当扩展包：

- Amazon S3: `league/flysystem-aws-s3-v3 ~1.0`
- Rackspace: `league/flysystem-rackspace ~1.0`

#### FTP 驱动配置

Laravel 强大的文件系统能很好的支持 FTP，不过 FTP 的配置信息并没有被包含在 `filesystems.php` 文件中，你可以使用以下样本代码进行配置：

    'ftp' => [
        'driver'   => 'ftp',
        'host'     => 'ftp.example.com',
        'username' => 'your-username',
        'password' => 'your-password',

        // Optional FTP Settings...
        // 'port'     => 21,
        // 'root'     => '',
        // 'passive'  => true,
        // 'ssl'      => true,
        // 'timeout'  => 30,
    ],

#### Rackspace 驱动配置


Laravel 强大的文件系统能很好的支持 Rackspace，不过 Rackspace 的配置信息并没有被包含在 `filesystems.php` 文件中，你可以使用以下样本代码进行配置：

    'rackspace' => [
        'driver'    => 'rackspace',
        'username'  => 'your-username',
        'key'       => 'your-key',
        'container' => 'your-container',
        'endpoint'  => 'https://identity.api.rackspacecloud.com/v2.0/',
        'region'    => 'IAD',
        'url_type'  => 'publicURL',
    ],

<a name="basic-usage"></a>
## 基本用法

<a name="obtaining-disk-instances"></a>
### 获得磁盘实例

`Storage` facade 用于对任何已设置的磁盘进行交互。举例来说，你可以使用 facade 的 `put` 方法将一张头像保存到默认磁盘上。当使用 `Storage` facade 调用了方法却未先调用 `disk` 方法时，默认磁盘将被自动传递给该方法。

    <?php

    namespace App\Http\Controllers;

    use Storage;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 更新指定用户的头像。
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function updateAvatar(Request $request, $id)
        {
            $user = User::findOrFail($id);

            Storage::put(
                'avatars/'.$user->id,
                file_get_contents($request->file('avatar')->getRealPath())
            );
        }
    }

当使用多个磁盘时，你可以通过 `Storage` facade 的 `disk` 方法访问特定磁盘。当然，你也可以使用链式调用（chain methods）对磁盘使用各种执行方法。

    $disk = Storage::disk('s3');

    $contents = Storage::disk('local')->get('file.jpg')

<a name="retrieving-files"></a>
### 提取文件

`get` 方法提取指定文件的内容，该文件的原始字符串内容将通过该方法获取：

    $contents = Storage::get('file.jpg');

`has` 方法可以用于判定指定的文件是否存在于磁盘上：

    $exists = Storage::disk('s3')->has('file.jpg');

### 文件 URLs

当使用 `local` 或者 `s3` 驱动的时候，你可以使用 `url` 方法来获取文件的 URL。如果你使用 `local` 驱动，一般会在传参的路径前面加上 `/storage`。如果是 `s3` 的话，会返回完整的 S3 文件系统 URL:

    $url = Storage::url('file1.jpg');

> **注意：** 当使用 `local` 驱动的时候，请确定 [创建符号链接到 `public/storage`](#the-public-disk) 来指向 `storage/app/public` 文件夹。

#### 文件信息

`size` 方法获取文件的大小并以 bytes 显示：

    $size = Storage::size('file1.jpg');

`lastModified` 方法返回文件的最后修改时间并以 UNIX 时间戳显示：

    $time = Storage::lastModified('file1.jpg');

<a name="storing-files"></a>
### 保存文件

`put` 方法保存单个文件于磁盘上。你能同时传递 PHP 的 `resource` 给 `put` 方法，它将使用文件系统底层的 stream 支持。强烈建议使用 stream 处理大型文件。

    Storage::put('file.jpg', $contents);

    Storage::put('file.jpg', $resource);

`copy` 方法用于复制一个已存在的文件到磁盘的新位置。

    Storage::copy('old/file1.jpg', 'new/file1.jpg');

`move` 方法用于重命名或是移动一个已存在的文件到新位置。

    Storage::move('old/file1.jpg', 'new/file1.jpg');

#### 插入到文件

`prepend` 及 `append` 方法允许你轻易的将内容插入到一个文件的开头或结尾：

    Storage::prepend('file.log', 'Prepended Text');

    Storage::append('file.log', 'Appended Text');

<a name="file-visibility"></a>
### 文件可见性

文件的可见性可以通过 `getVisibility` 来获取和 `setVisibility` 方法来设置。文件可见性是多平台上的文件权限的抽象层：

    Storage::getVisibility('file.jpg');

    Storage::setVisibility('file.jpg', 'public');

你也可以选择在使用 `put` 创建文件时设置可见性，可见性的值为 `public` 或者是 `private`：

    Storage::put('file.jpg', $contents, 'public');

<a name="deleting-files"></a>
### 删除文件

`delete` 方法接受一个文件名称或文件名称数组，用以移除磁盘上的文件：

    Storage::delete('file.jpg');

    Storage::delete(['file1.jpg', 'file2.jpg']);

<a name="directories"></a>
### 目录

#### 获取单个目录内的所有文件

`files` 方法返回指定目录下的文件数组。如果你希望返回包含指定目录下所有子目录的文件，则可以使用 `allFiles` 方法。

    $files = Storage::files($directory);

    $files = Storage::allFiles($directory);

#### 获取单个目录内所有目录

`directories` 方法返回指定目录下的目录数组。另外，你也可以使用 `allDirectories` 方法获取指定目录下的子目录以及子目录所包含的目录。

    $directories = Storage::directories($directory);

    // 递归...
    $directories = Storage::allDirectories($directory);

#### 创建目录

`makeDirectory` 方法将创建指定的目录，包括任何所需的子目录。

    Storage::makeDirectory($directory);

#### 删除目录

最后，`deleteDirectory` 方法能移除磁盘上的单个目录以及所包含的全部文件。

    Storage::deleteDirectory($directory);

<a name="custom-filesystems"></a>
## 自定义文件系统

Laravel 集成的 Flysystem 提供许多默认驱动；然而 Flysystem 本身不仅仅提供了这些，还包括其它保存系统的接口（adapter）。你能在 Laravel 的应用当中通过创建新的驱动使用这些额外的接口。

为了建构一个自定义的磁盘系统，你将需要创建一个像是 `DropboxServiceProvider` 的 [服务提供者](/docs/{{version}}/providers)。并在该提供者的 `boot` 方法使用 `Storage` facade 的 `extend` 方法自定义你的驱动。

    <?php

    namespace App\Providers;

    use Storage;
    use League\Flysystem\Filesystem;
    use Dropbox\Client as DropboxClient;
    use Illuminate\Support\ServiceProvider;
    use League\Flysystem\Dropbox\DropboxAdapter;

    class DropboxServiceProvider extends ServiceProvider
    {
        /**
         * 运行服务注册后的启动进程。
         *
         * @return void
         */
        public function boot()
        {
            Storage::extend('dropbox', function($app, $config) {
                $client = new DropboxClient(
                    $config['accessToken'], $config['clientIdentifier']
                );

                return new Filesystem(new DropboxAdapter($client));
            });
        }

        /**
         * 在容器注册绑定。
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

`extend` 方法的第一个参数是你的驱动名称，第二个参数则是一个接受 `$app` 及 `$config` 变量的闭包。该闭包必须返回 `League\Flysystem\Filesystem` 实例。`$config` 变量包含了定义在 `config/filesystems.php` 对指定磁盘的设置。

当你通过创建服务提供者注册该扩展后，你便能在 `config/filesystem.php` 配置文件中使用 `dropbox` 驱动。
