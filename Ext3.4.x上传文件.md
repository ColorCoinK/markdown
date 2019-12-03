---
title: Ext3.4.x上传文件
toc: true
layout: blog
categories:
  - Blog
  - Ext
tags:
  - Blog
  - Ext 3.4.x
date: 2019-03-07 15:51:39
---
使用`Ext 3.4.x`版本实现上传`.txt`文件(文件内为id值),删除`table_a`对应记录
<!-- more -->

要求: 

  上传一个txt文件,根据文件内的id删除对应的记录。

需求分析: 

  1. 使用 `Ext 3.4.0` 上传文件;
  2. `Java` 解析前端上传的文件,得到待删除 `List<Long>`;
  3. 使用 `jdbctemplate` 批量删除接口,完成删除操作。

# js 页面(表单代码) 

## 页面出现连个按钮

```css
<style type=text/css>
.upload-icon {
	background: url('../images/image_add.png') no-repeat 0 0 !important;
}

.x-form-file-wrap {
	position: relative;
	height: 22px;
}

.x-form-file-wrap .x-form-file {
	position: absolute;
	right: 0;
	-moz-opacity: 0;
	filter: alpha(opacity : 0);
	opacity: 0;
	z-index: 2;
	height: 22px;
}

.x-form-file-wrap .x-form-file-btn {
	position: absolute;
	right: 0;
	z-index: 1;
}

.x-form-file-wrap .x-form-file-text {
	position: absolute;
	left: 0;
	z-index: 3;
	color: #777;
}
</style>
```

## 文件类型不支持,需要引用`fileuploadfield`组件

> `css` 文件内容

```css
.upload-icon {
  // 文件路径需要根据自己需要调整
  // 这张为 ext 图片上传png
   /* https://docs.sencha.com/extjs/4.2.4/extjs-build/examples/shared/icons/fam/image_add.png */
	background: url('../images/image_add.png') no-repeat 0 0 !important;
}

.x-form-file-wrap {
	position: relative;
	height: 22px;
}

.x-form-file-wrap .x-form-file {
	position: absolute;
	right: 0;
	-moz-opacity: 0;
	filter: alpha(opacity : 0);
	opacity: 0;
	z-index: 2;
	height: 22px;
}

.x-form-file-wrap .x-form-file-btn {
	position: absolute;
	right: 0;
	z-index: 1;
}

.x-form-file-wrap .x-form-file-text {
	position: absolute;
	left: 0;
	z-index: 3;
	color: #777;
}
```

> `js` 文件内容 

  以下 `javascript` 代码的样式可以自己修改,主要内容是 `xtype : 'fileuploadfield'、xtype:'button'`。同时,提交方式也可以自行替换(Ajax等)

```js
Ext.onReady(function () {

  var fileForm = new Ext.FormPanel({
    title: '删除H码文件',
    frame: true,
    fileUpload: true, // required parameter
    collapsible: true,
    region: 'north',
    labelWidth: 40,
    height: 100,
    width: '100%',
    margins: '0 120',
    items: [{
        layout: 'form',
        border: false,
        items: [{
            items: {
                labelWidth: 100,
                xtype: 'fieldset',
                labelAlign: 'right',
                autoHeight: true,
                items: [{
                    xtype: 'panel',
                    layout: 'column',
                    border: false,
                    items: [{
                        layout: 'column',
                        border: false,
                        items: [{
                            layout: 'form',
                            items: [{
                                xtype: 'fileuploadfield', // 需要引用 fileuploadfield.js文件
                                id: 'form-file',
                                width: 200,
                                emptyText: '请选择一个文件',
                                fieldLabel: '删除H码文件',
                                buttonText: '',
                                name: 'hcodeFile',
                                buttonCfg: {
                                    iconCls: 'upload-icon'
                                }
                            }]
                        }, {
                            layout: 'form',
                            items: [{
                                layout: 'form',
                                xtype: 'button',
                                text: '上传',
                                width: 50,
                                style:{
                                    marginLeft: '20px'
                                },
                                handler: function () {
                                    if (fileForm.getForm().isValid()) {
                                        var filename = Ext.getCmp('form-file').getValue();
                                        if (filename.indexOf('txt') > -1) {
                                        } else {
                                            msg('错误', '上传的文件不是txt文件类型,请重新选择!');
                                            return;
                                        }
                                        fileForm.getForm().submit({
                                            url: '../delete',// 请求 API 地址
                                            waitMsg: '正在上传...',
                                            waitTitle: '请等待',
                                            success: function (form, action) {
                                                msg('提示', action.result.msg);
                                                fileForm.getForm().reset();
                                                // 刷新表格数据
                                                // gridReload();
                                            },
                                            failure: function (form, action) {
                                                msg('错误', action.result.msg);
                                                fileForm.getForm().reset();
                                                // 刷新表格数据
                                                // gridReload();
                                            }
                                        });
                                    }
                                }
                            }]
                        }]
                    }]
                }]
            }
        }]
    }]
  });

  // 提示框内容
  var msg = function (title, msg) {
        Ext.Msg.show({
            title: title,
            msg: msg,
            minWidth: 200,
            modal: true,
            icon: Ext.Msg.INFO,
            buttons: Ext.Msg.OK
        });
    };
});
```

