---
title: Ext 4.0 使用笔记
toc: true
layout: blog
categories:
  - blog
tags:
  - Blog
  - Web Plugin
date: 2019-02-27 15:20:13
---
{{title}}
<!-- more -->
# Ext 4.0 使用笔记	

## 获取Grid 当前选中行数据	

> 获取行数据 		

```javascript
	var record = Ext.getCmp('now_alarm').getSelectionModel().getSelection();
	//获取选中单行的某列值 
	var deviceName = record[0].data.DEVICE_NAME;

```

## 列宽自动适应

```javascript
	forceFit:true,//注意不要用autoFill:true;那样设置当GridPanel的大小变化(比如resize了它)时不会自动调整column的宽度
	scrollOffset:0//不加这个的话,会在grid的最右边有个空白,留作滚动条的位置

```

## load store 
```javascript
	//查询参数
  var getParams = function() {
		return {
			sheetIds : Ext.getCmp('sheetId').getValue(),
			station : Ext.getCmp('station').getValue(),
			maintain : Ext.getCmp('maintainer').getValue(),
			startTime : Ext.util.Format.date(Ext.getCmp('startTime').getValue(),'Y-m-d'),
			//status : state,// Ext.getCmp('status').getValue(),
			endTime : Ext.util.Format.date(Ext.getCmp('endTime').getValue(),'Y-m-d'),
			worksheettype : Ext.getCmp('worksheetType').getValue()
		}
	} 
	// 条件查询
	function reportLoad(){
		var params = getParams();
//		console.log(params);
		for ( var key in params) {
			if (params[key] == null || params[key] == "") {
				delete params[key];
				continue;
			}
		}
		reportStore.setProxy({
			type : 'ajax',
			url: '../centerSheet/getWorkSheet',
			extraParams : params,
			reader:{
				type:'json',
				root:'records',
				totalProperty:'totalCount'
			}
		});
		switch (state) {
		case 0:
			Ext.getCmp('no_confirm').getStore().load();
			break;
		case 1:
			Ext.getCmp('no_repair').getStore().load();
			break;
		case 2:
			Ext.getCmp('no_statements').getStore().load();
			break;
		case 4:
			Ext.getCmp('statements').getStore().load();
			break;
		default:
			break;
		}
	}
```

## 创建window	

```javascript
 Ext.create('Ext.window.Window', {
		title : '问题单信息录入',
		labelWidth : 75, // label settings here cascade unless overridden
		url : 'save-form.php',
		frame : true,
		bodyStyle : 'padding:5px 5px 0',
		width : document.body.clientWidth * 0.58,
		renderTo : Ext.getBody(),
		layout : 'column', // arrange fieldsets side by side
		xtype : 'form',
		defaults : {
			bodyPadding : 4
		},
		items : [ {
			xtype : 'fieldset',
			columnWidth : 1,
			title : '告警基本信息',
			collapsible : true,
			defaultType : 'textfield',
			defaults : {
				anchor : '100%'
			},
			layout : 'anchor',
			items : [ {
				fieldLabel : 'Field 1',
				name : 'field1'
			}, {
				fieldLabel : 'Field 2',
				name : 'field2'
			} ]
		} ]
	}).show();
```

## 获取选中行`displayField`的值	

```javascript
Ext.getCmp('distributeUserName').getRawValue(),
```

## 获取点击行数据 	

` Ext.getCmp('now_alarm').getStore().getAt(index).data`	

## Store load

> 第一种		

```javascript
	historyStore.setProxy({
		type : 'ajax',
		url : newUrl,
		reader:{
			type:"json",
			root:'root',
			totalProperty:'totalCount'
		}
		});
	Ext.getCmp('history_alarm').getStore().load();
```
> 第二种		

```javascript
// now_alarm : GridPanel
var gridStore =  Ext.getCmp('now_alarm').getStore();
gridStore.reload(gridStore.lastOptions);

```

## `store`加载传参		

> `proxy` 设置 `extraParams`

```javascript
	xtype : 'combo',
	fieldLabel : '维保人员',
	id : 'sendOperUser',
	name : 'sendOperUser',
	style:{
		margin : '10px 10px 10px 50px'
	},
	resizable : false,
	store : Ext.create('Ext.data.Store',{
    	autoLoad : true,
    	fields : [ {
			name : 'id',
			mapping :'ID',
		}, {
			name : 'name',
			mapping :'NAME',
		}],
    	proxy : {
    		type : "ajax",
    		url : '../alarm/getMaintainer',
    		extraParams : {
    			error_sheet : errorSheetId,
    			company : company
			},
    		reader : {
    			type : "json"
    		}
    	},
    	listeners: {
			load: function(store, records, successful, eOpts){
				for(k in records){
					if (records[k].data) {
						// console.log(records[k].data);
						if (distribute_account == records[k].data.name) {
							Ext.getCmp('sendOperUser').setValue(records[k].data.id);
						}
					}
				}
			}
		}
    }),
    valueField : 'id',
	displayField : 'name',
	mode : 'local',
	triggerAction : 'all',
	allowBlank : false,
	blankText : '必选项',
	maxHeight : 150
```

