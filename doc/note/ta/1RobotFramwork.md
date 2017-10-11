build in libraries
http://robotframework.googlecode.com/hg/doc/libraries/BuiltIn.html
 
 
our robot server
    ip:    10.140.3.10.     root@dell380
    start vnc:    vncserver :1
    python & robot home:   
        cd /usr/local/bin/
        ll command to show startwith l ,it means the command is a softlink. find pyton and pybot
        pybot -> /usr/local/python2.7.3/bin
        python -> /usr/local/python2.7.3/bin/python
 
不能用域名，因为可能没有解析服务器要用ip。
另外需要用path指定firefox的启动路径。
需要能够有display
 
profile的位置必须指定为默认的，如果不是默认有可能代理配置有问题。
 
firefox -profilemanager
? 需要使用firefox默认的profile，但是有可能会有权限问题。
    https必须可以绕过。
    代码需要把一些硬编码改为可以配置的。