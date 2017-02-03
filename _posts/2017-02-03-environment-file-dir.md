---
layout: post
title:  "Android文件缓存的路径总结"
date:   2017-02-03
categories: 数据持久化
tags: File Environment 数据持久化
---

* content
{:toc}

该文主要Environment类进行翻译总结归纳，同时测试了Context类获取文件存储路径的结果。




### Environment类大揭秘

**Environment有如下8个get方法：**

 - `Environment.getExternalStorageState();`

		该方法用于Returns the current state of the primary "external" storage device.
		
		返回值有以下常量：
		
		`MEDIA_BAD_REMOVAL` 	Sd卡被卸载前已经被移除 。Storage state if the media was removed before it was unmounted.
		
		`MEDIA_CHECKING` 	Sd卡正在进行磁盘检查。 Storage state if the media is present and being disk-checked.
		
		`MEDIA_EJECTING` 	Storage state if the media is in the process of being ejected.
		
		`MEDIA_MOUNTED` 	Sd卡存在，且具有读写权限。Storage state if the media is present and mounted at its mount point with read/write access.
		
		`MEDIA_MOUNTED_READ_ONLY` 	Sd卡存在，仅具有读权限。Storage state if the media is present and mounted at its mount point with read-only access.
		
		`MEDIA_NOFS` 	Sd卡存在，但是空白，或者正在使用不受支持的文件系统。Storage state if the media is present but is blank or is using an unsupported filesystem.
		
		`MEDIA_REMOVED` 	Sd卡被移除。Storage state if the media is not present.
		
		`MEDIA_SHARED` 	Sd卡未安装，通过USB大容量分享。Storage state if the media is present not mounted, and shared via USB mass storage.
		
		`MEDIA_UNKNOWN` 	未知状态。Unknown storage state, such as when a path isn't backed by known storage media.
		
		`MEDIA_UNMOUNTABLE` 	Sd卡存在，但不能被安装。Storage state if the media is present but cannot be mounted.
		
		`MEDIA_UNMOUNTED`  Sd卡存在，但未安装。Storage state if the media is present but not mounted. 

- `Environment.getExternalStorageState(new File(new URI("123")));`

	该方法用于Returns the current state of the storage device that provides the given path. 

- `Environment.getStorageState(new File(new URI("123")));`
	该方法已过时，用上一个方法代替。


 - `Environment.getDataDirectory();`

	该方法用于Return the user data directory.返回值为File类型。

- `Environment.getDownloadCacheDirectory();`

	该方法用于Return the download/cache content directory.返回值为File类型。

- **非常重要**`Environment.getExternalStorageDirectory();`

	该方法用于Return the primary external storage directory.返回值为File类型。 当用户将手机和电脑连接在一起，此时存储设备被移除了，或者其他问题，返回的目录有可能是不能用的，需判定当前状态。此时不要对External一词感到疑惑，仅指该目录可以更好的存储用于分享的内容媒介。 它是一个可以持有想当多的数据的文件系统，该系统可以被所有应用访问但不需要任何权限。一般来说它是Sd卡，但“外部存储”也可以使内置的存储设备来实现，此时“外部存储”是区别于受保护的内部存储的， 并且“外部存储”可以作为一个文件系统安装在电脑上。
	在具有多用户的设备上（由UserManager描述），每一个用户有它们自己独立的外部存储。应用程序们在运行的时候可以仅访问用户的外部存储。具有多个外部存储目录的设备，一个外部存储目录意味着一个用户可以交互的主要的外部存储。为了避免玷污该用户的根命名空间，因此应用程序不应该直接使用这个顶级目录，可以访问（使用）次级存储目录。对于一个应用程序而言，任何私有的文件都应该存储在由`Context.getExternalFilesDir`方法返回的目录下，当该应用被卸载的时候，系统将考虑删除这个文件。其他共享型的文件则应该存放于由Environment类的`getExternalStoragePublicDirectory(String)`方法返回的目录下。拥有写的权限就默认拥有读的权限。从Android的KITKAT版本开始，如果我们的应用程序仅需存储内部数据，应该使用Context的`getExternalFilesDir(String)`方法与`getExternalCacheDir()`方法，这不需要读与写的权限。这两个方法返回的路径在不同的设备平台上可能会发生改变，因此应用程序应该仅仅持有相对路径。

	On devices with multiple users (as described by UserManager), each user has their own isolated external storage. Applications only have access to the external storage for the user they're running as.  

