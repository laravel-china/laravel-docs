# Laravel 的文件系统和云存储功能集成

- [简介](#introduction)
- [配置](#configuration)
    - [公开磁盘](#the-public-disk)
    - [本地驱动](#the-local-driver)
    - [驱动的前置条件](#driver-prerequisites)
- [获取磁盘实例](#obtaining-disk-instances)
- [提取文件](#retrieving-files)
    - [文件 URLs](#file-urls)
    - [文件元数据](#file-metadata)
- [保存文件](#storing-files)
    - [文件上传](#file-uploads)
    - [文件可见性](#file-visibility)
- [删除文件](#deleting-files)
- [目录](#directories)
- [自定义文件系统](#custom-filesystems)


<a name="introduction"></a>
## 简介

Laravel 提供强大文件抽象能力，这得益于 Frank de Jonge 的 [Flysystem](https://github.com/thephpleague/flysystem) 扩展包。Laravel 集成的 flysystem 提供了可支持本地文件系统、Amazon S3及 Rackspace 云存储的简单易用的驱动程序。更棒的是，由于每个系统的API保持不变，所以在这些存储项之间切换是非常轻松的。

<a name="configuration"></a>
## 配置

文件系统配置文件位于 `config/filesystems.php`。该文件能让你设置所有的「磁盘（disk）」。每个磁盘代表一个特定的存储驱动及存储位置。各种支持驱动的配置示例已包含其中，仅需要简单的根据你的偏好配置及凭证设置进行修改即可。

当然，你可随意配置多组磁盘，即使多个磁盘使用相同的驱动。

<a name="the-public-disk"></a>
### 公开磁盘

「公开磁盘」就是指你的文件将可被公开访问，默认下， `public` 磁盘使用 `local` 驱动且将文件存放在 `storage/app/public` 目录下。为了能通过网络访问，你需要创建 `public/storage` 到 `storage/app/public` 的符号链接。这个约定能让你的可公开访问文件保持在同一个目录下，这样在不同的部署系统间就可以轻松共享，如 [Envoyer](https://envoyer.io) 的“不停服”部署系统。

你可以使用 `storage:link` Artisan 命令创建符号链接：

    php artisan storage:link

当然了，当文件放好且符号链接创建完毕后，你就可以用 `asset` 辅助函数创建 URL 了：

    echo asset('storage/file.txt');

<a name="the-local-driver"></a>
### 本地驱动

当使用 `local` 驱动时，所有的操作都是相对于你在配置文件中定义的 `root` 目录进行的。该目录默认是 `storage/app`。所以，下面方法会把文件保存在 `storage/app/file.txt`：

    Storage::disk('local')->put('file.txt', 'Contents');

<a name="driver-prerequisites"></a>
### 驱动的预先需求

#### Composer 包
在使用 S3 或 Rackspace 驱动之前，你需要通过 Composer 安装适当扩展包：

- Amazon S3: `league/flysystem-aws-s3-v3 ~1.0`
- Rackspace: `league/flysystem-rackspace ~1.0`

#### S3 驱动配置

S3 驱动配置信息位于 `config/filesystems.php` 配置文件中。 此文件有个关于S3 驱动的配置数组例子。你可根据自己的 S3 配置和凭证修改该数组。

#### FTP 驱动配置

Laravel 集成的 Flysystem 能很好的支持 FTP，不过 FTP 的配置示例没被包含在框架默认的 `filesystems.php` 文件中，需要的话照着下面的例子配置：

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


Laravel 集成的 Flysystem 能很好的支持 Rackspace，不过 Rackspace 的配置示例没被包含在框架默认的 `filesystems.php` 文件中，需要的话照着下面的例子配置：

    'rackspace' => [
        'driver'    => 'rackspace',
        'username'  => 'your-username',
        'key'       => 'your-key',
        'container' => 'your-container',
        'endpoint'  => 'https://identity.api.rackspacecloud.com/v2.0/',
        'region'    => 'IAD',
        'url_type'  => 'publicURL',
    ],

<a name="obtaining-disk-instances"></a>
## 获得磁盘实例

`Storage` facade 用于和所有已设置的磁盘交互。例如，你可以调 facade 的 `put` 方法将一张头像保存到默认磁盘上。调 `Storage` facade 的方法前若未先调用 `disk` 方法，此方法会被自动传递给默认磁盘。


    use Illuminate\Support\Facades\Storage;

    Storage::put('avatars/1', $fileContents);

要是你的应用和多个磁盘交互，你可以通过 `Storage` facade 的 `disk` 方法访问特定的磁盘：

    Storage::disk('s3')->put('avatars/1', $fileContents);

<a name="retrieving-files"></a>
## 提取文件

`get` 方法被用作提取文件内容，此方法返回该文件的原始字符串内容。 切记，所有文件路径都是基于配置文件中 `root` 目录的相对路径。

    $contents = Storage::get('file.jpg');

`exists` 方法可以被用于判断一个文件是否存在于磁盘：

    $exists = Storage::disk('s3')->exists('file.jpg');

<a name="file-urls"></a>
### 文件 URLs

当使用 `local` 或者 `s3` 驱动的时候，你可以使用 `url` 方法来获取给定文件的 URL。如果你使用 `local` 驱动，一般会在传参的路径前面加上 `/storage` 且返回相对路径。如果是 `s3` 的话，返回的是完整的 S3 文件系统的 URL：


    use Illuminate\Support\Facades\Storage;

    $url = Storage::url('file1.jpg');

> {note} 切记，如果使用 `local` 驱动，所有想被公开访问的文件都应该放在 `storage/app/public` 目录下。此外，你应该在`public/storage` [创建符号链接 ] (#the-public-disk) 来指向 `storage/app/public` 文件夹。

#### 定制本地 URL 主机

如果你想给使用 `local` 驱动的存储文件预定义主机的话，你可以在磁盘配置数组中添加 `url` 键：

    'public' => [
        'driver' => 'local',
        'root' => storage_path('app/public'),
        'url' => env('APP_URL').'/storage',
        'visibility' => 'public',
    ],


<a name="file-metadata"></a>
### 文件元数据

除了读写文件，Laravel 还可以提供有关文件本身的信息。例如，`size` 方法可用来获取以字节为单位的文件大小：

    use Illuminate\Support\Facades\Storage;

    $size = Storage::size('file1.jpg');

`lastModified` 方法返回的最后一次文件被修改的 UNIX 时间戳：

    $time = Storage::lastModified('file1.jpg');

<a name="storing-files"></a>
## 保存文件

`put` 方法用于保存文件原始内容到一个磁盘上。你也可以传递 PHP 的 `resource` 给 `put` 方法，它将使用 Flysystem 下的 stream 支持。强烈建议使用 streams 处理大型文件：

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents);

    Storage::put('file.jpg', $resource);

#### 自动流

如果您想 Laravel 自动管理指定文件流传输到您想要的存储位置，你可以使用 `putFile` 或 `putFileAs` 方法。这个方法可以接受一个 `Illuminate\HTTP\File` 或 `Illuminate\HTTP\UploadedFile` 实例，并自动将文件传输到你想要的位置：

    use Illuminate\Http\File;

    // 自动生成唯一文件名...
    Storage::putFile('photos', new File('/path/to/photo'));

    // 手动指定一个文件名...
    Storage::putFileAs('photos', new File('/path/to/photo'), 'photo.jpg');

关于 `putFile` 方法有些重要的提醒。我们只指定一个目录名，而非文件名。默认情况下，该 `putFile` 方法将生成以为唯一ID作为文件名。文件的路径将被 `putFile` 方法返回，因此您可以在数据库中存储路径及文件名。

`putFile` 和 `putFileAs` 方法也接受一个参数指定被存储文件的「可见性」。在使用如 S3 的云存储时，若希望该文件可被公开访问，这将非常有用：

    Storage::putFile('photos', new File('/path/to/photo'), 'public');

#### 插入到文件

`prepend` 及 `append` 方法允许你将内容写入到一个文件的开头或结尾：

    Storage::prepend('file.log', 'Prepended Text');

    Storage::append('file.log', 'Appended Text');

#### 复制 & 移动文件

`copy` 方法用于复制一个已存在的文件到磁盘的新位置。`move` 方法用于重命名或是移动一个已存在的文件到新位置：

    Storage::copy('old/file1.jpg', 'new/file1.jpg');

    Storage::move('old/file1.jpg', 'new/file1.jpg');

<a name="file-uploads"></a>
### 文件上传

在 Web 应用中，存储文件最常见的例子之一就是存储用户上传的文件，如个人资料图片，照片和文档。 Laravel 通过使用文件上传实力的 `store` 方法，使其可以非常容易的存储上传的文件。只需使用你想存储的路径来调用 `store` 方法即可：


    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserAvatarController extends Controller
    {
        /**
         * 更新用户头像。
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            $path = $request->file('avatar')->store('avatars');

            return $path;
        }
    }

关于此例有些注意事项。我们只指定一个目录名，而不是文件名。默认情况下，`store` 方法将生成唯一ID来作为文件名。此文件路径将被 `store` 方法返回，因此你可以在数据库中存储路径及文件名。

你也可以调用`Storage` facade 的 `putFile` 方法来执行和上面例子相同的文件操作：


    $path = Storage::putFile('avatars', $request->file('avatar'));


#### 指定文件名

如果你不喜欢自动生成的文件名，你可以使用 `storeAs` 方法，它接收的路径、文件名、磁盘（可选的）作为它的参数：


    $path = $request->file('avatar')->storeAs(
        'avatars', $request->user()->id
    );
    
当然，你也可以使用 `Storage` facade 的 `putFileAs` 方法，可以和上面例子的文件操作有相同效果：

    $path = Storage::putFileAs(
        'avatars', $request->file('avatar'), $request->user()->id
    );

#### 指定磁盘

默认情况下，此方法将使用默认的磁盘。如果你想指定其他磁盘，给`store` 方法的第二个参数传磁盘名：

    $path = $request->file('avatar')->store(
        'avatars/'.$request->user()->id, 's3'
    );

<a name="file-visibility"></a>
### 文件可见性

在 Laravel 的 Flysystem 集成里，「可见性」 是跨多平台的文件权限抽象。文件可以被设定为 `public` 或 `private` 。当一个文件声明为 `public` 时，就意味着文件一般可供他人访问。例如，使用S3驱动时，你可检索 `public` 文件的URL。

你可通过 `put` 方法设定文件可见性：

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents, 'public');

如果文件已经被保存，其可见性可以通过 `getVisibility` 来获取和 `setVisibility` 方法来设置。

    $visibility = Storage::getVisibility('file.jpg');

    Storage::setVisibility('file.jpg', 'public')

<a name="deleting-files"></a>
## 删除文件

`delete` 方法接受一个文件名称或文件数组，用于从磁盘移除文件：

    use Illuminate\Support\Facades\Storage;

    Storage::delete('file.jpg');

    Storage::delete(['file1.jpg', 'file2.jpg']);

<a name="directories"></a>
## 目录

#### 获取某目录内的所有文件

`files` 方法返回指定目录下的所有文件数组。如果你还想获取指定目录下子目录的文件，可以使用 `allFiles` 方法。


    use Illuminate\Support\Facades\Storage;

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

最后，`deleteDirectory` 方法删除目录及所包含的全部文件。

    Storage::deleteDirectory($directory);

<a name="custom-filesystems"></a>
## 自定义文件系统

Laravel 的 Flysystem 集成提供一系列开箱即用的驱动支持；然而 Flysystem 不仅限于此，还拥有其它存储系统适配器。如果在你的 Laravel 的应用中想使用额外的存储适配器，你可以创建自定义驱动。

为了建构一个自定义的文件系统，你需要创建一个如 `DropboxServiceProvider` 的 [服务提供者](/docs/{{version}}/providers)。并在该提供者的 `boot` 方法使用 `Storage` facade 的 `extend` 方法自定义你的驱动。

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

`extend` 方法的第一个参数是驱动名，第二个参数则是一个接受 `$app` 及 `$config` 变量的闭包。该闭包必须返回 `League\Flysystem\Filesystem` 的实例。`$config` 变量包含了在 `config/filesystems.php` 定义的特定磁盘配置。

一旦通过创建服务提供者注册此扩展后，你就可以在 `config/filesystem.php` 配置文件中使用 `dropbox` 驱动。

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@Rambone](https://github.com/zuoRambo)  | <img class="avatar-66 rm-style" src="http://tva1.sinaimg.cn/crop.0.0.1002.1002.180/92d03bcdjw8f0asasf3m1j20ru0rvaeo.jpg">  |  译者  | php,go求职 简历请发zuoxiaojie@lianjia.com | 