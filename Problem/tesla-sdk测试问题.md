1、crash

(gdb) bt
	#0  0x00000032f0632625 in raise () from /lib64/libc.so.6
	#1  0x00000032f0633e05 in abort () from /lib64/libc.so.6
	#2  0x00000032f062b74e in __assert_fail_base () from /lib64/libc.so.6
	#3  0x00000032f062b810 in __assert_fail () from /lib64/libc.so.6
	#4  0x0000000000dff199 in folly::fbvector<std::shared_ptr<TESLA::SDK::IOThreadConnManager>, std::allocator<std::shared_ptr<TESLA::SDK::IOThreadConnManager> > >::operator[](unsigned long) const () at /root/.conan/data/folly/master20160315/mogu/stable/package/1eccacc7e37d051ed9cb7499346481c0de6c135b/include/folly/FBVector.h:1037
	#5  0x0000000000dfd630 in TESLA::SDK::IOThreadPoolConnManager::GetNextThreadConnManager() const () at /home/tianyuan/tesla-cpp-sdk/src/net/TeslaIoThreadPoolManager.cpp:344
	#6  0x0000000000de5159 in TESLA::SDK::CTeslaSdkInterface::Excute(TESLA::SDK::RequestInfo const&, std::shared_ptr<TESLA::SDK::CRequestCallback>, std::shared_ptr<long>) ()
    at /home/tianyuan/tesla-cpp-sdk/src/TeslaSdkInterface.cpp:73
	#7  0x0000000000c6bb88 in impdu::MGCAppRequestHandler::OnMessageDataHandle (this=0x7fa020ffe300, message_boxed=...)
    at /home/tianyuan/im-cloud-server/server/tesla_router_server/message_handler/im_pdu_request_handler.cc:135
	#8  0x0000000000c6cb1f in impdu::CImPduDataHandler<impdu::MGCAppRequest>::DoHandle (this=0x7fa020ffe300, raw_boxed=...)
    at /home/tianyuan/im-cloud-server/baselib/message_util/message_handler.h:99
	#9  0x0000000000c6ae2f in impdu::MGCAppRequestHandler::Execute (pipepline=0x7fa01f0361e0, raw_pdu=0x7fa01f0362d0)
    at /home/tianyuan/im-cloud-server/server/tesla_router_server/message_handler/im_pdu_request_handler.cc:39
	#10 0x0000000000dd676e in impdu::MessageHandlerFactory::DispatchMessageHandler (pipepline=0x7fa01f0361e0, pdu=0x7fa01f0362d0)
    at /home/tianyuan/im-cloud-server/baselib/message_util/message_handler.cc:62
	#11 0x0000000000dcf086 in IMConnHandler::read (this=0x7fa01f03e0c0, ctx=0x7fa01f025210, msg=...) at /home/tianyuan/im-cloud-server/baselib/net/im_conn_handler.cc:103
	#12 0x0000000000dbf154 in wangle::ContextImpl<IMConnHandler>::read (this=0x7fa01f025210, msg=...)
    at /opt/im-cloud-server-thirdparty/linux/include/wangle/channel/HandlerContext-inl.h:264
	#13 0x0000000000dbf9d1 in wangle::ContextImpl<StatisticsConnHandler>::fireRead (this=0x7fa01f025190, msg=...)
    at /opt/im-cloud-server-thirdparty/linux/include/wangle/channel/HandlerContext-inl.h:166
	#14 0x0000000000dd6124 in StatisticsConnHandler::read (this=0x7fa01f0350e0, ctx=0x7fa01f025190, msg=...)
    at /home/tianyuan/im-cloud-server/baselib/net/statistics_conn_handler.cc:23
	#15 0x0000000000dc01f2 in wangle::ContextImpl<StatisticsConnHandler>::read (this=0x7fa01f025190, msg=...)
    at /opt/im-cloud-server-thirdparty/linux/include/wangle/channel/HandlerContext-inl.h:264
	#16 0x0000000000dc0a6f in wangle::InboundContextImpl<CImPduRawDataDecoder>::fireRead (this=0x7fa01f03a100, msg=...)
    at /opt/im-cloud-server-thirdparty/linux/include/wangle/channel/HandlerContext-inl.h:335
	#17 0x0000000000dc2e3e in wangle::ByteToMessageDecoder<std::unique_ptr<folly::IOBuf, std::default_delete<folly::IOBuf> > >::read (this=0x7fa01f01c210, ctx=0x7fa01f03a100,
    q=...) at /opt/im-cloud-server-thirdparty/linux/include/wangle/codec/ByteToMessageDecoder.h:57
	#18 0x0000000000dc0e9b in wangle::InboundContextImpl<CImPduRawDataDecoder>::read (this=0x7fa01f03a100, msg=...)
    at /opt/im-cloud-server-thirdparty/linux/include/wangle/channel/HandlerContext-inl.h:384
	#19 0x0000000000dc1db2 in wangle::ContextImpl<wangle::AsyncSocketHandler>::fireRead (this=0x7fa01f025110, msg=...)
    at /opt/im-cloud-server-thirdparty/linux/include/wangle/channel/HandlerContext-inl.h:166
	#20 0x0000000000dabe35 in wangle::AsyncSocketHandler::readDataAvailable (this=0x7fa01f01c1b0, len=65536)
    at /opt/im-cloud-server-thirdparty/linux/include/wangle/channel/AsyncSocketHandler.h:138
	#21 0x0000000001082e96 in folly::AsyncSocket::handleRead() () at io/async/AsyncSocket.cpp:1359
	#22 0x0000000001080bca in folly::AsyncSocket::ioReady(unsigned short) () at io/async/AsyncSocket.cpp:1242
	#23 0x000000000110937f in event_persist_closure (base=0x7fa01f014300, flags=1) at event.c:1319
---Type <return> to continue, or q <return> to quit---packet_write_poll: Connection to 10.11.2.103: Can't assign requested address


	分析：获取下一个线程时程序 crash 掉，查看代码：
	thread_conn_managers_[current_thread_id_++];
	是线程不安全的，存在多线程下 current_thread_id_ 异常的情况；
	
	