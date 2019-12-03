---
title: Java导出Excel并下载
toc: true
layout: blog
categories:
  - Blog
tags:
  - Blog
date: 2019-03-25 09:46:41
---
Java使用POI生成Excel、下载文件
<!-- more -->

# <h3>Java查询结果生成多 Sheet 的页 Excel,提供页面导出功能</h3>

## 查询结果直接导出 Excel

* 思路: 使用 `import javax.servlet.http.HttpServletResponse` 获取输出流,将文件写入,实现导出

> controller 代码

```java
  @RequestMapping(value = "xx" method = RequestMethod.POST)
  public void batchCommonalityQuery(HttpServletResponse response) {
    try {
      String fileName = file.getOriginalFilename();
      // 文件名称   
      String name = "xx.xlsx";

      response.setContentType("application/octet-stream");
      response.addHeader("Content-Disposition",	"attachment;filename=\"" + new String(name.getBytes("UTF-8"), "ISO-8859-1"));
      
      OutputStream outputStream = response.getOutputStream();
      
      List<List<UserUrlVO>> list = this.commonalityService.batchCommonsQuery(file, outputStream);
    } catch (Exception e) {
      e.printStackTrace();
    }
  }
```

> service 代码

```java
  private void exportExcel(List<List<T>> list, List<String> sheetName, String[] header, OutputStream out) {
    HSSFWorkbook workbook = new HSSFWorkbook();

    try {
      for (int i = 0; i < list.size(); i++) {
        // sheet页名
        HSSFSheet sheet = workbook.createSheet(sheetName.get(i));

        // list内容
        List<T> values = list.get(i);
        HSSFRow row;// = sheet.createRow(values.size());

        // 设置行标题(Excel第一行)
        row = sheet.createRow(0);
        for (int h = 0; h < header.length; h++) {
          row.createCell(h).setCellValue(header[h]);
          sheet.autoSizeColumn(h, true);
          sheet.setColumnWidth(h, sheet.getColumnWidth(h) * 17 / 10);
        }
        // 写入行 —— 单元格
        for (int j = 0; j < values.size(); j++) {
          row = sheet.createRow(j + 1);

          Object object = values.get(j);
          // 遍历实体类属性
          Field[] fields = object.getClass().getDeclaredFields();
          // 单元格内容
          for (int k = 0; k < fields.length; k++) {
            Field field = fields[k];
            field.setAccessible(true);
            row.createCell(k).setCellValue(field.get(object).toString());
          }
        }
      }
      // 响应下载
      workbook.write(out);
      out.close();
    } catch (Exception e) {
      e.printStackTrace();
    } finally {
      try {
        out.flush();
        out.close();
      } catch (IOException e) {
        e.printStackTrace();
      }
    }
  }
```

## 将文件保存到服务器,提供返回下载路径  

> controller 代码

```java
  @RequestMapping(value = "downloadFile")
	public void downloadFile(@RequestParam(value = "filepath", required = true) String filePath,HttpServletResponse response) {
		this.commonalityService.downloadFile(filePath, response);
	}
```

> service 代码

1. 将文件写入本地

```java

final static String EXCEL_PATH = System.getProperty("os.name").toLowerCase().startsWith("win") ? "D:\\fileupload\\excel\\" : "/root/commonality";

  // 将文件写入本地
  private void writeExcel(List<List<T>> list, List<String> sheetName, String[] header, String fileName) {
    HSSFWorkbook workbook = new HSSFWorkbook();

    try {
      for (int i = 0; i < list.size(); i++) {
        // sheet页名
        HSSFSheet sheet = workbook.createSheet(sheetName.get(i));

        // list内容
        List<T> values = list.get(i);
        HSSFRow row;// = sheet.createRow(values.size());

        // 设置行标题(Excel第一行)
        row = sheet.createRow(0);
        for (int h = 0; h < header.length; h++) {
          row.createCell(h).setCellValue(header[h]);
          sheet.autoSizeColumn(h, true);
          sheet.setColumnWidth(h, sheet.getColumnWidth(h) * 17 / 10);
        }
        // 写入行 —— 单元格
        for (int j = 0; j < values.size(); j++) {
          row = sheet.createRow(j + 1);

          Object object = values.get(j);
          // 遍历实体类属性
          Field[] fields = object.getClass().getDeclaredFields();
          // 单元格内容
          for (int k = 0; k < fields.length; k++) {
            Field field = fields[k];
            field.setAccessible(true);
            row.createCell(k).setCellValue(field.get(object).toString());
          }
        }
      }
      // 判断文件夹目录是否存在,不存在则创建
      File file = new File(EXCEL_PATH);
      if (!file.exists()) {
        // 创建的是多级目录,file.mkdir(); 只能创建一级目录
        file.mkdirs();
      }
      // 生成文件保存到服务器
      String path = EXCEL_PATH + fileName;
      FileOutputStream stream = new FileOutputStream(path);
      workbook.write(stream);
      stream.close();
    } catch (Exception e) {
      e.printStackTrace();
    }
  }
```

2. 下载文件

```java
	public void downloadFile(String filePath, HttpServletResponse response) {
		OutputStream outputStream = null;
		FileInputStream fis = null;
		try {
			response.setContentType("application/octet-stream");
			response.setCharacterEncoding("UTF-8");
			response.addHeader("Content-Disposition",
					"attachment;filename=\"" + new String(filePath.getBytes("UTF-8"), "ISO-8859-1"));

			outputStream = response.getOutputStream();
			fis = new FileInputStream(EXCEL_PATH + filePath);
			byte[] bytes = new byte[1024];
			int lengtth = 0;
			while ((lengtth = fis.read(bytes)) != -1) {
				outputStream.write(bytes, 0, lengtth);
			}
			outputStream.flush();
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			try {
				if (outputStream != null) {
					outputStream.close();
				}
				if (fis != null) {
					fis.close();
				}
			} catch (IOException e) {
				e.printStackTrace();
			}

		}
	}
```

> 公共方法  

```java
  // @描述：是否是2003的excel，返回true是2003
  private boolean isExcel2003(String filePath) {
    return filePath.matches("^.+\\.(?i)(xls)$");
  }

  // @描述：是否是2007的excel，返回true是2007
  private boolean isExcel2007(String filePath) {
    return filePath.matches("^.+\\.(?i)(xlsx)$");
  }

  private Object getCellFormatValue(Cell cell) {
    Object cellValue = null;
    if (cell != null) {
      switch (cell.getCellType()) {
      case Cell.CELL_TYPE_NUMERIC:
        cellValue = NumberToTextConverter.toText(cell.getNumericCellValue());
        break;
      case Cell.CELL_TYPE_STRING:
        cellValue = cell.getRichStringCellValue().getString().trim();
        break;
      default:
        cellValue = "";
        break;
      }
    } else {
      cellValue = "";
    }
    return cellValue;
  }
```