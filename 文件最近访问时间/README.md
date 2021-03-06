最近学习到了操作系统安全分级的相关知识，对BLP模型有了一定的了解。针对实现了MAC安全机制的操作系统来说，可以有效地防御木马的攻击。但是由于MAC安全机制仍存在一些安全漏洞，不能解决隐蔽通道的问题。记录下本次自己构造了两个隐蔽通道和实现的过程，今后想回顾的时候可以回来看看。  

一、实现环境  
---   
支持开启SeLinux的Linux系统，开启MAC


二、场景分析
---     
（1）选择的隐蔽信道名称：最近访问时间信道。  
（2）信道类型：常驻内存类型。  
（3）隐蔽信道三元组：发送方（fopen函数）、数据结构stat的st_atime成员、接收方（stat函数）。  
（4）存在条件：当用户进程访问一个文件时，系统内核都会更新该文件的最近访问时间。同时，系统的安全策略允许高安全级用户读访问低安全级的文件，而低安全级可以察觉到这种访问。  
（5）发送方动作：发送方读取机密文件，然后，根据该文件内容的当前字节的ASIIC码决定要访问的低级用户的文件数目。  
（6）接收方动作：首先确定一组高级用户可以读的文件。使用系统调用stat读取这些文件的最近访问时间信息，作为原始信息记录下来。待高级用户发送完消息后，使用系统调用stat读取这些文件的最近访问时间信息，并与原始信息比较，将最近访问事件有变化的时间记录目录解释为ASIIC码值。  
（7）噪音情况：出现噪音的原因主要是接收方在接收的CPU时间片可能会不够用。  
  

三、具体攻击流程设计
---    
（1）启动到隐蔽通道处理前的安全核SeLinux。  
（2）创建两个用户user\_h和user\_l，user_h的安全范畴包含user\_l的安全范畴，user\_h的安全级高于user\_l的安全级。此外，设置用户主目录安全级与用户安全级相同；将两个用户主目录自主访问控制权限放开，即均通过chmod命令将其权限设为0777，允许u,g,o可读、写和执行。    
（3）利用两个安全等级不同的用户，检验SeLinux的安全性，“不可上读，不可下写”。即，高级用户可以读低安全级文件，但是不能创建低安全级文件，也不能通过拷贝等方式向下传递信息；低安全级用户也无法读高安全级文件。    
（4）发送方读取机密文件，然后，根据该文件内容的当前字节的ASIIC码决定要访问的低级用户的文件数目。接收方首先确定一组高级用户可以读的文件。使用系统调用stat读取这些文件的最近访问时间信息，作为原始信息记录下来。待高级用户发送完消息后，使用系统调用stat读取这些文件的最近访问时间信息，并与原始信息比较，将最近访问事件有变化的时间记录目录解释为ASIIC码值。  
（5）检验（4）的结果，测试隐蔽信道是否能够成功传输信息。同时，对比之前的操作，分析结果。
 
