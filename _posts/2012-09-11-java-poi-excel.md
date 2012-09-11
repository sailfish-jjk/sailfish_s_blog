---
layout : post
category: problems
title : JAVA POI 操作excel遇到问题
---
Excel在"XXX.xls"中发现不可读取的内容

大概只能输出150行左右的excel,超出部分文本不显示了.

找了半天,把所有格式删了,发现

	HSSFRichTextString richString = new HSSFRichTextString(textValue);
	HSSFFont font3 = workbook.createFont();
	font3.setBoldweight(HSSFFont.BOLDWEIGHT_NORMAL);
	richString.applyFont(font3);
	cell.setCellValue(richString);

这段话删了就好使了....

要用回最原始的:

	cell.setCellValue(textValue);

这个方法还是个过时的方法,被那个richText替代了.....

以为过时了就不能用了,原来不是这样,汗.....

最近两个项目里用到了javamail和poi, 等项目完了要重构一下代码, 整理成util方便以后使用.