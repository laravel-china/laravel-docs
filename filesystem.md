# 文件系统与云存储

- [简介](#introduction)
- [设置](#configuration)
    - [公开磁盘](#the-public-disk)
    - [本地端驱动](#the-local-driver)
    - [驱动的预先需求](#driver-prerequisites)
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

Laravel 强大的文件抽象层得力于 Frank de Jonge 的 [Flysystem](https://github.com/thephpleague/flysystem) 扩展包。Laravel 的 flysystem 集成提供了可给本地端磁盘系统、Amazon S3、以及 Rackspace 云存储使用的各种驱动（driver）。并且能像使用 API 一样，轻易的切换这些存储方式来应对各式系统。

<a name="configuration"></a>
## 设置

文件系统配置文件位于 `config/filesystems.php`。该文件能让你设置所有的「磁盘（disk）」。每个磁盘代表一个唯一的存储驱动以及存储位置。各种支持驱动的例子已包含其中，仅需要简单的根据你的偏好配置及凭证设置进行修改即可。

当然，你也可以设置多组磁盘，甚至在多个磁盘使用相同的驱动。

<a name="the-public-disk"></a>
### 公开磁盘

公开磁盘意味着你的所有文件将可以公开访问，默认的 `public` 磁盘使用 `local` 驱动，并且存储文件至 `storage/app/public` 文件夹中。为了能公开访问，你需要创建 `public/storage` 文件夹，然后作为符号链接到 `storage/app/public` 文件夹。这个约定能使你在使用 [Envoyer](https://envoyer.io) 类似的部署器的时候保证文件夹配置的统一。

你就可以使用 `storage:link` Artisan 命令创建符号链接:

    php artisan storage:link

当符号链接创建完毕以后，你就可以使用 `asset` 辅助函数创建共用的 URL：

    echo asset('storage/file.txt');

<a name="the-local-driver"></a>
### 本地端驱动

当使用 `local` 驱动时，所有的操作都是相对于配置文件的 `root` 目录设置进行的。该目录默认是 `storage/app`。因此下列方法将把文件保存在 `storage/app/file.txt`：

    Storage::disk('local')->put('file.txt', 'Contents');

<a name="driver-prerequisites"></a>
### 驱动的预先需求

#### Composer 包
在使用 S3 或 Rackspace 驱动之前，你需要通过 Composer 安装适当扩展包：

- Amazon S3: `league/flysystem-aws-s3-v3 ~1.0`
- Rackspace: `league/flysystem-rackspace ~1.0`

#### S3 驱动配置

S3 驱动配置信息存储在 `config/filesystems.php` 配置文件里。 这个文件有S3 驱动的配置数组的例子。你可以随意编辑

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

<a name="obtaining-disk-instances"></a>
## 获得磁盘实例

`Storage` facade 用于对任何已设置的磁盘进行交互。举例来说，你可以使用 facade 的 `put` 方法将一张头像保存到默认磁盘上。当使用 `Storage` facade 调用了方法却未先调用 `disk` 方法时，默认磁盘将被自动传递给该方法。


    use Illuminate\Support\Facades\Storage;

    Storage::put('avatars/1', $fileContents);

当使用多个磁盘时，你可以通过 `Storage` facade 的 `disk` 方法访问指定磁盘。当然，你也可以使用链式调用（chain methods）对磁盘使用各种执行方法。

    Storage::disk('s3')->put('avatars/1', $fileContents);

<a name="retrieving-files"></a>
## 提取文件

`get` 方法提取指定文件的内容，该文件的原始字符串内容将通过该方法获取：该文件的原始字符串的内容将通过该方法返回。 所有的操作都是相对于配置文件的 `root` 目录设置进行的。

    $contents = Storage::get('file.jpg');

`exists` 方法可以被用于确定是否一个文件在盘上存在：

    $exists = Storage::disk('s3')->exists('file.jpg');

<a name="file-urls"></a>
### 文件 URLs

当使用 `local` 或者 `s3` 驱动的时候，你可以使用 `url` 方法来获取文件的 URL。如果你使用 `local` 驱动，一般会在传参的路径前面加上 `/storage`。如果是 `s3` 的话，会返回完整的 S3 文件系统的 URL:


    use Illuminate\Support\Facades\Storage;

    $url = Storage::url('file1.jpg');

> {note} 当使用 `local` 驱动的时候，请确定 [创建符号链接到 `public/storage`](#the-public-disk) 来指向 `storage/app/public` 文件夹。

<a name="file-metadata"></a>
### 文件元数据

除了读取和写入文件，Laravel还可以提供有关文件本身的信息。例如，`size` 方法可被用于获得以字节的文件的大小：

    use Illuminate\Support\Facades\Storage;

    $size = Storage::size('file1.jpg');

`lastModified` 方法返回的最后一次文件被修改的 UNIX 时间戳：

    $time = Storage::lastModified('file1.jpg');

<a name="storing-files"></a>
## 保存文件

`put` 方法保存单个文件于磁盘上。你能同时传递 PHP 的 `resource` 给 `put` 方法，它将使用文件系统底层的 stream 支持。强烈建议使用 stream 处理大型文件：

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents);

    Storage::put('file.jpg', $resource);

#### 自动流

如果您想 Laravel 自动管理指定文件流传输到您想要的存储位置，你可以使用 `putFile` 或 `putFileAs` 方法。这个方法可以接受一个 `Illuminate\HTTP\File` 或 `Illuminate\HTTP\UploadedFile` 实例，并自动将文件传输到你想要的位置：

    use Illuminate\Http\File;

    // Automatically calculate MD5 hash for file name...
    Storage::putFile('photos', new File('/path/to/photo'));

    // 手动指定一个文件名...
    Storage::putFile('photos', new File('/path/to/photo'), 'photo.jpg');

还有要注意的有关 `putFile` 方法的一些重要的事情。请注意，我们只指定一个目录名，而不是文件名。默认情况下，该 `putFile` 方法将自动基于该文件的内容而生成。这是通过的文件内容的 MD5 哈希来完成的。该文件的路径将被 `putFile` 方法被返回，因此您可以在数据库中存储路径和包括生成的文件名。

该 `putFile` 和 `putFileAs` 方法也接受一个参数来保存制定文件的“可见性”。如果正在使用云存储磁盘，如 S3 ，并希望该文件可以被公开访问，这将非常有用：

    Storage::putFile('photos', new File('/path/to/photo'), 'public');

#### 插入到文件

`prepend` 及 `append` 方法允许你轻易的将内容插入到一个文件的开头或结尾：

    Storage::prepend('file.log', 'Prepended Text');

    Storage::append('file.log', 'Appended Text');

#### 复制 & 移动文件

`copy` 方法用于复制一个已存在的文件到磁盘的新位置。
`move` 方法用于重命名或是移动一个已存在的文件到新位置：

    Storage::copy('old/file1.jpg', 'new/file1.jpg');

    Storage::move('old/file1.jpg', 'new/file1.jpg');

<a name="file-uploads"></a>
### 文件上传

在 Web 应用程序中，最常见的例子之一就是存储文件是存储用户上传的文件，如个人资料图片，照片和文档。 Laravel 通过使用 `store` 方法，使得它在上传文件的实例中非常容易的存储上传了的文件。只需使用你想存储的路径来调用 `store` 方法即可：


    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserAvatarController extends Controller
    {
        /**
         * 上传 avatar 给用户.
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

需要注意这个例子中一些重要的事情。我们只指定一个目录名，而不是文件名。默认情况下，`store` 方法将基于文件的内容自动生成文件名。这是通过获取该文件内容的 MD5 散列来实现的。该文件的路径将被 `store` 方法被退回，因此您可以在数据库中存储路径和包括生成的文件名。

您也可以调用在`Storage` facade `putFile` 方法来执行和上面例子相同的文件操作：


    $path = Storage::putFile('avatars', $request->file('avatar'));

> {note} 如果您要上传非常大的文件，你不妨手动指定文件名。计算大的文件文件的MD5哈希需要占用大量内存。

#### 指定文件名

如果你不喜欢的文件名被自动分配给您的存储的文件，你可以使用 `storeAs` 方法，它接收的路径，文件名。选择的磁盘作为它的参数：


    $path = $request->file('avatar')->storeAs(
        'avatars', $request->user()->id
    );
当然，你也可以在 `putFileAs` 方法里使用 `Storage` ，将执行上面的例子中相同的文件操作：

    $path = Storage::putFileAs(
        'avatars', $request->file('avatar'), $request->user()->id
    );

#### 指定磁盘

默认情况下，此方法将使用默认的磁盘。如果您想指定其他磁盘，通过磁盘名称作为第二个参数 `store` 方法：

    $path = $request->file('avatar')->store(
        'avatars/'.$request->user()->id, 's3'
    );

<a name="file-visibility"></a>
### 文件可见性

在Laravel的Flysystem整合，“可见性”是多平台上的文件权限的抽象层。你也可以选择在使用 `put` 创建文件时设置可见性，可见性的值为 `public` 或者是 `private` 。当一个文件为 `public` ，要表示文件一般应给他人使用。例如，使用S3驱动程序时，您可以检索 `public` 文件的URL。

使用 `put` 在创建文件时设置可见性：

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents, 'public');

如果文件已经被保存，那么可见性可以通过 `getVisibility` 来获取和 `setVisibility` 方法来设置。

    $visibility = Storage::getVisibility('file.jpg');

    Storage::setVisibility('file.jpg', 'public')

<a name="deleting-files"></a>
## 删除文件

`delete` 方法接受一个文件名称或文件名称数组，用以移除磁盘上的文件：

    use Illuminate\Support\Facades\Storage;

    Storage::delete('file.jpg');

    Storage::delete(['file1.jpg', 'file2.jpg']);

<a name="directories"></a>
## 目录

#### 获取单个目录内的所有文件

`files` 方法返回指定目录下的文件数组。如果你希望返回包含指定目录下所有子目录的文件，则可以使用 `allFiles` 方法。


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

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@LXY](https://github.com/dongli0)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5832_1473813539.jpeg?imageView2/1/w/380/h/380">  |  翻译  | PHP 小学生。 |
