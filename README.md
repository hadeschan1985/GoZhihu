# 项目：GoZhihu

已实现API功能： 

Version1支持:

1. 通过单个问题id获取批量答案, 可保存图片和HTML回答
2. 通过集合id获取批量问题后获取批量答案, 可保存图片和HTML回答

Version2支持:

3. 根据用户唯一域名id获取其关注的人，和关注她的人, 导出CSV(正在实现)
4. 获取一个人的所有回答(正在实现)

~~鸡肋(不要看):~~

~~5. 关注别人（风险大容易被封杀,建议不要使用）~~

~~6. 登录(本来可以登录，后来某乎加了倒置验证码),该验证码破解较容易~~

~~7. 通过答案id获取单个回答（有点鸡肋，还是写了）~~


目录结构及获取的数据如下:

```

--- zhihu_windows_amd64.exe 
--- zhihu_linux_x86_64
--- cookie.txt
--- data  收藏夹和回答生成的数据在data文件夹
     --- 27761934-如何让自拍的照片看上去像是别人拍的？.xx   *去重标志,如果要重新获取答案,请将`.xx`文件去掉
     --- 27761934  * 回答文件集
        ---zhi-zhi-zhi-41-89-167963702 * 一个用户的回答 包括图片
           --- zhi-zhi-zhi-41-89-167963702的回答.html (里面的图片链接都替换成本地链接)
           --- https###pic1.zhimg.com#v2-22407b227c9a7a19aa0057f38bf6e754_r.png  (已经不是这种样子了)
               https###pic1.zhimg.com#v2-7782ff69838c379173415458b97b5008_xll.jpg
               https###pic1.zhimg.com#v2-c41bf767819fbc61b3ff7bb4c2900884_r.jpg

        ---zhi-zhi-wei-zhi-zhi-36-38-164986419
        ---zhi-zhi-wei-zhi-zhi-hu-hu-wei-hu-hu-164880780

     --- 27761934-html  生成的html集,可以点击查看(可选择防盗链, 请用非火狐浏览器查看)
        --- 1.html
        --- 2.html

--- people   获取用户粉丝数据和所有回答在此文件夹下
```

## 一.小白指南

> 可以下载EXE文件

