# Laravel 的文件系统和云存储功能集成

- [简介](#introduction)
- [配置](#configuration)
    - [公共磁盘](#the-public-disk)
    - [本地驱动](#the-local-driver)
    - [驱动之前](#driver-prerequisites)
- [获取磁盘实例](#obtaining-disk-instances)
- [检索文件](#retrieving-files)
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

Laravel 提供了一个强大的文件系统抽象，这得益于 Frank de Jonge 强大的 [Flysystem](https://github.com/thephpleague/flysystem) 扩展包。Laravel 文件系统集成为使用本地文件系统、Amazon S3 和 Rackspace 云存储提供了简单易用的驱动程序。更棒的是，由于每个系统的 API 保持不变，所以在这些存储选项之间切换是非常简单的。

<a name="configuration"></a>
## 配置

文件系统的配置文件位于 `config/filesystems.php`。在这个文件中你可以配置所有「磁盘」。每个磁盘代表特定的存储驱动及存储位置。每种支持的驱动程序的示例配置都包含在配置文件中。因此，只需要修改配置即可反映你的存储偏好和凭据。

当然，你可以根据需要配置多个磁盘，甚至你还可以使多个磁盘共用同一个驱动。

<a name="the-public-disk"></a>
### 公共磁盘

`public` 磁盘适用于要公开访问的文件。默认情况下， `public` 磁盘使用 `local` 驱动，并且将这些文件存储在 `storage/app/public` 目录下。为了使它们能通过网络访问，你需要创建 `public/storage` 到 `storage/app/public` 的符号链接。这种方式能把可公开访问文件都保留在同一个目录下，以便在使用零停机时间部署系统如[Envoyer](https://envoyer.io/) 的时候，就可以轻松地在不同的部署之间共享这些文件。

你可以使用 Artisan 命令 `storage:link` 来创建符号链接：

    php artisan storage:link

当然，一旦一个文件被存储并且已经创建了符号链接，你就可以使用辅助函数 `asset` 来创建文件的 URL：

    echo asset('storage/file.txt');

<a name="the-local-driver"></a>
### 本地驱动

使用 `local` 驱动时，所有文件操作都你在配置文件中定义的 `root` 目录相关。该目录的默认值是 `storage/app`。因此，以下方法会把文件存储在 `storage/app/file.txt` 中：

    Storage::disk('local')->put('file.txt', 'Contents');

<a name="driver-prerequisites"></a>
### 驱动之前

#### Composer 包

在使用 S3 或 Rackspace 的驱动之前，你需要通过 Composer 安装相应的软件包：

- Amazon S3: `league/flysystem-aws-s3-v3 ~1.0`
- Rackspace: `league/flysystem-rackspace ~1.0`

#### S3 驱动配置

S3 驱动的配置信息位于 `config/filesystems.php` 配置文件中。该文件包含 S3 驱动程序的配置数组示例。你可根据自己的 S3 配置和凭证自由修改该数组。

#### FTP 驱动配置

Laravel 的文件系统集成能很好的支持 FTP，不过 FTP 的配置示例并没有被包含在框架默认的 `filesystems.php` 文件中。需要的话可以使用下面的示例配置：

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

Laravel 的文件系统集成也能很好的支持 Rackspace，不过 Rackspace 的配置示例也没被包含在框架默认的 `filesystems.php` 文件中。需要的话可以使用下面的示例配置：

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
## 获取磁盘实例

`Storage` facade 用于和所有已配置的磁盘进行交互。例如，你可以使用 facade 的 `put` 方法将头像保存到默认磁盘上。调 `Storage` facade 的方法前若未先调用 `disk` 方法，此方法会被自动传递给默认磁盘。如果没有先调用 `disk` 方法就调用了 `Storage` facade 上的方法，那调用的方法会自动被传递到默认磁盘：

    use Illuminate\Support\Facades\Storage;

    Storage::put('avatars/1', $fileContents);

如果应用程序和多个磁盘交互，则可以使用 `Storage` facade 上的 `disk` 方法来处理特定磁盘上的文件：

    Storage::disk('s3')->put('avatars/1', $fileContents);

<a name="retrieving-files"></a>
## 检索文件

`get` 方法可以用于检索文件的内容，此方法返回该文件的原始字符串内容。 切记，所有文件路径的指定都应该相对于为磁盘配置的 `root` 目录：

    $contents = Storage::get('file.jpg');

`exists` 方法可以用来判断磁盘上是否存在指定的文件：

    $exists = Storage::disk('s3')->exists('file.jpg');

<a name="file-urls"></a>
### 文件 URLs

当使用 `local` 或者 `s3` 驱动时，你可以使用 `url` 方法来获取给定文件的 URL。如果你使用的是 `local` 驱动，一般只是在给定的路径前面加上 `/storage` 并返回一个相对的 URL 到那个文件。如果使用的是 `s3` 驱动，会返回完整的远程 URL：

    use Illuminate\Support\Facades\Storage;

    $url = Storage::url('file1.jpg');

> {note} 切记，如果使用的是 `local` 驱动，则所有想被公开访问的文件都应该放在 `storage/app/public` 目录下。此外，你应该在 `public/storage` [创建一个符号链接 ](#the-public-disk) 来指向 `storage/app/public` 目录。

#### 临时 URLs

对于使用 `s3` 驱动来存储的文件，可以使用 `temporaryUrl` 方法创建给定文件的临时 URL 。这个方法接收路径和 `DateTime` 实例来指定 URL 何时过期：

    $url = Storage::temporaryUrl(
        'file1.jpg', Carbon::now()->addMinutes(5)
    );

#### 自定义本地 URL 主机

如果要使用 `local` 驱动为存储在磁盘上的文件预定义主机，可以向磁盘配置数组添加一个 `url` 选项：

    'public' => [
        'driver' => 'local',
        'root' => storage_path('app/public'),
        'url' => env('APP_URL').'/storage',
        'visibility' => 'public',
    ],

<a name="file-metadata"></a>
### 文件元数据

除了读写文件，Laravel 还可以提供有关文件本身的信息。例如，`size` 方法可用来获取文件的大小（以字节为单位）：

    use Illuminate\Support\Facades\Storage;

    $size = Storage::size('file1.jpg');

`lastModified` 方法返回最后一次文件被修改的 UNIX 时间戳：

    $time = Storage::lastModified('file1.jpg');

<a name="storing-files"></a>
## 保存文件

`put` 方法可用于将原始文件内容保存到磁盘上。你也可以传递 PHP 的 `resource` 给 put 方法，它将使用文件系统下的底层流支持。强烈建议在处理大文件时使用流：

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents);

    Storage::put('file.jpg', $resource);

#### 自动流式传输

如果你想 Laravel 自动管理将给定文件流式传输到你想要的存储位置，你可以使用 `putFile` 或 `putFileAs` 方法。这个方法接受 `Illuminate\HTTP\File` 或 `Illuminate\HTTP\UploadedFile` 实例，并自动将文件流式传输到你想要的位置：

    use Illuminate\Http\File;

    // 自动为文件名生成唯一的 ID...
    Storage::putFile('photos', new File('/path/to/photo'));

    // 手动指定文件名...
    Storage::putFileAs('photos', new File('/path/to/photo'), 'photo.jpg');

关于 `putFile` 方法有些重要的事情要注意。请注意，我们只指定一个目录名，而不是文件名。默认情况下，`putFile` 方法将生成唯一的 ID 作为文件名。该文件的路径将被 `putFile` 方法返回，因此可以将路径（包括生成的文件名）存储在数据库中。

`putFile` 和 `putFileAs` 方法也接受一个参数来指定存储文件的「可见性」。如果你将文件存储在诸如 S3 的云盘上，并且该文件可以公开访问，这是特别有用的：

    Storage::putFile('photos', new File('/path/to/photo'), 'public');

#### 前置&追加

`prepend` 及 `append` 方法允许你写入内容到文件的开头或结尾：

    Storage::prepend('file.log', 'Prepended Text');

    Storage::append('file.log', 'Appended Text');

#### 复制 & 移动

`copy` 方法可用于将现有文件复制到磁盘的新位置，而 `move` 方法可用于重命名或将现有文件移动到新位置：

    Storage::copy('old/file1.jpg', 'new/file1.jpg');

    Storage::move('old/file1.jpg', 'new/file1.jpg');

<a name="file-uploads"></a>
### 文件上传

在 Web 应用中，存储文件最常见的例子之一就是存储用户上传的文件，如个人资料图片、照片和文档。 Laravel 可以使用文件上传实例的 `store` 方法来存储上传的文件。只需使用你想存储上传文件的路径调用 `store` 方法：

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

关于这个例有些注意事项。我们只指定目录名，而不是文件名。默认情况下，`store` 方法将生成唯一的 ID 来作为文件名。该文件的路径将被 `store` 方法返回，因此你可以在数据库中存储路径及生成的文件名。

你也可以调用 `Storage` facade 的 `putFile` 方法来执行和上面例子相同的文件操作：

    $path = Storage::putFile('avatars', $request->file('avatar'));

#### 指定文件名

如果你不喜欢自动生成的文件名，你可以使用 `storeAs` 方法，它接收的路径、文件名、磁盘（可选的）作为它的参数：

    $path = $request->file('avatar')->storeAs(
        'avatars', $request->user()->id
    );

当然，你也可以使用 `Storage` facade 的 `putFileAs` 方法，和上面例子的文件操作有相同效果：

    $path = Storage::putFileAs(
        'avatars', $request->file('avatar'), $request->user()->id
    );

#### 指定磁盘

默认情况下，此方法将使用默认的磁盘。如果你想指定其他磁盘，就将磁盘名称作为第二个参数传递给 `store` 方法：

    $path = $request->file('avatar')->store(
        'avatars/'.$request->user()->id, 's3'
    );

<a name="file-visibility"></a>
### 文件可见性

在 Laravel 的文件系统集成里，「可见性」 是跨多个平台的文件权限的抽象。文件可能被声明为 `public` 或 `private` 。当一个文件声明为 `public` 时，就意味着文件通常可以被其他人访问。例如，使用 S3 驱动时，你可检索 `public` 文件的 URL。

你可通过 `put` 方法在设置文件时指定它的可见性：

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents, 'public');

如果文件已经被保存，可以通过 `getVisibility` 和 `setVisibility` 方法检索和设置其可见性。

    $visibility = Storage::getVisibility('file.jpg');

    Storage::setVisibility('file.jpg', 'public')

<a name="deleting-files"></a>
## 删除文件

`delete` 方法接受文件名或文件名数组参数来删除磁盘中相应的文件：

    use Illuminate\Support\Facades\Storage;

    Storage::delete('file.jpg');

    Storage::delete(['file1.jpg', 'file2.jpg']);

<a name="directories"></a>
## 目录

#### 获取目录中的所有文件

`files` 方法返回给定目录下的所有文件的数组。如果你想检索包含所有子目录在内的给定目录中的所有文件的列表，可以使用 `allFiles` 方法：

    use Illuminate\Support\Facades\Storage;

    $files = Storage::files($directory);

    $files = Storage::allFiles($directory);

#### 获取目录内所有目录

`directories` 方法返回给定目录下的所有目录的数组。另外，你可以使用 `allDirectories` 方法获取给定目录及其所有子目录中的所有目录的列表：

    $directories = Storage::directories($directory);

    // 递归...
    $directories = Storage::allDirectories($directory);

#### 创建目录

`makeDirectory` 方法将创建给定的目录，包括任何所需的子目录：

    Storage::makeDirectory($directory);

#### 删除目录

最后，`deleteDirectory` 方法可用于删除目录及其所有文件：

    Storage::deleteDirectory($directory);

<a name="custom-filesystems"></a>
## 自定义文件系统

Laravel 的文件系统集成提供一系列开箱即用的驱动支持；然而文件系统不仅限于此，它还拥有其它存储系统的适配器。如果你想在 Laravel 的应用中使用额外的存储适配器，可以创建自定义驱动。

为了设置自定义文件系统，你需要一个文件系统适配器。让我们添加一个社区维护的 Dropbox 适配器到我们的项目中：

    composer require spatie/flysystem-dropbox

接下来，你需要创建一个[服务提供器](/docs/{{version}}/providers)，如 DropboxServiceProvider。并在该提供器的 `boot` 方法使用 `Storage` facade 的 `extend` 方法自定义你的驱动程序。

    <?php

    namespace App\Providers;

    use Storage;
    use League\Flysystem\Filesystem;
    use Spatie\Dropbox\Client as DropboxClient;
    use Illuminate\Support\ServiceProvider;
    use Spatie\FlysystemDropbox\DropboxAdapter;

    class DropboxServiceProvider extends ServiceProvider
    {
        /**
         * 执行注册后引导服务。
         *
         * @return void
         */
        public function boot()
        {
            Storage::extend('dropbox', function ($app, $config) {
                $client = new DropboxClient(
                    $config['authorizationToken']
                );

                return new Filesystem(new DropboxAdapter($client));
            });
        }

        /**
         * 在容器中注册绑定。
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

`extend` 方法的第一个参数是驱动程序的名称，第二个参数是接收 `$app` 及 `$config` 变量的闭包。该解析闭包必须返回 `League\Flysystem\Filesystem` 的实例。`$config` 变量包含了特定磁盘在 `config/filesystems.php` 中定义的值。

一旦通过创建服务提供器注册扩展名后，你就可以在 `config/filesystem.php` 配置文件中使用 `dropbox` 驱动程序。

## 译者署名

| 用户名 | 头像 | 职能 | 签名 |
| --- | --- | --- | --- |
| [@Godruoyi](https://segmentfault.com/blog/godruoyi) | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5359_1470894290.jpeg?imageView2/1/w/200/h/200"> | 翻译   | 听大悲咒敲代码的二楞徐                         |
| [@Rambone](https://github.com/zuoRambo)  | <img class="avatar-66 rm-style" src="http://tva1.sinaimg.cn/crop.0.0.1002.1002.180/92d03bcdjw8f0asasf3m1j20ru0rvaeo.jpg"> | 翻译   | php,go求职 简历请发zuoxiaojie@lianjia.com |


---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
