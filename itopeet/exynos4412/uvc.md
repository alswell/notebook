### VIDIOC_STREAMON: No space left on device
http://blog.csdn.net/zhangwu1241/article/details/52983271

先说下原因，linux中为usb camera提供了一个统一的驱动以方便使用，只要符合驱动规范就可以实现即插即用usb camera设备，即免驱动安装乐。 usb bus的 bandwidth是有限的，而本着贪心原则，camera会要求获取最大带宽（usb2.0 camera480？）；而将两个camera接入一路usb bus，打开第二个camera就会出现”No space left on device”的错误。
- 解决方法一：
接在不同的usb bus上，使用lsusb 命令查看bus信息， 类似“Linux Foundation 2.0 root hub”表示该总线为usb 2.0;
- 解决方法二：
降低打开视频流的分辨率，改为320x240;并对uvcvideo驱动设置参数，强制为camera分配带宽时计算所需带宽而非申请全部带宽；（只对YUYV格式有效，对有些camera此方法可以支持640x480分辨率） 
```
sudo rmmod uvcvideo 
sudo modprobe uvcvideo quirks=128 
```
