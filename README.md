# Convert-Channel
    隐蔽信道（Convert Channel）是MAC下的一种非法通信方式，可以使信息从高密级用户流向低密级用户，从而间接违背BLP模型中“不可上读，不可下写”的安全要求。  
    本次针对两个不同的场景（1）进程描述符和（2）文件最近访问时间 两个漏洞构造隐蔽通道，实现信息从高密级用户流向低密级用户。    
    注：构造隐蔽通道三元组：（1）发送方，一般指系统函数。（2）接受方，一般指系统函数，可以与发送方不一致。（3）共享资源，并非为了通信而存在的资源。  