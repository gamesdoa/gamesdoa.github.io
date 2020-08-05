---
title: UOS打包符合商店的deb包
date: 2020-08-05
tags: 
	- uos
	- linux
categories:
    - uos
keywords: uos,deb
description: 如何在UOS系统上打包符合商店规则的deb包。
---

构建一个规范的软件目录
--
- 新建文件夹 com.domainname-version 例如 com.gamesdoa-1.0.0

- 在 com.gamesdoa-1.0.0文件夹下 新建 com.gamesdoa 目录

- 在 com.gamesdoa 目录下新建entries、files两个文件夹和 info 文件

- 在entries 下新建applications文件夹

- 在applications下新建桌面文件appname.desktop 例如 IPBindClient.desktop

		[Desktop Entry]
		Name=IP绑定工具
		Type=Application
		Categories=Utility;
		Exec=/opt/apps/com.gamesdoa/files/IPBindClient
		Icon=/opt/apps/com.gamesdoa/files/IPBindClient.png
		Terminal=false

- **将库和二进制可执行文件放在files目录（二进制可以执行的文件）**

- 修改 info 文件

		info
		{
		"appid":"com.gamesdoa",
		"name":"IP绑定工具",
		"version":"1.0.0",
		"arch":["amd64"],
		"permissions":
		{
		"autostart":false,
		"notification":false,
		"trayicon":true,
		"clipboard":true,
		"account":false,
		"bluetooth":false,
		"camera":false,
		"audio_record":true,
		"installed_apps":false
		}
		}

使用dh_make生成debian目录
--
在com.gamesdoa-1.0.0目录下运行命令

	dh_make --createorig -s

>如果不存在dh_make指令，运行下面命令安装
>
>		sudo apt-get install build-essential dh-make

>如果需要切换超级用户
	
>		sudo  su

确认信息输入y即可

整理自动生成的debian
--
- 修改自动生成debian目录下的control文件

		Source: com.gamesdoa
		Section: utils
		Priority: optional
		Maintainer: unknown <yang@unknown>
		Build-Depends: debhelper (>= 11)
		Standards-Version: 4.1.3
		Homepage: <insert the upstream URL, if relevant>
		#Vcs-Browser: https://salsa.debian.org/debian/com.gamesdoa
		#Vcs-Git: https://salsa.debian.org/debian/com.gamesdoa.git
		
		Package: com.gamesdoa
		Architecture: amd64
		Depends: ${shlibs:Depends}, ${misc:Depends}
		Description: <insert up to 60 chars description>
		 <insert long description, indented with spaces>

	> 注意 Section、Priority、Architecture、Package

- 在debian 目录下新建install文件

		touch install

	在install文件指定安装路径，这里填写
	
		com.gamesdoa/ /opt/apps
		com.gamesdoa/entries/applications/IPBindClient.desktop /usr/share/applications

	> 将com.gamesdoa目录安装到/opt/apps目录下
	
	> 将桌面文件安装到/usr/share/applications目录下（菜单可见）

- 修改rules文件

		#!/usr/bin/make -f
		# See debhelper(7) (uncomment to enable)
		# output every command that modifies files on the build system.
		#export DH_VERBOSE = 1
		
		# see FEATURE AREAS in dpkg-buildflags(1)
		#export DEB_BUILD_MAINT_OPTIONS = hardening=+all
		
		# see ENVIRONMENT in dpkg-buildflags(1)
		# package maintainers to append CFLAGS
		#export DEB_CFLAGS_MAINT_APPEND  = -Wall -pedantic
		# package maintainers to append LDFLAGS
		#export DEB_LDFLAGS_MAINT_APPEND = -Wl,--as-needed
		
		%:
			dh $@
		override_dh_auto_build:
		 
		override_dh_shlibdeps:
		 
		override_dh_strip:
		
		# dh_make generated override targets
		# This is example for Cmake (See https://bugs.debian.org/641051 )
		#override_dh_auto_configure:
		#	dh_auto_configure -- #	-DCMAKE_LIBRARY_PATH=$(DEB_HOST_MULTIARCH)


	> override了三处

- 删除所有EX、ex结尾的文件

		rm *.EX *.ex

编译并生成deb包
--
- 在com.gamesdoa-1.0.0目录下执行(777权限可能导致失败)

		dpkg-source -b .

	> 可能由于gcc版本的问题出错，略过

- 执行下面的命令构建deb包 

		dpkg-buildpackage -us -uc -nc

整理deb包内部结构
--
- 解压到temp目录下
		
		fakeroot dpkg-deb -R com.gamesdoa_1.0.0-1_amd64.deb temp

- 整理包内目录结构

		mv temp/usr/share/doc temp/opt/apps/com.gamesdoa/files

- 最后重新归档安装包

		fakeroot dpkg-deb -b temp com.gamesdoa_1.0.0_amd64.deb