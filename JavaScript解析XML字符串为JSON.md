---
title: JavaScript解析XML字符串为JSON
toc: true
layout: blog
categories:
  - Blog
  - HTML
tags:
  - Blog
  - XML
  - JSON
date: 2019-06-28 10:13:21
---
{{ title }}
<!-- more -->

# `JavaScript`使用`ObjTree.js`解析`xml`文件为`JSON`对象

- 插件下载地址<a href="http://www.kawa.net/works/js/xml/objtree-e.html#download" target="_blank">http://www.kawa.net/works/js/xml/objtree-e.html#download</a>

- xml文件内容

```xml
<?xml version="1.0" encoding="utf-8"?>
<Organization>
    <Department coding="001" name="根节点" modifytime="1436940546" MaxDepID="0" sn="" memo="" deptype="0" depsort="0" isPlatform="0" chargebooth="0">
        <Device id="1000019001" type="65537" name="海康106" manufacturer="2" ip="127.0.0.2" prot="61001" unitnum="1" status="2" rights="110000111001111">
            <UnitNodes index="0" channelnum="48" streamType="81" type="1" subType="0" zeroChnEncode="0">
                <Channel id="10000978001@001$1$0$0" name="GB49_1-" desc="" channelType="1" channelSN="10005"></Channel>
                <Channel id="10000978001@001$1$0$1" name="GB49_1-" desc="" channelType="1" channelSN="10005" ></Channel>
                <Channel id="10000978001@001$1$0$2" name="GB49_1-" desc="" channelType="1" channelSN="10005" ></Channel>
            </UnitNodes>
        </Device>
        <Device id="1000018001" type="65548" name="海康106" manufacturer="1" ip="127.0.0.2" prot="61001" unitnum="2" status="2" rights="110000111001111">
            <UnitNodes index="0" channelnum="48" streamType="81" type="1" subType="0" zeroChnEncode="0">
                <Channel id="10000978001@001$1$0$0" name="GB49_1-" desc="" channelType="1" channelSN="10005"></Channel>
                <Channel id="10000978001@001$1$0$1" name="GB49_1-" desc="" channelType="1" channelSN="10005" ></Channel>
                <Channel id="10000978001@001$1$0$2" name="GB49_1-" desc="" channelType="1" channelSN="10005" ></Channel>
            </UnitNodes>
        </Device>
        <Device id="1000017001" type="65548" name="海康106" manufacturer="1" ip="127.0.0.2" prot="61001" unitnum="3" status="2" rights="110000111001111">
            <UnitNodes index="0" channelnum="48" streamType="81" type="1" subType="0" zeroChnEncode="0">
                <Channel id="10000978001@001$1$0$0" name="GB49_1-" desc="" channelType="1" channelSN="10005"></Channel>
                <Channel id="10000978001@001$1$0$1" name="GB49_1-" desc="" channelType="1" channelSN="10005" ></Channel>
                <Channel id="10000978001@001$1$0$2" name="GB49_1-" desc="" channelType="1" channelSN="10005" ></Channel>
            </UnitNodes>
        </Device>
        <Device id="1000016001" type="65548" name="海康106" manufacturer="1" ip="127.0.0.2" prot="61001" unitnum="4" status="2" rights="110000111001111">
            <UnitNodes index="0" channelnum="48" streamType="81" type="1" subType="0" zeroChnEncode="0">
                <Channel id="10000978001@001$1$0$0" name="GB49_1-" desc="" channelType="1" channelSN="10005"></Channel>
                <Channel id="10000978001@001$1$0$1" name="GB49_1-" desc="" channelType="1" channelSN="10005" ></Channel>
                <Channel id="10000978001@001$1$0$2" name="GB49_1-" desc="" channelType="1" channelSN="10005" ></Channel>
            </UnitNodes>
        </Device>
        <Device id="1000015001" type="65548" name="海康106" manufacturer="2" ip="127.0.0.2" prot="61001" unitnum="5" status="2" rights="110000111001111">
            <UnitNodes index="0" channelnum="48" streamType="81" type="1" subType="0" zeroChnEncode="0">
                <Channel id="10000978001@001$1$0$0" name="GB49_1-" desc="" channelType="1" channelSN="10005"></Channel>
                <Channel id="10000978001@001$1$0$1" name="GB49_1-" desc="" channelType="1" channelSN="10005" ></Channel>
                <Channel id="10000978001@001$1$0$2" name="GB49_1-" desc="" channelType="1" channelSN="10005" ></Channel>
            </UnitNodes>
        </Device>
    </Department>
    <Department coding="001" name="根节点" modifytime="1436940546" MaxDepID="0" sn="" memo="" deptype="0" depsort="0" isPlatform="0" chargebooth="0">
        <Device id="1000019001" type="65537" name="海康106" manufacturer="2" ip="127.0.0.2" prot="61001" unitnum="1" status="2" rights="110000111001111">
            <UnitNodes index="0" channelnum="48" streamType="81" type="1" subType="0" zeroChnEncode="0">
                <Channel id="10000978001@001$1$0$0" name="GB49_1-" desc="" channelType="1" channelSN="10005"></Channel>
                <Channel id="10000978001@001$1$0$1" name="GB49_1-" desc="" channelType="1" channelSN="10005" ></Channel>
                <Channel id="10000978001@001$1$0$2" name="GB49_1-" desc="" channelType="1" channelSN="10005" ></Channel>
            </UnitNodes>
        </Device>
        <Device id="1000018001" type="65548" name="海康106" manufacturer="1" ip="127.0.0.2" prot="61001" unitnum="2" status="2" rights="110000111001111">
            <UnitNodes index="0" channelnum="48" streamType="81" type="1" subType="0" zeroChnEncode="0">
                <Channel id="10000978001@001$1$0$0" name="GB49_1-" desc="" channelType="1" channelSN="10005"></Channel>
                <Channel id="10000978001@001$1$0$1" name="GB49_1-" desc="" channelType="1" channelSN="10005" ></Channel>
                <Channel id="10000978001@001$1$0$2" name="GB49_1-" desc="" channelType="1" channelSN="10005" ></Channel>
            </UnitNodes>
        </Device>
        <Device id="1000017001" type="65548" name="海康106" manufacturer="1" ip="127.0.0.2" prot="61001" unitnum="3" status="2" rights="110000111001111">
            <UnitNodes index="0" channelnum="48" streamType="81" type="1" subType="0" zeroChnEncode="0">
                <Channel id="10000978001@001$1$0$0" name="GB49_1-" desc="" channelType="1" channelSN="10005"></Channel>
                <Channel id="10000978001@001$1$0$1" name="GB49_1-" desc="" channelType="1" channelSN="10005" ></Channel>
                <Channel id="10000978001@001$1$0$2" name="GB49_1-" desc="" channelType="1" channelSN="10005" ></Channel>
            </UnitNodes>
        </Device>
        <Device id="1000016001" type="65548" name="海康106" manufacturer="1" ip="127.0.0.2" prot="61001" unitnum="4" status="2" rights="110000111001111">
            <UnitNodes index="0" channelnum="48" streamType="81" type="1" subType="0" zeroChnEncode="0">
                <Channel id="10000978001@001$1$0$0" name="GB49_1-" desc="" channelType="1" channelSN="10005"></Channel>
                <Channel id="10000978001@001$1$0$1" name="GB49_1-" desc="" channelType="1" channelSN="10005" ></Channel>
                <Channel id="10000978001@001$1$0$2" name="GB49_1-" desc="" channelType="1" channelSN="10005" ></Channel>
            </UnitNodes>
        </Device>
        <Device id="1000015001" type="65548" name="海康106" manufacturer="2" ip="127.0.0.2" prot="61001" unitnum="5" status="2" rights="110000111001111">
            <UnitNodes index="0" channelnum="48" streamType="81" type="1" subType="0" zeroChnEncode="0">
                <Channel id="10000978001@001$1$0$0" name="GB49_1-" desc="" channelType="1" channelSN="10005"></Channel>
                <Channel id="10000978001@001$1$0$1" name="GB49_1-" desc="" channelType="1" channelSN="10005" ></Channel>
                <Channel id="10000978001@001$1$0$2" name="GB49_1-" desc="" channelType="1" channelSN="10005" ></Channel>
            </UnitNodes>
        </Device>
    </Department>
</Organization>
```

