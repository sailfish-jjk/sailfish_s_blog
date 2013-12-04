----
layout: post
title: Openfire好友关系
category: notes
----

继续学习Openfire，这次是好友关系的部分。

我们经常需要把某个用户的好友关系导出，进行一系列的操作，这时候就用到了RosterManager。通过插件的方式，把RosterManager的`getRoster(String username)`方法，把用户的好友关系发布成Dubbo服务，为其他应用提供服务。

这部分主要表述的是为我们的服务加上缓存，Openfire本身自带缓存机制，这个之前已经讨论过了。这次的好友关系，我想把其改造成通过Redis的方式缓存，与公司内私有云环境集成，得到可伸缩扩展的效果。

为保证数据一致性，需要监听Openfire好友关系的变更状态，以同步更新缓存。google未果后，想到两个解决方案：XMPP协议入手，或者从Openfire源码入手。协议又臭又长，我选择了后者。

打开Openfire源码，找到roster包，里面就都是好友关系相关的实现啦。首先看到熟悉的RosterManager，好友关系的管理类，就从这儿入手吧。

	@Override
	public void initialize(XMPPServer server) {
	    super.initialize(server);
	    this.server = server;
	    this.routingTable = server.getRoutingTable();

	    RosterEventDispatcher.addListener(new RosterEventListener() {
	        public void rosterLoaded(Roster roster) {
	            // Do nothing
	        }

	        public boolean addingContact(Roster roster, RosterItem item, boolean persistent) {
	            // Do nothing
	            return true;
	        }

	        public void contactAdded(Roster roster, RosterItem item) {
	            // Set object again in cache. This is done so that other cluster nodes
	            // get refreshed with latest version of the object
	            rosterCache.put(roster.getUsername(), roster);
	        }

	        public void contactUpdated(Roster roster, RosterItem item) {
	            // Set object again in cache. This is done so that other cluster nodes
	            // get refreshed with latest version of the object
	            rosterCache.put(roster.getUsername(), roster);
	        }

	        public void contactDeleted(Roster roster, RosterItem item) {
	            // Set object again in cache. This is done so that other cluster nodes
	            // get refreshed with latest version of the object
	            rosterCache.put(roster.getUsername(), roster);
	        }
	    });
	}

RosterManager的初始化方法里，看到了`RosterEventDispatcher.addListener()`方法，正是我们想要的好友状态变更监听器。 已经看到，Openfire默认的实现就是在这里进行的缓存更新：`rosterCache.put(roster.getUsername(), roster);`

