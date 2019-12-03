---
title: SpringBoot 图片上传
toc: true
layout: SpringBoot
categories:
  - SpringBoot
tags:
  - SpringBoot
  - EnableScheduling
date: 2019-02-28 11:00:13
---
{{title}}
<!-- more -->
# SpringBoot 实现图片上传功能 

> SpringBoot 是为了简化Spring应用的创建、运行、测试、调试、部署等一系列问题而诞生的产物，自动装配的特性让我们可以更好的关注业务本事而不是外部的xml配置，我们只需遵循规范，引入相关的依赖就可以轻易搭建出一个web工程 

**说明**

* 文件上传控件较多，本文以引用`LayUI`图片上传控件为例
* 页面上传插件使用Layui的图片上传控件，<a href="http://www.layui.com/demo/upload.html" target="_blank">点击跳转</a>	
* 文件上传使用JQuery 的Ajax请求对应图片上传API，API返回包含图片路径格式的JSON数据	
* 上传图片路径并非在项目文件夹中，需配置静态资源文件路径	

## 配置文件上传控件	

> 第一步：添加 `Layui` 的文件上传控件	
	
1. 将`Layui`库添加进项目的静态资源文件夹中，在`src/main/resources/satic`文件夹下创建`layui`文件夹存放`Layui`库等资源文件;	

2. `HTML`页面引用js及css文件，图片上传控件需要用到的资源有`jquery.mini.js/jquery.js、layer.js、layui.js、layui.css`。相关资源文件路径需要配置为项目具体引用路径或使用在线配置路径`layui`，详细内容可到<a href="http://www.layui.com" target="_blank">`layui官网`</a>查看。

3. 初始化插件及样式	

```js
<script type="text/javascript">
layui.use('upload',function() {
	var $ = layui.jquery, upload = layui.upload;
	//普通图片上传
	var uploadInst = upload.render({
		url : '../picture/uploadImg'//图片上传action路径配置
		,data : {
			folder : 'hotel/'
		},
		before : function(obj) {
			//预读本地文件示例，不支持ie8
			obj.preview(function(index, file,
					result) {
				$('#imgSrc').attr('src', result); //图片链接（base64）,显示缩略图
			});
		},
		done : function(data) {
			if (data.data.message == 'success') {
				$('#hotelPicture').val(
						data.data.picUrl);//设置图片相对路径，表单提交时提交图片url
				return layer.msg('上传成功!')
			}
		},
		error : function() {
			//演示失败状态，并实现重传
			var demoText = $('#demoText');
			demoText
					.html('<span style="color: #FF5722;">上传失败</span> <a class="layui-btn layui-btn-mini demo-reload">重试</a>');
			demoText.find('.demo-reload').on(
					'click', function() {
						uploadInst.upload();
					});
		}
	});
});
</script>
```	

##  修改配置文件 

> 第二步: 修改配置文件 

```properties
# file write static resource path
web.upload-path=F:/temp/springboot/

spring.mvc.static-path-pattern=/**
spring.resources.static-locations=classpath:/templates/,classpath:/META-INF/resources/,classpath:/resources/,\
  classpath:/static/,classpath:/public/,file:${web.upload-path}
```

注： web.upload-path 为图片保存路径，若使用linux系统部署需要将绝对路径替换为相对路径。	例如：

## API 接口	

> 第三步：文件上传API	

	文件上传API待更新

## 页面显示配置	

> 第四步：图片在前端页面显示路径配置，不配置显示不了图片	

```java
package cn.com.weiyi.wisdomh.Config;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
public class WebAppConfig extends WebMvcConfigurerAdapter {
    @Value("${web.upload-path}")
    private String imgPath;//application.properties中的文件资源位置
    
    /**
     * 图片访问方法
     */
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        if(imgPath.equals("") || imgPath.equals("${web.upload-path}")){
            String imagesPath = WebAppConfig.class.getClassLoader().getResource("").getPath();
            if(imagesPath.indexOf(".jar")>0){
                imagesPath = imagesPath.substring(0, imagesPath.indexOf(".jar"));
            }else if(imagesPath.indexOf("classes")>0){
                imagesPath = "file:"+imagesPath.substring(0, imagesPath.indexOf("classes"));
            }
            imagesPath = imagesPath.substring(0, imagesPath.lastIndexOf("/"))+"/images/";
            imgPath = imagesPath;
        }
        LoggerFactory.getLogger(WebAppConfig.class).info("imagesPath="+imgPath);
        registry.addResourceHandler("/images/**").addResourceLocations(imgPath);
        super.addResourceHandlers(registry);
    }
}
```