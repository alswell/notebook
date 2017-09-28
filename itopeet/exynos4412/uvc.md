### VIDIOC_STREAMON ?_? No space left on device(28)
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
#### References：
- sonix uvc驱动的添加 RT5350支持H264 http://blog.csdn.net/lubing20044793/article/details/36953679
- modprobe XXX not found 解决与Depmod命令; insmod/modprobe的区别 http://blog.csdn.net/adaptiver/article/details/6305404
- rmmod命令 http://man.linuxde.net/rmmod
- https://stackoverflow.com/questions/24923145/open-2-usb-cameras-simultaneously-on-vlc-player-ubuntu-12-04
- https://stackoverflow.com/questions/11394712/libv4l2-error-turning-on-stream-no-space-left-on-device