为了保证Openfire以后可以顺利升级，决定不改动源代码，而采用插件重写的方式，对外部应用需要调用的好友关系进行缓存。插件实现还是采用匿名内部类的方式，当然，直接实现RosterEventListener接口也是可行的~


	public class RosterServiceImpl implements RosterService {

		private static final Logger log = LoggerFactory.getLogger(RosterServiceImpl.class);
		private static final String ROSTER_CACHE_KEY = "openfire_roster_";
		//测试时模仿Openfire的缓存方式进行缓存，开发时替换成Redis
		private static Map<String, Set<String>> caches = new ConcurrentHashMap<String, Set<String>>();
		private RosterManager rosterManager;

		public RosterServiceImpl() {
			log.info("RosterServiceImpl init...");
			//实例化RosterManager
			rosterManager = XMPPServer.getInstance().getRosterManager();
			//为RosterService增加监听
			RosterEventDispatcher.addListener(new RosterEventListener() {
				public void rosterLoaded(Roster roster) {
					// Do nothing
				}

				public boolean addingContact(Roster roster, RosterItem item, boolean persistent) {
					// Do nothing
					return true;
				}

				public void contactAdded(Roster roster, RosterItem item) {
					// Set object again in cache. This is done so that other cluster nodes
					// get refreshed with latest version of the object
				}

				public void contactUpdated(Roster roster, RosterItem item) {
					// 联系人关系更新事件（from。。。to。。。both。。。none。。。）
					// 更新缓存中的联系人好友关系
					log.info("----------------contact contactUpdated!~" + roster.getUsername() + ":" + item.getJid()
							+ "&relation:" + item.getSubStatus());
					//业务要求只有关系为Both的好友才算好友，因此只更新Both好友列表缓存（添加好友）
					if (RosterItem.SUB_BOTH.equals(item.getSubStatus())) {
						Set<String> friendList = getFriendList(roster.getUsername());
						//更新缓存
						caches.put(ROSTER_CACHE_KEY + roster.getUsername(), friendList);
						log.info("cache update :" + ROSTER_CACHE_KEY + roster.getUsername()
								+ " ...after update listsize is:" + friendList.size());
					}

				}

				public void contactDeleted(Roster roster, RosterItem item) {
					// 联系人关系删除事件
					// 更新缓存中的好友关系
					log.info("----------------contact contactDeleted!~" + roster.getUsername() + ":" + item.getJid()
							+ "&relation:" + item.getSubStatus());
					//取消关注时，更新缓存信息（取消关注时，好友关系为none）
					if (RosterItem.SUB_NONE.equals(item.getSubStatus())) {
						Set<String> friendList = getFriendList(roster.getUsername());
						//更新缓存
						caches.put(ROSTER_CACHE_KEY + roster.getUsername(), friendList);
						log.info("cache delete :" + ROSTER_CACHE_KEY + roster.getUsername()
								+ " ...after update listsize is:" + friendList.size());
					}
				}
			});
		}

		@Override
		public Set<String> getFriendList(String username) {
			Set<String> set = new HashSet<String>();
			try {
				Roster roster = rosterManager.getRoster(username);
				//联系人列表。
				Collection<RosterItem> items = roster.getRosterItems();
				//只查询互为好友关系的好友列表。
				for (RosterItem item : items) {
					//互为好友的加入到结果集合中。
					if (RosterItem.SUB_BOTH.equals(item.getSubStatus()) && item.getJid() != null) {
						set.add(item.getJid().getNode());
					}
				}

			} catch (UserNotFoundException e) {
				log.error("user not found", e);
			}
			return set;
		}

		@Override
		public Set<String> getFriendListFromCache(String username) {
			if (caches.containsKey(ROSTER_CACHE_KEY + username.trim())) {
				log.info("cache hit!" + ROSTER_CACHE_KEY + username.trim() + "... after update listsize is:"
						+ caches.get(ROSTER_CACHE_KEY + username.trim()).size());
				return caches.get(ROSTER_CACHE_KEY + username.trim());
			}
			Set<String> set = getFriendList(username);
			caches.put(ROSTER_CACHE_KEY + username.trim(), set);
			log.info("not found in cache , and put friendList into cache ... with key:" + ROSTER_CACHE_KEY
					+ username.trim() + "   ...size:" + set.size());
			return set;
		}

	}


写完丢到测试环境，运行正常~ 

既然问题已经解决了，开始思考如何从协议的角度入手解决此问题。首先肯定是对协议进行了解：[RFC6121](http://wiki.jabbercn.org/RFC6121#.E6.9B.B4.E6.96.B0.E4.B8.80.E4.B8.AA.E5.90.8D.E5.86.8C.E6.9D.A1.E7.9B.AE)


通过协议可以知道，当好友关系状态改变时，服务器从用户向联系人递送"subscribed"类型的出席信息节，并且初始化一个名册推送给这个联系人的所有已请求名册的可用资源，包含一个关于这个用户的更新的名册条目，其'subscription'属性值进行改变。

因此，我们可以通过拦截iq包，或者presence包进行对好友关系变更的处理。代码如下：


	public class SubPresenceInterceptor implements PacketInterceptor {

		private static final Logger log = LoggerFactory.getLogger(SubPresenceInterceptor.class);

		@Override
		public void interceptPacket(Packet packet, Session session, boolean incoming, boolean processed)
				throws PacketRejectedException {

			if (session != null) {
				log.debug("intercept packet from:" + packet.getFrom() + ",to:" + packet.getTo());
			}

			if (packet != null) {
				//只处理出席信息。(Presence包示例，IQ包类似，只要筛选出NAMESPACE是jabber:iq:roster的就行了)
				if (packet instanceof Presence) {
					Presence presence = (Presence) packet;
					//处理订阅类的出席信息，会接收到两次出席信息，只处理resource不为空的那次。
					if (Presence.Type.subscribe.equals(presence.getType())) {
						if (presence.getFrom() != null && !StringUtils.isEmpty(presence.getFrom().getResource())) {
							RosterManager rosterManager = XMPPServer.getInstance().getRosterManager();
							try {
								Roster roster = rosterManager.getRoster(presence.getFrom().getNode());
								RosterItem item = roster.getRosterItem(presence.getTo());//好友之间订阅关系
								log.info("roster item relationship ...:" + presence.getFrom().getNode() + " with "
										+ presence.getTo() + " is :" + item.getSubStatus());
								if ((RosterItem.SUB_NONE.equals(item.getSubStatus()) || RosterItem.SUB_BOTH.equals(item
										.getSubStatus())) && item.getJid() != null) {
									//好友关系产生变化时，更新缓存
								}

							} catch (UserNotFoundException e) {
								log.error("user not found", e);
							}

						}
					}
				}
			}
		}

	}

-EOF-
