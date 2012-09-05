---
layout : post
category: problems
title : SQL UPDATE语句执行顺序问题
---
昨天开发定时任务的时候遇到的问题。

描述：定时扫描售价表信息变化，并更新日表中的单价信息以及总价。

	UPDATE DT_xms_salebill_daily 
	SET 
		saleprice = (case when mediatype = 2 then ? else ? end ) , 
		saleamount = (salprice * feenum)  
	WHERE companyname = ?
	
SQL这样写会出现saleprice列更新，但是saleamount没有更新。再执行一次saleamount才更新。于是考虑是否是 update 执行顺序问题，google之，得到如下结论:

1. 先变量再字段 
 
2. 变量之间, 从左到右  

3. 字段之间, 并行执行

套用到我遇到的问题就是，saleprice和saleamount同步更新，此时saleamount中乘数(saleprice)取值是未更新前的saleprice。因此，想要达到预想效果需要把SQL拆分:

	UPDATE DT_xms_salebill_daily 
	SET 
		saleprice = (case when mediatype = 2 then ? else ? end ) 
	WHERE companyname = ?
	
	UPDATE DT_xms_salebill_daily 
	SET saleamount = (salprice * feenum)
	
	.....
	
以上。
