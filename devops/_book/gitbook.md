### 安装nodejs和gitbook环境

* wget https://nodejs.org/dist/v10.9.0/node-v10.9.0-linux-x64.tar.xz    // 下载
* tar xf  node-v10.9.0-linux-x64.tar.xz       // 解压
* cd node-v10.9.0-linux-x64/                  // 进入解压目录
* ./bin/node -v                               // 执行node命令 查看版本
	v10.9.0                               //大于4.0版本就好
* ln -s /usr/software/nodejs/bin/npm   /usr/local/bin/ 
* ln -s /usr/software/nodejs/bin/node   /usr/local/bin/
* npm install gitbook-cli -g
* ln -s /usr/local/nodejs/bin/gitbook /usr/local/bin/

### 生成gitbook初始化
* gitbook init                            
	* README.md //这是本页面
	* SUMMARY.MD //这是电子书的目录
* gitbook build      
	* 生成静态站点，当前目录会生成_book目录，即web静态站点

### 相关联系
* [参考文档](https://einverne.github.io/gitbook-tutorial/)
* [生成目录折叠](https://yunchangwang.github.io/2018/01/28/gitbook%E7%9B%AE%E5%BD%95%E6%8A%98%E5%8F%A0/)
	* npm install gitbook-plugin-toggle-chapters
	* 在 book.json 添加 {"plugins":["toggle-chapters"]}
* [生成book.json](http://www.chengweiyang.cn/gitbook/customize/book.json.html)