## 文件上传 `Java` 后端代码

### Controller —— 获取传递的参数,获取文件

```java
import org.springframework.web.multipart.MultipartFile;

@RequestMapping(value = "delet", method = RequestMethod.POST)
public void deleteHcode(@RequestParam("hcodeFile") MultipartFile hcodeFile, HttpServletResponse response)
    throws IOException {
  // 设置 response.setCharsetEncoding("utf-8") 时在当前代码中依然无效,所以改为设置 ContentType
  response.setContentType("text/html;charset=utf-8");
  
  // Service 中的方法名
  Map<String, Object> result = this.hcodeService.removeHcodeList(hcodeFile);

  // 返回 JSON 时,前端显示异常.需要改为 PrintWiter
  PrintWriter out = response.getWriter();
  out.write("{success:" + result.get("success") + ",msg:'" + result.get("msg") + "'}");

  out.flush();
  out.close();
}
```

### Service —— 将 `file` 转为 `list`

```java
import org.springframework.web.multipart.MultipartFile;

public Map<String, Object> removeHcodeList(MultipartFile file) {
  Map<String, Object> result = new HashMap<String, Object>();
  result.put("success", false);
  try {

    // 获取上传 H码文件中的 hcode list
    List<Long> hcodeList = this.readHcodeFile(file);

    hcodeDao.removeHcodeList(hcodeList);

    result.put("success", true);
    result.put("msg", "删除成功");
  } catch (IllegalArgumentException fileException) {
    result.put("msg", "删除失败,文件内容错误,请检查文件的准确性...");
  } catch (IOException e) {
    result.put("msg", "删除失败,系统IO异常,请稍后重试");
  } catch (Exception e) {
    result.put("msg", "删除失败,系统异常,请稍后重试");
    e.printStackTrace();
  }
  return result;
}

/**
  * 
  * @Title: readHcodeFile
  * @Description: 读取文件的方式可以根据文件内容进行调整
  * @param file(*.txt 文件)
  * @return
  * Set<Long>
  * @throws IOException 
  */
private List<Long> readHcodeFile(MultipartFile file) throws IOException {
  InputStreamReader reader = null;
  BufferedReader br = null;
  try {
    reader = new InputStreamReader(file.getInputStream(), "UTF-8");
    br = new BufferedReader(reader);

    Set<Long> hcodeList = new HashSet<Long>();
    String line = "";
    while ((line = br.readLine()) != null && line.length() > 1) {
      line = line.trim();
				if (line == null) {
					continue;
				}
				hcodeList.add(Long.parseLong(line));
        hcodeList.add(Long.parseLong(line));
    }
    return new ArrayList<Long>(hcodeList);
  } catch (Exception e) {
    throw new IllegalArgumentException("文件内容错误");
  } finally {
    reader.close();
    br.close();
  }
}
```

**注：**
  当前方法读取的文件如下图所示
  ![上传带解析的文件][ext3.4.0-fileupload]

[ext3.4.0-fileupload]: ../../../images/ext-upload-file.png

### Repository 批量删除的方法 —— `jdbctemplae` 方式

```java
import org.springframework.jdbc.core.BatchPreparedStatementSetter;

public void removeHcodeList(final List<Long> hcodeList) {
  String sql = "delete from table_a t where t.hcode = ?";
  // batchUpdate 为批量操作方法
  this.getJdbcTemplate().batchUpdate(sql, new BatchPreparedStatementSetter() {

    @Override
    public void setValues(PreparedStatement preparedstatement, int i) throws SQLException {
      Long hcode = hcodeList.get(i);
      preparedstatement.setLong(1, hcode);
    }

    @Override
    public int getBatchSize() {
      return hcodeList.size();
    }
  });
}
```

## 常见异常 

> 不支持文件上传

```java
org.springframework.web.bind.MissingServletRequestParameterException: Required MultipartFile parameter 'hcodeFile' is not present
```

* 方法: 在 `applicationContext.xml` 文件中实例化 `bean`,添加如下内容

```xml
<!-- 支持文件上传 -->
<bean id="multipartResolver"
  class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
</bean>
```