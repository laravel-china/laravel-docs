# 文件系统与云存储

- [简介](#introduction)
- [设置](#configuration)
- [基本用法](#basic-usage)
    - [获取磁盘实体](#obtaining-disk-instances)
    - [提取文件](#retrieving-files)
    - [保存文件](#storing-files)
    - [删除文件](#deleting-files)
    - [目录](#directories)
- [自定义文件系统](#custom-filesystems)

<a name="introduction"></a>
## 简介

Laravel 强大的文件抽象层得力于 Frank de Jonge 的 [Flysystem](https://github.com/thephpleague/flysystem) 扩展包。Laravel 的 flysystem 集成以各种驱动（driver）提供本地端磁盘系统、Amazon S3、以及 Rackspace 云存储。并且能像使用 API 一般，轻易的切换这些保存方式来面对各式系统。

<a name="configuration"></a>
## 设置

文件系统配置文件位于 `config/filesystems.php`。该文件能让你设置所有的「磁盘（disk）」。每个磁盘代表一种唯一的保存驱动以及保存位置。各种支持驱动例子已包含在其中，仅需要简单的根据你的偏好配置及凭证设置进行修改即可。

当然，你可以设置多组磁盘，甚至使用相同驱动。

#### 本地端驱动

当使用 `local` 驱动，所有的操作是相对于配置文件中的 `root` 目录设置进行。该目录默认是 `storage/app`。因此下列方法将把文件保存在 `storage/app/file.txt`：

    Storage::disk('local')->put('file.txt', 'Contents');

#### 其他驱动的预先需求

在使用 S3 或 Rackspace 驱动之前，你需要通过 Composer 安装适当扩展包：

- Amazon S3: `league/flysystem-aws-s3-v3 ~1.0`
- Rackspace: `league/flysystem-rackspace ~1.0`

<a name="basic-usage"></a>
## 基本用法

<a name="obtaining-disk-instances"></a>
### 获得磁盘实体

`Storage` facade 用于对任何已设置的磁盘进行交互。举例来说，你可以使用 facade 的 `put` 方法将一张头像保存到默认磁盘。当使用 `Storage` facade 调用任一方法而未先调用 `disk` 方法，默认磁盘将自动传递给该方法。

    <?php

    namespace App\Http\Controllers;

    use Storage;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Update the avatar for the given user.
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

面对使用多个磁盘时，你可以通过 `Storage` facade 的 `disk` 方法访问特定磁盘。当然，你也可以使用链式调用（chain methods）对磁盘使用各种运行方法。

    $disk = Storage::disk('s3');

    $contents = Storage::disk('local')->get('file.jpg')

<a name="retrieving-files"></a>
### 提取文件

`get` 方法提取给定文件的内容，该文件的原始字符串内容将通过该方法获取：

    $contents = Storage::get('file.jpg');

`has` 方法可以用于判定给定的文件是否存于磁盘上：

    $exists = Storage::disk('s3')->has('file.jpg');

#### 文件信息

`size` 方法获取文件的大小并以 bytes 显示：

    $size = Storage::size('file1.jpg');

`lastModified` 方法返回文件的最后修改时间并以 UNIX 时间戳显示：

    $time = Storage::lastModified('file1.jpg');

<a name="storing-files"></a>
### 保存文件

`put` 方法保存单一文件于磁盘上。你能同时传递 PHP 的 `resource` 给 `put` 方法，它将使用文件系统底层的 stream 支持。强烈建议使用 stream 处理大型文件。

    Storage::put('file.jpg', $contents);

    Storage::put('file.jpg', $resource);

`copy` 方法用于复制一个存在的文件到磁盘的新位置。

    Storage::copy('old/file1.jpg', 'new/file1.jpg');

`move` 方法被用于重命名或是移动一个存在的文件到新位置。

    Storage::move('old/file1.jpg', 'new/file1.jpg');

#### 插入到文件

`prepend` 及 `append` 方法允许你轻易的插入内容到一个文件的开头或结尾：

    Storage::prepend('file.log', 'Prepended Text');

    Storage::append('file.log', 'Appended Text');

<a name="deleting-files"></a>
### 删除文件

`delete` 方法接受一个文件名称或文件名称数组，用以移除磁盘上的文件：

    Storage::delete('file.jpg');

    Storage::delete(['file1.jpg', 'file2.jpg']);

<a name="directories"></a>
### 目录

#### 获取单一目录内所有文件

`files` 方法返回给定目录下的文件数组。如果你希望返回包含给定目录下所有子目录的文件，你可以使用 `allFiles` 方法。

    $files = Storage::files($directory);

    $files = Storage::allFiles($directory);

#### 获取单一目录内所有目录

`directories` 方法返回给定目录下的目录数组。另外，你也可以使用 `allDirectories` 方法获取给定目录下子目录以及子目录所包含的目录。

    $directories = Storage::directories($directory);

    // Recursive...
    $directories = Storage::allDirectories($directory);

#### 创建目录

`makeDirectory` 方法将创建给定的目录，包括任何所需的子目录。

    Storage::makeDirectory($directory);

#### 删除目录

最后，`deleteDirectory` 方法能移除磁盘上的单一目录以及所包含的全部文件。

    Storage::deleteDirectory($directory);

<a name="custom-filesystems"></a>
## 自定义文件系统

Laravel 集成的 Flysystem 提供许多默认的驱动；然而 Flysystem 本身提供了不仅仅这些，还包括其他保存系统的接口（adapter）。你能在 Laravel 的应用当中通过创建新的驱动使用这些额外的接口。

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
         * Perform post-registration booting of services.
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
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

`extend` 方法的第一个参数是你的驱动名称，第二个参数则是一个接受 `$app` 及 `$config` 变量的闭包。该闭包必须返回 `League\Flysystem\Filesystem` 实例。`$config` 变量包含了定义在 `config/filesystems.php` 对指定磁盘的设置。

当你通过创建服务提供者注册该扩展，你便能在 `config/filesystem.php` 配置文件中使用 `dropbox` 驱动。
