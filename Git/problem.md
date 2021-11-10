# 一些问题

## 1.gitee添加公钥使用ssh访问

* 生成ssh-key

  ```bash
  #ssh-keygen -t rsa -c "youremail@example.com"
  ```

* 添加id_rsa

  ```bash
  #ssh-add ~/.ssh/id_rsa
  ```

* 配置gitee的ssh公钥

  gitee -> 设置 -> 安全设置 -> SSH公钥 -> 添加公钥

* 确认并添加主机到本机SSH可信列表，若返回`Hi XXX! You've successfully authenticated, but Gitee.com does not provide shell access.`内容则证明添加成功。

  ```bash
  #ssh -T git@gitee.com
  ```

注：

​	执行ssh-add时出现`**Could not open a connection to your authentication agent**`，在执行 ssh-add ~/.ssh/id_ras 时发生此错误。执行如下命令：

```bas
#ssh-agent bash
```

​	然后再执行ssh-add ~/.ssh/id_rsa即可。