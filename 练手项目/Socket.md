# 功能描述
想要实现Windows下面，python作为服务器端创建通信，C++作为客户端链接并将图像内容传给python进行处理

## 涉及内容
 - socket 
 - OpenCV

## 有的没的
 严格意义上来说算是头一次上手opencv，小问题非常的多，甚至是因为没有打上`cv.waitKey(0)`卡了好久 = =

 只能说这玩意确实得好好看看

 剩下的懒得敲字了，直接全贴上来，看啥时候需要用再瞅瞅

# 有用的部分

## C++ 

```cpp

#include <stdio.h>
#include <string>
#include <iostream>
#include <Winsock2.h>
#include <opencv2/opencv.hpp>
#include <vector> 
#include<windows.h>   
#pragma comment(lib,"ws2_32.lib")

using namespace cv;
using namespace std;

void main()
{

/*-------------------------------DLL初始化-----------------------------------*/

	WSADATA wsaData;
	WSAStartup(MAKEWORD(2, 2), &wsaData);

/*-------------------------------创建socket-----------------------------------*/
	SOCKET sockClient;
	SOCKADDR_IN addrServer;
	memset(&addrServer, 0, sizeof(addrServer));  //每个字节都用0填充
	sockClient = socket(AF_INET, SOCK_STREAM, 0);
	addrServer.sin_addr.S_un.S_addr = inet_addr("127.0.0.1");
	addrServer.sin_family = AF_INET; //定义地址族IPv4因特网域
	addrServer.sin_port = htons(2345);  //连接端口
	
	//连接到服务端
	connect(sockClient, (SOCKADDR*)&addrServer, sizeof(SOCKADDR));

/*-------------------------------准备读取视频流-----------------------------------*/
	Mat image;
	VideoCapture capture;
	vector<uchar> data_encode;
	image = capture.open(0);
	while (capture.read(image))
	{
		cv::resize(image, image, cv::Size(1920, 1080));
		imencode(".jpg", image, data_encode);
		string binary_data(data_encode.begin(), data_encode.end());
/*-------------------------------整理告知报文-----------------------------------*/
		int len_encode = binary_data.size();
		string len = to_string(len_encode);
		int length = len.length();
		for (int i = 0; i < 16 - length; i++)
		{
			len = len + " ";
		}

		send(sockClient, len.c_str(), strlen(len.c_str()), 0);
		Sleep(10);
/*-------------------------------发送图片内容-----------------------------------*/		
		send(sockClient, binary_data.c_str(), binary_data.size(), 0);

	}
	closesocket(sockClient);
	WSACleanup();
}
connect(sockClient, (SOCKADDR*)&addrServer, sizeof(SOCKADDR));

```


## python

```python

import socket
import cv2 as cv
import numpy as np

# 创建一个socket对象，默认TCP套接字
s = socket.socket()
s.bind(('127.0.0.1',2345))  # 绑定端口
s.listen(5)                 # 监听端口
print("正在连接中……")

# 建立连接之后，持续等待连接

# 阻塞等待连接
sock,addr = s.accept()
print(sock,addr)
# 一直保持发送和接收数据的状态 
cv.namedWindow("out", cv.WINDOW_FREERATIO)

while True:
    length = sock.recv(16) #首先接收来自客户端发送的大小信息
    print('receive the first head lenth:')
    print(length)
    realLength = int(length)
    stringData = sock.recv(realLength)
    print('receive the s ring data: \n')
    
    # 输出解码后的图像
    string2Numpy = np.frombuffer(stringData, np.uint8)
    numpy2Image = cv.imdecode(string2Numpy,cv.IMREAD_COLOR)
    showImage = cv.imshow("out",numpy2Image)
    cv.waitKey(1)
    
sock.close()
```