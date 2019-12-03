---
title: Java使用EasyExcel上传文件
toc: true
layout: blog
categories:
  - Blog 
  - Java
tags:
  - Java
  - EasyExcel
date: 2019-02-27 15:20:13
---

{{title}}

<!-- more -->

# `Java`使用`EasyExcel` 上传文件

> 添加依赖	

```pom
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>easyexcel</artifactId>
    <version>1.1.2-beta5</version>
</dependency>
```

> 部分代码	

导入信息实体类(上传文件对应实体类)

```java
/**
 * 
 * @ClassName: Advertising
 * @Description: 广告信息批量导入Model
 * @date 2019/01/29
 */
public class AdvertisingImportModel extends BaseRowModel {

	@ExcelProperty(index = 0)
	private String ad;

	@ExcelProperty(index = 1)
	private String address;

	public String getAd() {
		return ad;
	}

	public void setAd(String ad) {
		this.ad = ad.trim();
	}

	public String getAddress() {
		return address;
	}

	public void setAddress(String address) {
		this.address = address.trim();
	}

	@Override
	public String toString() {
		return "{\"ad\":\"" + ad + "\",\"address\":\"" + address + "\"}";
	}

}
```

导出文件上传实体类

```java
public class AdvertisingExportModel extends BaseRowModel {

	@ExcelProperty(value = "用户AD", index = 0)
	private String name;

	@ExcelProperty(value = "用户地址", index = 1)
	private String address;

	public AdvertisingExportModel() {
	}

	public AdvertisingExportModel(String name, String address) {
		this.name = name;
		this.address = address;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getAddress() {
		return address;
	}

	public void setAddress(String address) {
		this.address = address;
	}

	@Override
	public String toString() {
		return "{\"name\":\"" + name + "\",\"address\":\"" + address + "\"}";
	}

}
```

需要自己创建 `ExcelListener` 

```java
public class ExcelListener extends AnalysisEventListener<Object> {

	private List<Object> data = new ArrayList<>();

	@Override
	public void invoke(Object object, AnalysisContext context) {
		data.add(object);
	}

	@Override
	public void doAfterAllAnalysed(AnalysisContext context) {

	}

	public List<Object> getData() {
		return data;
	}

	public void setData(List<Object> data) {
		this.data = data;
	}

}
```

`API`实现方法,简单起见将代码都写在了controller(可以将具体的实现写在service层中)	

```java
	/**
	 * 
	 * @Title: synchronizationInformation
	 * @Description: 上传文件并返回自己生成的数据文件
	 * @param file	需要同步的用户文件
	 * @param response	下载的文件
	 * void
	 */
	@RequestMapping(value = "easyexcel", consumes = "multipart/*", headers = "content-type=multipart/form-data", method = RequestMethod.POST)
	public void synchronizationInformation(@RequestParam(name = "file", required = false) MultipartFile file,
			HttpServletResponse response) {
		try {
			List<AdvertisingImportModel> result;
			// 1. 将excel 转换为 List<Object> list
			InputStream inputStream = new BufferedInputStream(file.getInputStream());
			// 文件名
			String fileName = file.getOriginalFilename();
			// 文件后缀
			String prefix = fileName.substring(fileName.lastIndexOf(".") + 1);

			ExcelListener excelListener = new ExcelListener();
			EasyExcelFactory.readBySax(inputStream, new Sheet(1, 1, AdvertisingImportModel.class), excelListener);
			// 文件内容
			List<Object> data = excelListener.getData();
			result = JSONObject.parseArray(JSON.toJSONString(data), AdvertisingImportModel.class);
			logger.info("上传文件包含内容:\t" + result.toString());

			// 原本是要调用http接口,作为list的返回值(现暂时模拟返回值为这些)
			// 2.导出栅格数据文件内容
			List<AdvertisingExportModel> list = new ArrayList<>();
			list = JSONObject.parseArray(
					"[{\"name\":\"亚索\",\"address\":\"艾欧尼亚\"},{\"name\":\"诺克\",\"address\":\"诺克萨斯\"},{\"name\":\"瑞文\",\"address\":\"艾欧尼亚\"}]",
					AdvertisingExportModel.class);

			ExcelTypeEnum excelTypeEnum = prefix == ".xls" ? ExcelTypeEnum.XLS : ExcelTypeEnum.XLSX;

			ServletOutputStream outputStream = response.getOutputStream();
			ExcelWriter writer = new ExcelWriter(outputStream, excelTypeEnum, true);

			Sheet sheet = new Sheet(1, 0, AdvertisingExportModel.class);
			sheet.setAutoWidth(true);
			sheet.setSheetName("第一个sheet");

			fileName = new String(new SimpleDateFormat("yyyy-MM-dd").format(new Date()).getBytes(), "utf-8");
			// System.out.println(fileName + prefix);
			response.setContentType("application/octet-stream");
			response.setCharacterEncoding("utf-8");
			response.setHeader("Content-Disposition", "attachment;filename=" + fileName + "." + prefix);

			writer.write(list, sheet);
			writer.finish();

			outputStream.flush();
			inputStream.close();
			outputStream.close();
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
```

**注：**

1. `InputStream` 使用`file.getInputStream()`时,会报异常。
2. 使用`easyexcel`工具类时,需要将项目中原有的`poi、poi-ooxml、poi-ooxml-schemas` 版本需与`easyexcel`版本一致。

---

> 使用EasyExcel时,遇到的问题 	

>> 1. ` org.apache.catalina.core.StandardWrapperValve`

* 控制台输出 

![StandardWrapperValve][inputStream error]

```java
二月 21, 2019 1:59:51 下午 org.apache.catalina.core.StandardWrapperValve invoke
严重: Servlet.service() for servlet [springmvc] in context with path [/iptvView] threw exception [Request processing failed; nested exception is com.alibaba.excel.exception.ExcelAnalysisException: File type error，io must be available markSupported,you can do like this <code> new BufferedInputStream(new FileInputStream(\"/xxxx\"))</code> "] with root cause
com.alibaba.excel.exception.ExcelAnalysisException: Xls must be available markSupported,you can do like this <code> new BufferedInputStream(new FileInputStream("/xxxx"))</code> 
```

* 问题原因

```java
	InputStream inputStream = file.getInputStream();//获取的时文件inputStream,获取xls时失败.抛出异常

	InputStream inputStream = new BufferedInputStream(file.getInputStream());//正确的获取方式
```

* 解决方式 

```
	改为第二种获取InputStream 的方式
```

>> 2. `java.lang.ClassNotFoundException: org.apache.poi.poifs.filesystem.FileMagic`	

* 问题原因

	项目中引用了`poi、poi-oomxl、poi-oomxl-schemas jar`包，并且版本与 `easyexcel` 版本不一致

* 解决方案	

	更新/降低 项目中引入的`poi`版本,使	版本保持一致

[inputStream error]: /images/easy-excel-poi.png