* 压缩`xml`文件的线上地址<a href="http://www.bejson.com/otherformat/xml/" target="_blank">http://www.bejson.com/otherformat/xml/</a>

- 解析方法

> 组装自定义的`JSON`格式数据
> 
```javascript
function parseXMLData(root) {
	console.info('----------------------------解析组织树数据----------------------------');
	var rootTree = [];
	for (let i = 0; i < root.length; i++) {
		// console.info(root[i]);
		let id = root[i]['-coding']; // 组织树编号
		let title = root[i]['-name']; // 组织树名称(对应为根节点名称)
		let device = root[i].Device; //设备树
		let children = [];
		for (let i = 0; i < device.length; i++) {
			let device_id = device[i]['-id'];
			let device_ip = device[i]['-ip'];
			let device_manufacturer = device[i]['-manufacturer'];
			let device_name = device[i]['-name'];
			let device_port = device[i]['-port'];
			let device_type = device[i]['-type'];

			let device_channels = device[i].UnitNodes.Channel;

			let channels = [];
			for (let i = 0; i < device_channels.length; i++) {
				// console.info(device_channels[i]);
				let channel_id = device_channels[i]['-id'];
				let channel_name = device_channels[i]['-name'];
				let channel_sn = device_channels[i]['-channelSN'];
				let channel_type = device_channels[i]['-channelType'];

				channels.push({
					"id": channel_id,
					"title": channel_name,
					"channelSN": channel_sn,
					"channelType": channel_type,
					"isLeaf": true
				});
			}
			children.push({
				"id": device_id,
				"title": device_name,
				"children": channels,
				"ip": device_ip,
				"port": device_port,
				"type": device_type,
				"manufacturer": device_manufacturer
			})

		}
		rootTree.push({
			"id": id,
			"title": title,
			"children": children
		});
	}
	//console.info(rootTree);
	return rootTree;
}
```

