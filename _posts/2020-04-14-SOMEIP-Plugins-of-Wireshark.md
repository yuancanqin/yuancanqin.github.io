---
layout:     post
title:      SOME/IP Plugins of Wireshark
subtitle:   Wireshark Plugins
date:       2020-04-14
author:     Richie
header-img: img/post-Wireshark-Plugins.jpg
catalog: true
tags:
    - Wireshark Plugins
---

# 1.	Open source Plugins Project
There're several open source projects about SOME/IP plugins on the Internet. One of them from Github was been updated on 22 Oct 2019 which got four lua files including SD and normal SOME/IP message decoding scripts, it's pretty new one. If you want to know more about it, feel free to contact me. Cause maybe there's a little bit compatibility issue between this plugins and the latest version of Wireshark, or cause by some other issue, as I'm facing now.
# 2.	Modification of someip.lua file
if you get the notice from Wireshark when you load the plugins at the very beginning as below:
```
someip.lua:13:in main chunk
```

For this issue I got a temporary solution as modifying the source code as below:
```
p_someip = Proto("someipa","Scalable service-Oriented MiddlewarE over IP")

local f_msg_id      = ProtoField.uint32("someipa.messageid","MessageID",base.HEX)
local f_len         = ProtoField.uint32("someipa.length","Length",base.HEX)
local f_req_id      = ProtoField.uint32("someipa.requestid","RequestID",base.HEX)
local f_pv          = ProtoField.uint8("someipa.protoversion","ProtocolVersion",base.HEX)
local f_iv          = ProtoField.uint8("someipa.ifaceversion","InterfaceVersion",base.HEX)
local f_mt          = ProtoField.uint8("someipa.msgtype","MessageType",base.HEX)
local f_rc          = ProtoField.uint8("someipa.returncode","ReturnCode",base.HEX)

-- SOME/IP-TP
local f_offset      = ProtoField.uint32("someipa.tp_offset","Offset",base.DEC,nil,0xfffffff0)
local f_reserved    = ProtoField.uint8("someipa.tp_reserved","Reserved",base.HEX,nil,0x0e)
local f_more_seg    = ProtoField.uint8("someipa.tp_more_segments","More Segments",base.DEC,nil,0x01)
```
Just a replacement of the "someip." by "someipa." to avoid the Proto bad argument.
this can bring you get the plugins in your wireshark loading correct at least.

But the wireshark will get stuck sometimes when loading the trace in it.

If you get the better Idea about this, PLZ sharing your brilliant idea.