> 拼接URL	

```javascript
proxy : {
	type : "ajax",
	url : '../alarm/getMaintainer?error_sheet='+errorSheetId+'&company='+company,
	reader : {
		type : "json"
	}
},
```
## 监听 `tabpanel` 事件 	

```javascript
 	listeners:{
    	tabchange :function(tabPanel,tab){
    		
    	}
    },
```

## 解析`store`返回自定义参数值	

```javascript
// 查询条件 封装
var getParams = function() {
		return {
			sheetIds : Ext.getCmp('sheetId').getValue(),
			station : Ext.getCmp('station').getValue(),
			maintain : Ext.getCmp('maintainer').getValue(),
			startTime : Ext.util.Format.date(Ext.getCmp('startTime').getValue(),'Y-m-d'),
			status : state,// Ext.getCmp('status').getValue(),
			endTime : Ext.util.Format.date(Ext.getCmp('endTime').getValue(),'Y-m-d'),
			worksheettype : Ext.getCmp('worksheetType').getValue()
		}
	} 
	// 条件查询
	function reportLoad(){
		// 移除为空的查询条件
		var params = getParams();
		for ( var key in params) {
			if (params[key] == null || params[key] == "") {
				delete params[key];
				continue;
			}
		}
		// 添加自定义返回参数`no_coinfirm`
		reportStore.setProxy({
			type : 'ajax',
			url: '../centerSheet/getWorkSheet',
			extraParams : params,
			reader:{
				type:'json',
				root:'records',
				totalProperty:'totalCount',
				no_coinfirm:'no_confirm',
				no_repair:'no_repair',
				no_statements:'no_statements',
				statements:'statements'
			}
		});
		// 加载对应的 tabpanel 的数据、更新titel
		reportStore.load({
			scope: this,
			callback: function(records, operation, success) {
				var reader = reportStore.getProxy().getReader(); 
				Ext.getCmp('no_confirm').setTitle(tabpanel1 + '&nbsp;&nbsp;'+reader.jsonData.no_confirm);
				Ext.getCmp('no_repair').setTitle(tabpanel2 + '&nbsp;&nbsp;'+reader.jsonData.no_repair);
				Ext.getCmp('no_statements').setTitle(tabpanel3 + '&nbsp;&nbsp;'+reader.jsonData.no_statements);
				Ext.getCmp('statements').setTitle(tabpanel3 + '&nbsp;&nbsp;'+reader.jsonData.statements);
			}
		});
	}
```

## `fieldset` 可折叠控制	 	

	`collapsible : true`

## 监听表格数据双击事件	

```javascript
listeners: {
    // 监听双击事件
    itemdblclick: function (dataView, record, item, index, e) {
      // 当前数据行数据
      var data = record.data;
      console.log(data);
      //传参
      var user_id = data.id;
      var process_time = Ext.util.Format.date(data.process_time, 'YmdHi');

      queryQosLog(user_id, process_time);
    }
  }
```

## `Ext window`遮罩效果实现	

```javascript
在Ext.window.Window 中添加属性
	modal: true,
```

## 设置隔行变色、同时指定行背景色

```js
/* CSS */
// 隔行背景色
.x-grid-record-ou td {
	background-color: #cacaca;
}

/* 设置背景色 */
.x-grid-record-fenghuo td{
	background-color: #ff3333;
}

/* js */
forceFit: true,//列宽自适应
viewConfig:{
	stripeRows:false,//开启隔行变色,默认false但是遇到的是没用。所以设置了false值
	getRowClass: function (record, rowIndex, rowParams, stroe) {
		if (record.get('type').indexOf('错误') != -1) {
		return 'x-grid-record-fenghuo td';// 指定的背景色class
		} else {
		if (rowIndex % 2 == 0) {
			return 'x-grid-record-ou'; // 隔行变色的class
		} else {
			return '';
		}
		}
	}
}
```

## 表单重置

```sql
// 需要获取的 region_form 的 xtype: 'form';formpanel 设置则无效
Ext.getCmp('region_form').getForm().reset();
```