---
title: 普通 Java/Jave Web 转为 Maven 项目
toc: true
layout: blog
categories:
  - blog
  - Java
tags:
  - Java
  - Maven
date: 2018/07/19 9:55:23
---
{{title}}
<!-- more -->
# Jar文件 更改为 pom.xml 依赖	

> 只需要运行方法即可	

```java
package com.sanss.util;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.util.jar.JarInputStream;
import java.util.jar.Manifest;

import org.dom4j.Element;
import org.dom4j.dom.DOMElement;
import org.jsoup.Jsoup;

import com.alibaba.fastjson.JSONObject;

/**
 * 
 * @ClassName: MakePomFromJars
 * @author: 
 * @Date: 2018/06/21
 * @Description: 将jar 包生pom依赖
 *
 */
public class MakePomFromJars {

	public static void main(String[] args) throws FileNotFoundException, IOException {
		Element dependencys = new DOMElement("dependencies");
		File dir = new File("D:\\Code\\sts-work-workspace\\iptvView\\WebRoot\\WEB-INF\\lib");// 生pom文件的lib路径

		System.out.println("读取文件路径为:\t" + dir.getPath());
		System.out.println("开始读取文件: >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>");

		StringBuffer missJar = new StringBuffer();
		for (File jar : dir.listFiles()) {
			JarInputStream jis = new JarInputStream(new FileInputStream(jar));
			Manifest manifest = jis.getManifest();
			jis.close();
			if (manifest == null) {
				continue;
			}
			String bundleName = manifest.getMainAttributes().getValue("Bundle-Name");
			String bundleVersion = manifest.getMainAttributes().getValue("Bundle-Version");
			Element element = null;

			System.out.println(jar.getName());
			StringBuffer sb = new StringBuffer(jar.getName());
			if (bundleName != null) {
				bundleName = bundleName.toLowerCase().replace(" ", "-");
				sb.append(bundleName + "\t" + bundleVersion);
				element = getDependices(bundleName, bundleVersion);
				// System.out.println(sb.toString());
				// System.out.println(element.asXML());
			}

			if (element == null || element.elements().size() == 0) {
				bundleName = "";
				bundleVersion = "";

				String[] ns = jar.getName().replace(".jar", "").split("-");
				for (String s : ns) {
					if (Character.isDigit(s.charAt(0))) {
						bundleVersion += s + "-";
					} else {
						bundleName += s + "-";
					}
				}

				if (bundleVersion.endsWith("-")) {
					bundleVersion = bundleVersion.substring(0, bundleVersion.length() - 1);
				}
				if (bundleName.endsWith("-")) {
					bundleName = bundleName.substring(0, bundleName.length() - 1);
				}
				element = getDependices(bundleName, bundleVersion);
				sb.setLength(0);
				sb.append(bundleName + "\t").append(bundleVersion);

				// System.out.println(sb.toString());
				// System.out.println(element.asXML());
			}
			element = getDependices(bundleName, bundleVersion);
			if (element.elements().size() == 0) {
				element.add(new DOMElement("groupId").addText("not found"));
				element.add(new DOMElement("artifactId").addText(bundleName));
				element.add(new DOMElement("version").addText(bundleVersion));

				missJar.append(bundleName + "\n");// 统计缺失的jar名称
			} else {
				dependencys.add(element);
				// System.out.println();
			}
		}
		System.err.println(missJar);
		System.out.println(dependencys.asXML());
	}

	private static Element getDependices(String key, String ver) {
		Element dependency = new DOMElement("dependency");
		// 设置代理
		// System.setProperty("http.proxyHost","127.0.0.1");
		// System.setProperty("http.proxyPort", "8090");
		try {
			String url = "http://search.maven.org/solrsearch/select?q=a%3A%22" + key + "%22%20AND%20v%3A%22" + ver
					+ "%22&rows=3&wt=json";
			org.jsoup.nodes.Document doc = Jsoup.connect(url).ignoreContentType(true).timeout(30000).get();
			String elem = doc.body().text();
			JSONObject response = JSONObject.parseObject(elem).getJSONObject("response");
			if (response.containsKey("docs") && response.getJSONArray("docs").size() > 0) {
				JSONObject docJson = response.getJSONArray("docs").getJSONObject(0);
				Element groupId = new DOMElement("groupId");
				Element artifactId = new DOMElement("artifactId");
				Element version = new DOMElement("version");
				groupId.addText(docJson.getString("g"));
				artifactId.addText(docJson.getString("a"));
				version.addText(docJson.getString("v"));

				dependency.add(groupId);
				dependency.add(artifactId);
				dependency.add(version);
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
		return dependency;
	}
}
```
