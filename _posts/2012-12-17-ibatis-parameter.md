---
layout: post
title: ibatis中传参$与#区别以及对List的排序
category: problems
---
开发中ibatis会用到： $ 和 # 符号。
 
一、区别
-----------

* `$aaa$` 输出参数是以字符串方式直接输出 123

* `#aaa#` 输出参数是以Parameter方式输出 @aaa

二、实例
-----------

1. SQL :

		<select id="selectMetaMandao"  parameterClass="java.util.HashMap" resultMap="WaasmetaMandaoMap" >  
			SELECT $subCategory$ rchannel,sum(mandao) mandao from
			( SELECT nvl(channel,'-') channel,ROUND(count(1)*0.1131) mandao  FROM WAAS_VIEW_USER_RESULT a 
			AND not EXISTS (
			SELECT msgid
			  FROM DT_XMS_MT b
			 WHERE EXISTS
				   (SELECT MSGID FROM WAAS_VIEW_USER_RESULT WHERE STATUS = 'SUCCESS' AND msgid = b.msgid)
			AND msgid = a.msgid)
			   GROUP BY channel) d LEFT JOIN waas_channel_summary c ON d.channel = c.original_channel 
			<isNotEmpty property="channel">
					WHERE c.original_channel LIKE '%'||#channel#||'%' 
			 </isNotEmpty>
			   GROUP BY $subCategory$
		</select>


2. 代码 : 

	@RequestMapping("queryMeta.do")
	public void selectMeta(HttpServletRequest request, HttpServletResponse response) {
		String channel = request.getParameter("channel");
		Map<String, Object> param = new HashMap<String, Object>();
		//以参数方式输出, 相当于一个变量 (活的)
		param.put("channel", channel); 
		//以字符串方式输出, 直接输出(死的)
		param.put("subCategory", channel == null ? "primary_category" : "ORIGINAL_CHANNEL"); 
		List<WAASUserMOTypeResult> data = wumtService.selectMeta(param);
		//因为楼上这个List查完之后重新封装的, 所以无序, 要对List里面的对象进行排序, 如下
		Collections.sort(data, new Comparator<WAASUserMOTypeResult>() {
			public int compare(WAASUserMOTypeResult o1, WAASUserMOTypeResult o2) {
				return o2.getShijia() - o1.getShijia(); //排序字段, 倒序
			}
		});
		
		......
		
		renderJson(response, resultJson.toString());

	}


3. 实现效果:

如果传channel参数, 则根据原始媒体(ORIGINAL_CHANNEL)分组, 不传则根据合并后的大媒体(primary_category) 分组.


{% include references.md %}
