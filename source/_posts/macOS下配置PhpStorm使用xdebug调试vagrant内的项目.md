---
title: macOS下配置PhpStorm使用xdebug调试vagrant内的项目
date: 2017-03-07 13:26:58
tags:
---

最近使用vagrant在macOS下进行PHP的开发，由于一直没法用xdebug调试，开发过程极其痛苦。折腾了半天终于折腾好了。记下步骤，免得下次再折腾。

1. 在php运行环境内配置xdebug，我的vagrant环境是Ubuntu，故安装过程很简单
	```bash
	sudo apt-get install php70-xdebug
	```
	安装后，打开xdebug配置文件,添加如下配置:
	```
	zend_extension=xdebug.so
	xdebug.remote_enable=1
	xdebug.remote_port=19000 #注意，这里的端口最好不要用默认的9000，因为macOS的安全限制还是什么的，9000无发访问。在这里折腾了好久
	xdebug.remote_start =1	  
	xdebug.idekey="vagrant"  #使用和PhpStorm内配置相同的key即可
	xdebug.remote_autostart=1
	xdebug.remote_connect_back=1
	xdebug.remote_handler=dbgp
	```
2. 在PhpStorm内配置PHP运行环境
	{% asset_img 1.png 添加Php运行环境 %}
	*一定要选vagrant*
	{% asset_img 2.png 选择vagrant %}
3. 配置xdebug的监听端口，注意这里的端口要填写和`xdebug.ini`内一致的
	{% asset_img 3.png 配置xdebug监听 %}
4. 填写server配置，其中host和port填写网站的访问地址和端口，文件映射一定要配置正确
	{% asset_img 4.png 配置网站server %}
5. 点击PhpStorm的右上角的下拉的调试配置，`Edit Configurations`,添加`Php Remote Debug`调试配置。其中`ide key`填写和xdebug的配置文件相同的，server选择上一步填写的
	{% asset_img 5.png 添加debug配置 %}
6. 点击右上角的电话按钮和debug按钮，添加断点，开始调试
	{% asset_img 6.png 添加断点并调试 %}