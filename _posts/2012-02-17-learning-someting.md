---
layout: post
title: 记在各种乱七八糟的事儿之中
category: problems
---
最近真的挺忙, 没时间好好整理成文了. 大牛随手指点两三招, 比自己闷头写代码学到的真心多啊. 多数是一些代码习惯, 可能说的时候很习惯就那样做了, 过段时间又忘了. 记下来吧, 也是成长的过程呢.

以下整理自近期和大牛的聊天记录... 

* 数据库表结构和查询的时候显示的内容不一样时,应该怎么建立or管理实体类?

	*增删改应该用一个跟数据表对应的实体类,查询的时候可以建立一个尽量字段全的实体类,(查询的字段可以比类中字段少,不能多)需要新增字段直接往查询类的里面加就行了...
下回要分开管理>.<*

* 更新或者插入操作需要数据唯一性验证时,应该在哪一层进行验证? 数据库有联合主键约束,前端有约束. 后台呢?

	*为了保证良好的用户体验,应该避免走到数据库哪一层,放controller里检验就行了,一些基本的非空验证,唯一约束...
service层处理业务逻辑,不做数据检验*

* 还是定时任务处理, 保证数据获取最小部分核心没问题(不会因为重复运行而出现重复数据(存在则更新,不存在则插入)),然后往外包业务逻辑.

	*一定要考虑如果任务停了怎么补数据...可以把时间作为参数传进去... 然后所有业务逻辑都是围绕一个时间参数,比较方便修改...
	时间可以用一天也可以用一个时间段,根据业务逻辑定*

* 刚学了一招转换的超级好使啊啊啊!!!开心ing...遇到数据库存的字段和实体类里面不一样,需要进行转换的时候.可以再实体类里进行get\set的转换.

*例如:这个DtXmsMtN实体类里,status是数据库存的字段,其中Y表示成功,其他表示失败...我想把它翻译成中文,有两个地方可以修改,一个是实体类初始化setStatus()时, 对这个自定义的statusStr进行翻译. 另一种是在getStatusStr时, 对status进行判断并翻译. 如下...*

	public class DtXmsMtN {

		@Column(value = "STATUS")
		private String status;

		private String statusStr;

		public String getStatus() {
			return status;
		}

		public void setStatus(String status) {
			this.status = status;
		}
		
		public String getStatusStr() {
			if (status.equals("Y"))
				statusStr = "成功";
			else
				statusStr="失败";
			return statusStr;
		}
	}



*另外, 大牛教育说, 不要为了一时的小省事儿而去丢掉数据库中的原始信息. 除非你确定以后再也不会用它. 否则不要直接去修改实体类中status的值, 而应该是建一个显示值来显示它~ so... 学习了~*

* 写帮助类的时候应该考虑的更完善一些,满足大部分需求... 不可以修改帮助类...(既然打算把它做成Util了, 就要做成通用的~ 动不动就修改, 叫啥通用... )  具体可以参考一下修改后的ExcelUtil~ 

* 写sql的时候只有要连表才用join,否则小表用in,大表用exsits~

		-- EXISTS强调的是是否返回结果集 (一个布尔值) 只要EXISTS引导的子句有结果集返回, 那么EXISTS这个条件就算成立了. 
		-- 所以EXISTS子句不在乎返回什么, 而是在乎是不是有结果集返回. 

		SELECT channel,round(count(1)* 0.1131) mdfail  FROM WAAS_VIEW_USER_RESULT a 
		WHERE is_old = 1 AND not EXISTS (
		SELECT msgid
		  FROM DT_XMS_MT b
		 WHERE EXISTS
			   (SELECT MSGID FROM WAAS_VIEW_USER_RESULT WHERE STATUS = 'SUCCESS' AND msgid = b.msgid)
		AND msgid = a.msgid)
		   GROUP BY channel;

		-- PS: 对比一下in, in 引导的子句只能返回一个字段, 比如楼上那句用in来写就是这样 :

		SELECT channel,round(count(1)* 0.1131) mdfail  FROM WAAS_VIEW_USER_RESULT a 
		WHERE is_old = 1 AND a.msgid NOT IN (
		SELECT msgid
		  FROM DT_XMS_MT b
		 WHERE b.msgid IN 
			   (SELECT MSGID FROM WAAS_VIEW_USER_RESULT WHERE STATUS = 'SUCCESS')
		)
		GROUP BY channel;
		
		-- 但是效率真心很低
		
