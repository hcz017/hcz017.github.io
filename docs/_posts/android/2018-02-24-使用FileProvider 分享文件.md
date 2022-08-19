---
date: 2018-02-24 15:28
status: public
title: '使用FileProvider 分享文件'
---

# 起因
我们的截图应用在Android O 上使用分享功能的时候crash了，错误关键词：**FileUriExposedException**。
Google 一下发现这个问题从Android N 开始出现的，当你给使用 *file:/// Uri* 分享文件的时候会抛出这个[异常](https://developer.android.com/reference/android/os/FileUriExposedException.html)。
但是奇怪的是我们在Android N 上使用分享功能的时候并没有出现问题，不管怎样有问题就要解决。
一句话概括，我们要做的就是使用 *content:// Uri* 代替 *file:/// Uri*。

# 解决过程
报错代码：
```java
    public void shareScreenshot() {
        final File imageFile = new File(mScreenshotPath);
        // Create a shareScreenshot intent
        Intent sharingIntent = new Intent(Intent.ACTION_SEND);
        sharingIntent.setType("image/png");
        sharingIntent.putExtra(Intent.EXTRA_STREAM, Uri.fromFile(imageFile));
        startActivity(Intent.createChooser(sharingIntent, getResources().getText(R.string.share)));
    }
```

当我们使用*Uri.fromFile(imageFile)*的时候获得了 *file:/// Uri*，就是使用这个 *file:/// Uri* 的时候报的错。
下面我们分几步把*file:/// Uri* 替换成 *content:// Uri*。

1. 指定FileProvider
   在AndroidManifest.xml 中定义FileProvider。
关注两个值，*android:authorities* 和*android:resource*，前者是包名跟.*fileprovider*， 后者是定义你想要访问的目录的文件，下面会细说。
   ```xml
   <manifest xmlns:android="http://schemas.android.com/apk/res/android"
       package="com.example.myapp">
       <application
           ...>
           <provider
               android:name="android.support.v4.content.FileProvider"
               android:authorities="com.ckt.screenshot.provider"
               android:grantUriPermissions="true"
               android:exported="false">
               <meta-data
                   android:name="android.support.FILE_PROVIDER_PATHS"
                   android:resource="@xml/provider_paths"/>
           </provider>
           ...
       </application>
   </manifest>
   ```

2. 指定可共享的目录
   在res/xml/创建和上面*android:resource *对应的文件provider_paths.xml。

   ```xml
   <paths>
       <files-path path="images/" name="myimages" /> <!--仅作举例-->
       <external-path name="external_files" path="Pictures/Screenshots/"/>
   </paths>
   ```
   在这个例子中*files-path* 标签代表了应用内部存储files/子目录下的文件， *external-path* 标签代表外置存储根目录。其他标签请查看[FileProvider](https://developer.android.com/reference/android/support/v4/content/FileProvider.html) 介绍。
   - ***path*** 代表的是**子目录**。它是标签代表的目录的子目录。比如上例中的两个目录，第一个是应用内部存储files/下的images/ 子目录，另一个是存储卡根目录下的Pictures/Screenshots/ 子目录。
注意： 不能用指定文件名的方式分享特定文件，也不能用通配符的方式指定一些文件。
   - ***name*** 是content Uri 中的一段，它可以说是path的**别名**，为了加强安全性，用name的值代替实际的path 路径添加到content Uri中；


3. 修改java 代码
   使用*FileProvider.getUriForFile* 获得content Uri。
   ```java
   +        Uri imageUri = FileProvider.getUriForFile(mContext,
   +                BuildConfig.APPLICATION_ID + ".provider",
   +                imageFile);
            // Create a shareScreenshot intent
            Intent sharingIntent = new Intent(Intent.ACTION_SEND);
            sharingIntent.setType("image/png");
   -        sharingIntent.putExtra(Intent.EXTRA_STREAM, Uri.fromFile(imageFile));
   +        sharingIntent.putExtra(Intent.EXTRA_STREAM, imageUri);
   ```
   实际文件路径 imageFile /storage/emulated/0/**Pictures/Screenshots**/Screenshot_20180224-104038.png
   而我们获得 imageUri content://com.ckt.screenshot.provider/**external_files**/Screenshot_20180224-104038.png

   结合*provider_paths.xml* 对比可以看出Pictures/Screenshots（path 实际路径）替换成了external_files（name 别名）。

# 不使用Uri.fromFile() 的几点理由
- Does not allow file sharing across profiles.
- 要求分享文件的应用有WRITE_EXTERNAL_STORAGE 的权限（Android 4.4 及以下）
- 要求接收文件的应用有 READ_EXTERNAL_STORAGE的权限，分享文件给没有这些权限的应用会失败，比如Gmail就没有这个权限。
最重要的一点，在Android N及以上会出现Crash。


# 参考连接
1. [FileUriExposedException](https://developer.android.com/reference/android/os/FileUriExposedException.html)
2. [FileProvider](https://developer.android.com/reference/android/support/v4/content/FileProvider.html)
3. [Setting Up File Sharing](https://developer.android.com/training/secure-file-sharing/setup-sharing.html#DefineMetaData)
4. [android.os.FileUriExposedException: file:///storage/emulated/0/test.txt exposed beyond app through Intent.getData()](https://stackoverflow.com/questions/38200282/android-os-fileuriexposedexception-file-storage-emulated-0-test-txt-exposed)
5. [解决 Android N 7.0 上 报错：android.os.FileUriExposedException](http://blog.csdn.net/yy1300326388/article/details/52787853)