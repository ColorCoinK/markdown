---
title: Ext4.2.4上传文件
toc: true
layout: blog
categories:
  - Blog
  - Ext
tags:
  - Blog
  - Ext 4.2.4
date: 2019-03-07 16:51:05
---
使用`Ext 4.2.4`版本导入`.xls、xlsx`文件内容到数据库 
<!-- more -->

# 页面内容

```html

```

## js 内容

```js
Ext.onReady(function () {
  Ext.create('Ext.window.Window', {
      id: "importInfo",
      title: "导入广告平台用户",
      autoShow: true,
      modal: true,
      constrainHeader: true,
      resizable: false,
      height: 250,
      width: 550,
      layout: "fit",
      items: [{
          xtype: 'form',
          width: 400,
          bodyPadding: 10,
          items: [{
              xtype: 'filefield',
              name: 'file',
              fieldLabel: '文件上传',
              labelWidth: 80,
              msgTarget: 'side',
              allowBlank: false,
              anchor: '100%',
              buttonText: '选择文件'
          }],
          buttons: [{
              text: '上传',
              handler: function () {
                  var form = this.up('form').getForm();
                  if (form.isValid()) {
                        form.submit({
                            url: '../user/importADInformation.do',
                            method: 'POST',
                            waitMsg: '文件正在上传',
                            success: function (form, action) {
                            Ext.Msg.alert('Success', '广告用户信息导入成功');
                              Ext.getCmp("importInfo").close();
                            },
                            failure: function(form, action){
                            var result = Ext.JSON.decode(action.response.responseText)
                            Ext.Msg.alert('Failed', result.msg);
                              Ext.getCmp("importInfo").close();
                            }
                        });
                  }
              }
          }]
      }]
  });
});
```

# Java 后端代码 

```tree
--top
  |
  --Controller
      |
      ---AdvertisingController
  |
  --Repository
        |
        ---AdvertisingUserDao
  |
  --Entity
      |
      ---AdvertisingEntity
      ---AdvertisingModel
  |
  --listeners
      |
      ---ImportAdvertisingUserExcelListener
```
## Controller

```java
@ResponseBody
@RequestMapping(value = "importADInformation", consumes = "multipart/*", headers = "content-type=multipart/form-data", method = RequestMethod.POST)
@MethodLog(remark = "批量导入用户数据")
public Object synchronizationInformation(@RequestParam(value = "file", required = false) MultipartFile file) {
  Map<String, Object> result = new HashMap<>();
  result.put("success", false);

  // 1. 获取Excel 内容,将Excel转换为List
  InputStream inputStream = null;
  try {
    inputStream = new BufferedInputStream(file.getInputStream());
    // 引用 easyexcel jar 代码地址：https://github.com/alibaba/easyexcel/
    // 使用 EasyExcel 读取 excel 文件内容,将 excel 转为 List<Object>
    ImportAdvertisingUserExcelListener excelListener = new ImportAdvertisingUserExcelListener();
    // 读excel数据,返回List<Object>.将 地址信息转为 50m 栅格编码,并将结果返回
    EasyExcelFactory.readBySax(inputStream, new Sheet(1, 1, AdvertisingModel.class), excelListener);
    List<Object> data = excelListener.getData();

    // 2. 获取入库的List
    List<AdvertisingEntity> list = JSONArray.parseArray(JSONObject.toJSONString(data), AdvertisingEntity.class);
    System.out.println(JSONObject.toJSONString(list));

    this.advertisingUserDao.batchSave(list);

    result.put("success", true);
    result.put("msg", "数据导入成功");
  } catch (IllegalArgumentException GDException) {
    result.put("msg", "文件格式错误");
  } catch (IOException e) {
    result.put("msg", "IO异常");
  } finally {
    try {
      inputStream.close();
    } catch (IOException e) {
      result.put("msg", "IO异常");
    } catch (Exception e) {
      result.put("msg", "文件类型错误(应为*.xls或*.xlsx)");
      e.printStackTrace();
    }
  }
  return result;
}
```

## Repository 

```java
public void batchSave(List<AdvertisingEntity> list) {
  Session session = null;
  try {
    session = this.getSession();
    Transaction transaction = session.beginTransaction();
    for (int i = 0; i < list.size(); i++) {
      AdvertisingEntity adverst = list.get(i);
      session.saveOrUpdate(adverst);
      if (i % 100 == 0) {
        session.flush();
        session.clear();
      }
    }
    transaction.commit();
  } catch (Exception e) {
    throw e;
  } finally {
    session.close();
  }
}
```