* Map<String,Object>在连表的时候真的很好用... 
	
	遇到过这样的问题, 一个页面中需要同时显示status为成功以及不成功的数量.
	
	解决方法一: SQL中查出成功的结果集, 然后再查出失败的结果集, 两个结果集 JOIN 查出最终所需结果. 不足: 大表查起来真心慢.
	
	解决方案二: SQL中分别查出两个结果集, 返回两个 Map<String,Object> 类型结果集. 代码中迭代合并. 不足: 还要在业务逻辑中进行处理.
	
	*String 作为sql查出来的key, 里面的Object作为value. ibatis返回HashMap*
	
	<img src="https://lh4.googleusercontent.com/-tS3GBCB_D2I/UM7NDfxs_yI/AAAAAAAABAo/Zzbd8ieWEdE/s1118/ibatis-sql-map.png"/>
	
	那么对于上图, key就是媒体名称, value是媒体对象 + 各种属性. 而通过resultMap返回的结果`Map<Object,Object>`. 实际类型是 `Map<Object,Map<Object,Object>>` 这种格式. 
	
	resultMap映射文件中的"property" 就是`Map<Object, Map<>>`中, Map的key值. [参考](http://blog.csdn.net/zhangbo_1991/article/details/6917054)
	
	SQLMap:
	
		<resultMap id="WaasmetaMandaoMap" class="waasUserMoTypejieguo">
			<!--map的key-->
			<result column="rchannel" property="channel" jdbcType="VARCHAR" />
			<result column="mandao" property="mandaoshibai" jdbcType="INTEGER" />
		</resultMap>
    
		<select id="selectMetaMandao"  parameterClass="java.util.HashMap" resultMap="WaasmetaMandaoMap">  
			SELECT $subCategory$ rchannel,sum(mandao) mandao from WAAS_VIEW_USER_RESULT
			GROUP BY $subCategory$
		</select>
		
	DAO :
	
		@Override
		public Map<String, WAASUserMOTypeResult> selectMetaMandao(Map<String, Object> params) {
			//这里的"channel"为sqlmap里的一个字段, 也就是返回Map的key
			return this.getSqlMapClientTemplate().
					queryForMap(sqlmapNamespace + ".selectMetaMandao", params, "channel");
		}
	
	Service进行合并:
	
		@Override
		public List<WAASUserMOTypeResult> selectMeta(Map<String, Object> params) {
			Map<String, WAASUserMOTypeResult> metalist = metaDao.selectMeta(params);
			Map<String, WAASUserMOTypeResult> map2 = metaDao.selectMetaMandao(params);

			List<WAASUserMOTypeResult> list = new ArrayList<WAASUserMOTypeResult>();
			Object[] keys = metalist.keySet().toArray();
			for (Object key : keys) {
				if (key == null) {
					continue;
				}

				WAASUserMOTypeResult result = metalist.get(key.toString());
				if (map2.get(key.toString()) != null) {
					result.setMandaoshibai(map2.get(key.toString()).getMandaoshibai());
				}

				list.add(result);
			}
			return list;
		}
		
* 巧用HashSet:

其实是对HashSet性质的一种巧妙的应用~ 不能不崇拜大牛啊... 好吧,还是我自己没见识~ 代码说明一切.

	/**
	 * 计算群组会员数（去重后）
	 * @param userid
	 * @param groupids
	 * @return 总计可发送人数
	 */
	@SuppressWarnings("unchecked")
	public int CountOfGroups(int userid, int... groupids){
		//TODO performance issue
		Set<String> unique = new HashSet<String>();
		for (int groupid : groupids) {
			boolean isDefault = isGlobalDefaultGroup(groupid);
			String sql = getRuleSqlByGroupid(userid, groupid, isDefault, "buyer_id");
			log.debug(sql);
			String cacheKey = "UserGroup::BuyerId:" + Coder.encryptMD5(sql);
			List<String> list = (List<String>) cachedClient.get(cacheKey);
			if (list == null) {
				list = dao.getJdbcTemplate().queryForList(sql, String.class);
				cachedClient.put(cacheKey, list);
			}

			if (list != null) {
				unique.addAll(list); //<--亮点在这里
				log.debug(groupid + "," + list.size() + "," + unique.size());
			}
		}
		return unique.size();
	} 
		
上面就是利用了HashSet的性质, 顺道查了一下[HashSet的实现](http://alex09.iteye.com/blog/539549).
	
HashSet 的实现其实非常简单，它只是封装了一个 HashMap 对象来存储所有的集合元素，所有放入 HashSet 中的集合元素实际上由 HashMap 的 key 来保存，而 HashMap 的 value 则存储了一个 PRESENT，它是一个静态的 Object 对象。 
		
巧用这件事儿还真得从性质入手, 但是光知道性质还不好使. 重点是灵活运用~ 要是实在想不到怎么用也行, 那就看到了别人的代码记下来. 然后运用~~ 啦啦~~我又好开心...
	
{% include references.md %}
