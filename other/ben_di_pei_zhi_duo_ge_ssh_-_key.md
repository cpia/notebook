# 本地配置多个ssh-key

## 直接上码

* 生成github与oschina的钥匙文件各一份

```
   ssh-keygen -t rsa -f ~/.ssh/id_rsa.github -C "Key for GitHub"
   
   ssh-keygen -t rsa -f ~/.ssh/id_rsa.oschina -C "Key for oschina"
 ```



* 然后在.ssh目录下修改config文件.有就修改没有新增。

  ```
  touch ~/.ssh/config    
  chmod 600 ~/.ssh/config
  ```
* 修改config文件

```
  Host github.com
	IdentityFile ~/.ssh/id_rsa.github
	User yourName
	
  Host *.oschina.net
	IdentityFile ~/.ssh/id_rsa.oschina
	User yourName
```
* 最后的是这样子地
    * ![文件效果](images/2016-04-25_113340.png)