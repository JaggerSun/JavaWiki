http://docs.seleniumhq.org/download/
 
https://pypi.python.org/pypi/selenium
 
http://code.google.com/p/selenium/wiki/HtmlUnitDriver#Emulating_a_Specific_Browser
 
because there some error when i use selenium with webdriver so i will do my work with selenium rc.
so i want to do this
 
rc must start the server and use another
 
class path have a unicode not string : solution:firefox_profile.py
http://blog.sina.com.cn/s/blog_5487728b010099ji.html
i modify the firefox_profile.py add a "self.profile_dir = newprof.encode()" line at line 120.
 
enviroment:
https://fttlab317lb.netact.noklab.net/
10.9.204.55
db:
10.9.204.53
基本为空的测试环境
 
https://fttlab316lb.netact.noklab.net/
10.9.204.51
db:10.9.204.49
as:10.9.204.50
TA lab
 
 
https://fttlab233lb.netact.noklab.net/  正常的环境，有数据
10.9.187.202
================
install license: 
/opt/nokia/oss/bin/LicenseUtil.sh -i J1600775.XML false
/opt/nokia/oss/bin/LicenseUtil.sh -d J1600775.XML false
 
TEX server:10.9.232.78
ssh root@nac8tex01
vncserver :1
vncpassword
 
 
scp -r /home/adapftt/wzj/Test002.java root@nac8tex01:/home/wzj/
unzip lib.zip
/opt/tex/lib/java/jre1.6.0_29/bin
export JAVA_HOME=/opt/tex/lib/java/jre1.6.0_29