Golang开发的爬虫，小白用户请下载[释出版本二进制](https://github.com/hunterhug/GoZhihu/releases)中的`zhihu_windows_amd64.exe`，并在同一目录下新建一个`cookie.txt`文件，

打开火狐浏览器后人工登录知乎，按F12，点击网络，刷新一下首页，然后点击第一个出现的`GET /`，找到消息头请求头，复制Cookie，然后粘贴到cookie.txt

![](doc/cookie.png)

点击EXE后,可选JS解决防盗链（这个是你要发布到自己的网站如：[减肥成功是什么感觉？给生活带来哪些改变？](http://www.lenggirl.com/zhihu/26613082-html/1.html)）
我们自己本地看的话就不要选择防盗链了！回答个数已经限制不大于500个。如果没有答案证明Cookie失效，请重新按照上述方法手动修改`cookie.txt`。

你也可以全部图片保存在本地, 这样数据会巨大!

结果：

![](doc/1.png)
![](doc/2.png)

## 二.API说明

下载

```bash
go get -u -v github.com/hunterhug/GoZhihu
```

此包在哥哥封装的爬虫包基础上开发：[土拨鼠（tubo）](https://github.com/hunterhug/GoSpider)，请进入`main`文件夹运行成品程序，`IDE`开发模式下，运行路径是不一样的，请在`IDE`项目根目录放`cookie.txt`文件

二次开发时你只需`import`本包。

```go
import zhihu "github.com/hunterhug/GoZhihu/src"
```

API如下：

1.Cookie相关
```go
// 设置cookie，需传入文件位置，文件中放cookie
func SetCookie(file string) error 
```

2.问题相关
```go
// 构造问题链接，返回url
func Question(id string) string

// 抓答案，需传入限制和页数,每次最多抓20个答案
func CatchAnswer(url string, limit, page int) ([]byte, error)

// 结构化回答，返回一个结构体
func StructAnswer(body []byte) (*Answer, error)
```

3.集合相关
```go
// 抓取收藏夹第几页列表
func CatchCoolection(id, page int) ([]byte, error)

// 抓取全部收藏夹页数,并返回问题ID和标题
func CatchAllCollection(id int) map[string]string 

// 解析收藏夹，返回问题ID和标题
func ParseCollection(body []byte) map[string]string
```

4.工具相关
```go
// 输出HTML选择防盗链方式
func SetPublishToWeb(put bool)

// 输出友好格式HTML，返回问题ID,回答ID，标题，作者，还有HTML
func OutputHtml(answer DataInfo) (qid, aid int, title, who, html string)

// 抓取图片前需要设置true
func SetSavePicture(catch bool) 

// 抓取html中的图片，保存图片在dir下
func SavePicture(dir string, body []byte) 

// 遇到返回的JSON中有中文乱码，请转意
func JsonBack(body []byte) ([]byte, error)

// 设置爬虫调试日志级别，开发可用:debug,info
func SetLogLevel(level string) 

// 设置爬虫暂停时间
func SetWaitTime(w int)
```

5.用户相关
```go
// 关注某人，建议不用
func FollowWho(who string) ([]byte, error) {
// 抓取用户：fensi 抓取你的粉丝，否则，抓取你的偶像，token为用户：如https://www.zhihu.com/people/hunterhug中的hunterhug,limit限制最多20条
func CatchUser(fensi bool, token string, limit, offset int) ([]byte, error) {
// 解析用户
func ParseUser(data []byte) FollowData {
}
```

建议不用API：

```go
// 抓单个答案，需传入问题ID和答案ID 鸡肋功能，弃用！
func CatchOneAnswer(Qid, Aid string) ([]byte, error) {
// 解析单个答案，待完善
func ParseOneAnswer(data []byte) map[string]string {
```

使用时需要先`SetCookie()`，再根据具体进行开发，使用如下：

```go
package main

import (
	"fmt"
	zhihu "github.com/hunterhug/GoZhihu/src"
	"strings"
)

// API使用说明
func main() {
	//  1. 设置爬虫暂停时间，可选
	zhihu.SetWaitTime(1)

	// 2. 调试模式设置为debug，可选
	zhihu.SetLogLevel("info")

	// 3. 需先传入cookie，必须
	e := zhihu.SetCookie("./cookie.txt")
	if e != nil {
		panic(e.Error())
	}

	// 4.构建问题，url差页数
	q := zhihu.Question("28467579")
	fmt.Println(q)

	// 5.抓取问题回答，按页数，传入页数是为了补齐url，策略是循环抓，直到抓不到可认为页数已完
	page := 1
	limit := 20
	body, e := zhihu.CatchAnswer(q, limit, page)
	if e != nil {
		fmt.Println(e.Error())
		return
	}
	if strings.Contains(string(body), "error") { //可能cookie失效
		b, _ := zhihu.JsonBack(body)
		fmt.Println(string(b))
	}

	// 6.结构化回答
	answers, e := zhihu.StructAnswer(body)
	if e != nil {
		fmt.Println(e.Error())
	} else {
		// 就不打出来了
		//fmt.Printf("%#v\n", answers.Page)
		//fmt.Printf("%#v\n", answers.Data)
	}

	// 7. 选择OutputHtml不要防盗链，因为回答输出的html经过了处理，所以我们进行过滤出好东西
	zhihu.SetPublishToWeb(false)
	qid,aid,t,who,html:=zhihu.OutputHtml(answers.Data[0])
	fmt.Println(qid)
	fmt.Println(aid)
	fmt.Println(t)
	fmt.Println(who)

	// 8. 抓图片
	zhihu.SetSavePicture(false)
	zhihu.SavePicture("test", []byte(html))

	// 9. 抓集合，第2页
	b, e := zhihu.CatchCoolection(78172986, 2)
	if e != nil {
		fmt.Println(e.Error())
	} else {
		// 解析集合
		fmt.Printf("%#v",zhihu.ParseCollection(b))
	}
}
```

登录待破解验证码：

```go
// 登录，验证码突破不了，请采用SetCookie
func Login(email, password string) ([]byte, error)
```

![](doc/ca.png)

```
_xsrf:2fc4811def8cd9f358465e4ea418b23b
password:z13112502886
captcha:{"img_size":[200,44],"input_points":[[19.2969,28],[40.2969,28],[68.2969,27],[89.2969,31],[112.297,34],[138.297,15],[161.297,27]]}
captcha_type:cn
email:wefwefwefwef@qq.com
```

待收集`https://www.zhihu.com/captcha.gif?r=1497501049814&type=login&lang=cn`进行机器学习！

![](doc/captcha.gif)

## 三.编译执行文件方式

### Linux操作系统下跨平台交叉编译

Linux二进制

```bash
cd main

# 64位
GOOS=linux GOARCH=amd64 go build -ldflags "-s -w" -x -o zhihu_linux_amd64 main.go

# 32位
GOOS=linux GOARCH=386 go build -ldflags "-s -w" -x -o zhihu_linux_386 main.go
```

Windows二进制

```bash
# 64位
GOOS=windows GOARCH=amd64 go build -ldflags "-s -w" -x -o zhihu_windows_amd64.exe main.go

# 32位
GOOS=windows GOARCH=386 go build -ldflags "-s -w" -x -o zhihu_windows_386.exe main.go
```

### Windows操作系统下编译

```bash
go build -o zhihu.exe main.go
```

如果你觉得项目帮助到你,欢迎请我喝杯咖啡,或加QQ：459527502

微信
![微信](https://raw.githubusercontent.com/hunterhug/hunterhug.github.io/master/static/jpg/wei.png)

支付宝
![支付宝](https://raw.githubusercontent.com/hunterhug/hunterhug.github.io/master/static/jpg/ali.png)

## 四.环境配置

### Ubuntu安装

[云盘](https://yun.baidu.com/s/1jHKUGZG)下载源码解压.下载IDE也是解压设置环境变量.

```bash
vim /etc/profile.d/myenv.sh

export GOROOT=/app/go
export GOPATH=/home/jinhan/code
export GOBIN=$GOROOT/bin
export PATH=.:$PATH:/app/go/bin:$GOPATH/bin:/home/jinhan/software/Gogland-171.3780.106/bin

source /etc/profile.d/myenv.sh
```

### Windows安装

[云盘](https://yun.baidu.com/s/1jHKUGZG) 选择后缀为msi安装如1.6

环境变量设置：

```bash
Path G:\smartdogo\bin
GOBIN G:\smartdogo\bin
GOPATH G:\smartdogo
GOROOT C:\Go\
```

### docker安装

我们的库可能要使用各种各样的工具，配置连我这种专业人员有时都搞不定，而且还可能会损坏，所以用docker方式随时随地开发。

先拉镜像

```bash
docker pull golang:1.8
```

Golang环境启动：

```bash
docker run --rm --net=host -it -v /home/jinhan/code:/go --name mygolang golang:1.8 /bin/bash

root@27214c6216f5:/go# go env
GOARCH="amd64"
```

其中`/home/jinhan/code`为你自己的本地文件夹（虚拟GOPATH），你在docker内`go get`产生在`/go`的文件会保留在这里，容器死掉，你的`/home/jinhan/code`还在，你可以随时修改文件配置。

启动后你就可以在里面开发了。


# LICENSE

欢迎加功能(PR/issues),请遵循Apache License协议(即可随意使用但每个文件下都需加此申明）

```
Copyright 2017 by GoSpider author.
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at
    http://www.apache.org/licenses/LICENSE-2.0
Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License
```