- **非常重要**`Environment.getExternalStoragePublicDirectory(Environment.TYPE);`

		该方法用于Get a top-level public external storage directory for placing files of a particular type. 返回值为File类型。

		获得一个用于存储一个特殊类型文件的顶级公开外部存储目录。用户通常在这个目录存储管理它们自己的文件。因此我们应该考虑放在此处的文件内容，以确保我们不会删除他们的文件或者阻碍他们自己的管理。

		参数主要有:
		DIRECTORY_ALARMS 	闹钟铃声文件类型。Standard directory in which to place any audio files that should be in the list of alarms that the user can select (not as regular music).
	
		DIRECTORY_DCIM 	相机的图片或视频类型。The traditional location for pictures and videos when mounting the device as a camera.
	
		DIRECTORY_DOCUMENTS 	文档类型。Standard directory in which to place documents that have been created by the user.
	
		DIRECTORY_DOWNLOADS 	用户下载的文件。Standard directory in which to place files that have been downloaded by the user.
	
		DIRECTORY_MOVIES 	用户可用的电影。Standard directory in which to place movies that are available to the user.
	
		DIRECTORY_MUSIC 	标准音乐文件。Standard directory in which to place any audio files that should be in the regular list of music for the user.
	
		DIRECTORY_NOTIFICATIONS 	音频文件通知铃声。Standard directory in which to place any audio files that should be in the list of notifications that the user can select (not as regular music).
	
		DIRECTORY_PICTURES 	用户可用的图片Standard directory in which to place pictures that are available to the user.
	
		DIRECTORY_PODCASTS 	播客音频。Standard directory in which to place any audio files that should be in the list of podcasts that the user can select (not as regular music).
	
		DIRECTORY_RINGTONES 	手机铃声音频。Standard directory in which to place any audio files that should be in the list of ringtones that the user can select (not as regular music). 

- `Environment.getRootDirectory();`

	该方法用于Return root of the "system" partition holding the core Android OS. 返回值为File类型。


**Environment有如下4个is方法：**

- `Environment.isExternalStorageEmulated();`与`Environment.isExternalStorageEmulated(new File(new URI("123")));`

	这两个方法用于：Returns whether the primary "external" storage device is emulated. If true, data stored on this device will be stored on a portion of the internal storage system.


- `Environment.isExternalStorageRemovable();`与`Environment.isExternalStorageRemovable(new File(new URI("123")));`

	这两个方法用于：true if the storage device can be removed (such as an SD card), or false if the storage device is built in and cannot be physically removed. 

        



### Context获取文件路径大揭秘

Context对象有如下四个方法用户获取文件（目录）路径：

- `getDir(String name,int mode);`

	检索，必要时创建，一个新目录。应用程序在该目录下可以放置它自己的数据文件。我们可以使用该方法返回的目录对象创建或者访问该目录下的文件。需要注意的是通过文件对象创建的文件仅对该应用程序可见；我们仅仅可以设置整个目录的mode，而非单个文件。

- `openFileOutput(String name,int mode);`

	打开与此上下文相关联的应用程序包的一个私有文件。如果这个文件不存在就创建它。由于属于内部存储，不需要权限。

- `getFilesDir();`

	返回由openFileOutput（String，int）方法创建的用于存储文件的文件系统的绝对路径。因为这个路径是输出内部存储的，因此读写该目录下的文件是不需要权限的。

- `getExternalFilesDir(String type);`

	该方法的参数使用的是Environment的成员变量，返回主要外部存储文件系统的绝对路径（同`Environment.getExternalStorageDirectory()`），应用程序可以在该路径下持久化它自己的文件。这些作为应用程序内部的文件，并没有像媒体一样通常对用户可见（通常是不可见的）。和getFilesDir（）方法返回的目录下的文件一样，当应用卸载的时候这些文件将被删除，但二者有如下明显的不同：

1. 外部文件不总是可使用的，如果用户拔掉了外部存储，或者外部存储挂载到了电脑上，其中的文件将消失。
2. 没有安全执行这些文件。例如，任何一个具备`WRITE_EXTERNAL_STORAGE`权限的应用都可以写这些文件。

	从Android的KITKAT开始，读写该方法返回的路径下的内容是不需要权限的；它总是给调用应用程序访问，这仅仅适用于调用程序的包名路径生成。

[参考1](https://www.zhihu.com/question/22996204?sort=created)

[参考2](http://blog.csdn.net/jaycee110905/article/details/21130557)

[参考3](http://blog.csdn.net/dalancon/article/details/17416041)


代码为：`String path = Envoronment.getExternalStorageDirectory().getPath +"BeautifulSchool";`

路径为`/storage/emulated/0/BeautifulSchool/文件名`

代码为：`String path = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES).getPath() + "BeautifulSchool";`

路径为`/storage/emulated/0/Pictures/BeautifulSchool/文件名`

代码为：`String path = context.getFileDir()+"BeautifulSchool";`

路径为`/data/user/0/com.qcsd99.www/files/BeautifulSchool/文件名`

代码为：`String path = context.getExternalFilesDir(null)+"BeautifulSchool";`

路径为`/storage/emulated/0/Android/data/com.qcsd99.www/files/BeautifulSchool/文件名`