---
layout: post
title: Openfire MessageRouter 消息路由
category: notes
---

上回的需求是要把消息路由给指定的资源上去，通过增加插件或者是修改源码的方式改变现有路由规则。于是了解了一下Openfire消息路由的实现方式。

主要是这个类·MessageRouter.route·完成的。

源码：

    public void route(Message packet) {
        if (packet == null) {
            throw new NullPointerException();
        }
        ClientSession session = sessionManager.getSession(packet.getFrom());
        try {
            // Invoke the interceptors before we process the read packet
            InterceptorManager.getInstance().invokeInterceptors(packet, session, true, false);
            if (session == null || session.getStatus() == Session.STATUS_AUTHENTICATED) {
                JID recipientJID = packet.getTo();

                // Check if the message was sent to the server hostname
                if (recipientJID != null && recipientJID.getNode() == null && recipientJID.getResource() == null &&
                        serverName.equals(recipientJID.getDomain())) {
                    if (packet.getElement().element("addresses") != null) {
                        // Message includes multicast processing instructions. Ask the multicastRouter
                        // to route this packet
                        multicastRouter.route(packet);
                    }
                    else {
                        // Message was sent to the server hostname so forward it to a configurable
                        // set of JID's (probably admin users)
                        sendMessageToAdmins(packet);
                    }
                    return;
                }

                try {
                    // Deliver stanza to requested route
                    routingTable.routePacket(recipientJID, packet, false);
                }
                catch (Exception e) {
                	log.error("Failed to route packet: " + packet.toXML(), e);
                    routingFailed(recipientJID, packet);
                }
            }
            else {
                packet.setTo(session.getAddress());
                packet.setFrom((JID)null);
                packet.setError(PacketError.Condition.not_authorized);
                session.process(packet);
            }
            // Invoke the interceptors after we have processed the read packet
            InterceptorManager.getInstance().invokeInterceptors(packet, session, true, true);
        } catch (PacketRejectedException e) {
            // An interceptor rejected this packet
            if (session != null && e.getRejectionMessage() != null && e.getRejectionMessage().trim().length() > 0) {
                // A message for the rejection will be sent to the sender of the rejected packet
                Message reply = new Message();
                reply.setID(packet.getID());
                reply.setTo(session.getAddress());
                reply.setFrom(packet.getTo());
                reply.setType(packet.getType());
                reply.setThread(packet.getThread());
                reply.setBody(e.getRejectionMessage());
                session.process(reply);
            }
        }
    }

做了几件事情：

1. 拿到了客户端session：ClientSession session = sessionManager.getSession(packet.getFrom()); ClientSession 主要动作是设置用户聊天是设置和使用的策略，获取用户名，获取和设置用户当前状态Presence，当前用户是否为匿名用户。

2. 通过反射，调用plugin（开始和结束调用两次）：InterceptorManager.getInstance().invokeInterceptors(packet, session, true, false);

3. 进行一些列bla bla的判断，把该给多媒体路由的给多媒体路由，把该给admin的发给admin

4. 最后就是调用RoutingTableImpl.routePacket（路由实现类）

5. 没路由出去的要么返回拒绝消息

RoutingTableImpl.routePacket ：

1. 如果接受者的domain和服务器的domain一致，就调用routeToLocalDomain

2. 如果接受者的domain包含服务器的domain，就调用routeToComponent。（消息发送对象为二级域名的情况，如主域名是：im.ebnew.com，如果此时需要我们扩展一个sub-domain，就会变成：ios.im.ebnew.com。服务器在此时就会调用routeToComponent）

3. 除此之外，服务器会调用 routeToRemoteDomain

routeToLocalDomain 源码：

	private boolean routeToLocalDomain(JID jid, Packet packet,
			boolean fromServer) {
		boolean routed = false;
		if (jid.getResource() == null) {
		    // Packet sent to a bare JID of a user
		    if (packet instanceof Message) {
		        // Find best route of local user
		        routed = routeToBareJID(jid, (Message) packet);
		    }
		    else {
		        throw new PacketException("Cannot route packet of type IQ 
                    or Presence to bare JID: " + packet.toXML());
		    }
		}
		else {
		    // Packet sent to local user (full JID)
		    ClientRoute clientRoute = usersCache.get(jid.toString());
		    if (clientRoute == null) {
		        clientRoute = anonymousUsersCache.get(jid.toString());
		    }
		    if (clientRoute != null) {
		        if (!clientRoute.isAvailable() && routeOnlyAvailable(packet, fromServer) &&
		                !presenceUpdateHandler.hasDirectPresence(packet.getTo(), packet.getFrom())) {
		        	Log.debug("Unable to route packet. Packet should only be sent to 
                        available sessions and the route is not available. {} ", packet.toXML());
		            routed = false;
		        }
		        else {
		            if (localRoutingTable.isLocalRoute(jid)) {
		                // This is a route to a local user hosted in this node
		                try {
		                    localRoutingTable.getRoute(jid.toString()).process(packet);
		                    routed = true;
		                } catch (UnauthorizedException e) {
		                    Log.error("Unable to route packet " + packet.toXML(), e);
		                }
		            }
		            else {
		                // This is a route to a local user hosted in other node
		                if (remotePacketRouter != null) {
		                    routed = remotePacketRouter
		                            .routePacket(clientRoute.getNodeID().toByteArray(), jid, packet);
		                    if (!routed) {
		                    	removeClientRoute(jid); // drop invalid client route
		                    }
		                }
		            }
		        }
		    }
		}
		return routed;
	}


情况：

1. 发送给bareJID的情况

    找到这个subscriber所对应的所有活的session。

    从这些session里面选择优先级最高的一个session。调用LocalClientSession的deliver方法把消息发出去。

    如果优先级最高的是一组session，那么再根据session的presence信息中的一些状态值排序，排序算法：多个session中，最后一个连接到服务器的session优先级最高。就用这个session把消息发出去。

2. 发送给fullJID的情况（即需求中指定resource情况）

    在 cache 中找fullJID，用户指定资源是否在线，以及是否可用，如果没有，return false.

    调用 messageRouter.routingFailed(jid, packet); 路由失败策略

MessageRouter.routingFailed 源码：

    public void routingFailed(JID receipient, Packet packet) {
        // If message was sent to an unavailable full JID of a user then retry using the bare JID
        if (serverName.equals(receipient.getDomain()) && receipient.getResource() != null &&
                userManager.isRegisteredUser(receipient.getNode())) {
            routingTable.routePacket(new JID(receipient.toBareJID()), packet, false);
        } else {
            // Just store the message offline
            messageStrategy.storeOffline((Message) packet);
        }
    }

情况：

1. fullJID路由失败，尝试路由到bare JID;(返回上一步骤，此时就会按路由规则，发送至其他resource)，因此，直接把这里改了，就可以实现发送到指定资源的需求了。但是也会影响到其他正常情况的路由。解决方案思考中……

2. 离线存储


--EOF--