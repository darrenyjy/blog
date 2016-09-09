---
title: maven-resources-plugin:2.6 or one of its dependencies could not be resolved
date: 2016-05-25 21:37:00
tags:
- maven
categories:
- maven
---
本猿想把maven默认仓库改成自己文件夹的时候不小心删掉了`` ~/.m2/repository ``里面所有的jar。当再次install时出现下面的错误。

``` bash
	jinyaoyuan:maven jinyaoyuan$ mvn install
	[INFO] Scanning for projects...
	[INFO]                                                                         
	[INFO] ------------------------------------------------------------------------
	[INFO] Building hello-first SNAPSHOT-0.0.1
	[INFO] ------------------------------------------------------------------------
	Downloading: https://repo.maven.apache.org/maven2/org/apache/maven/plugins/maven-resources-plugin/2.6/maven-resources-plugin-2.6.pom
	[INFO] ------------------------------------------------------------------------
	[INFO] BUILD FAILURE
	[INFO] ------------------------------------------------------------------------
	[INFO] Total time: 1.021 s
	[INFO] Finished at: 2016-05-25T12:20:48+08:00
	[INFO] Final Memory: 7M/155M
	[INFO] ------------------------------------------------------------------------
	[ERROR] Plugin org.apache.maven.plugins:maven-resources-plugin:2.6 or one of its dependencies could not be resolved: Failed to read artifact descriptor for org.apache.maven.plugins:maven-resources-plugin:jar:2.6: Could not transfer artifact org.apache.maven.plugins:maven-resources-plugin:pom:2.6 from/to central (https://repo.maven.apache.org/maven2): sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target -> [Help 1]
	[ERROR] 
	[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
	[ERROR] Re-run Maven using the -X switch to enable full debug logging.
	[ERROR] 
	[ERROR] For more information about the errors and possible solutions, please read the following articles:
	[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/PluginResolutionException```
看日志发现downloading不下来，以后是网络问题，直接访问链接发现可以访问，尼玛这是什么问题呢。找了很多blog，发现自己前段时间做cas时改了jre中lib库里的security里面的一个文件cacerts。然后恢复成原来的就OK了，简直神奇啊，那cacerts这个文件到底是干嘛的呢？这个文件是一个证书，有心的读者会看到上面的链接是https开头的，访问的时候客户端是需要这个cacerts文件授权滴。
	
