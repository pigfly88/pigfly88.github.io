---
layout: post
title:  linux常用命令
categories: linux
---

### 文件和目录

搜索文本内容：`cat test.log | grep 'passwd'`

动态展示最后20行内容：`tail -n 20 -f test.log`

搜索文件：`find / -name test.log`

文件和目录权限：

<table class="news">
<tbody><tr style="background-color: lightblue; text-align:center"><td>元件</td><td>內容</td><td>疊代物件</td><td>r</td><td>w</td><td>x</td></tr>
<tr><td>檔案</td><td>詳細資料data</td><td>文件資料夾</td><td>讀到文件內容</td><td>修改文件內容</td><td>執行文件內容</td></tr>
<tr><td>目錄</td><td>檔名</td><td>可分類抽屜</td><td>讀到檔名</td><td>修改檔名</td><td>進入該目錄的權限(key)</td></tr>
</tbody></table>

<table class="news">
<tbody><tr style="background-color: lightblue; text-align:center"><td>操作動作</td><td style="width: 70px;">/dir1</td><td style="width: 70px;">/dir1/file1</td><td style="width: 70px;">/dir2</td><td>重點</td></tr>
<tr style="text-align:center"><td>讀取 file1 內容</td><td>x</td><td>r</td><td>-</td><td>要能夠進入 /dir1 才能讀到裡面的文件資料！</td></tr>
<tr style="text-align:center"><td>修改 file1 內容</td><td>x</td><td>rw</td><td>-</td><td>能夠進入 /dir1 且修改 file1 才行！</td></tr>
<tr style="text-align:center"><td>執行 file1 內容</td><td>x</td><td>rx</td><td>-</td><td>能夠進入 /dir1 且 file1 能運作才行！</td></tr>
<tr style="text-align:center"><td>刪除 file1 檔案</td><td>wx</td><td>-</td><td>-</td><td>能夠進入 /dir1 具有目錄修改的權限即可！</td></tr>
<tr style="text-align:center"><td>將 file1 複製到 /dir2</td><td>x</td><td>r</td><td>wx</td><td>要能夠讀 file1 且能夠修改 /dir2 內的資料</td></tr>
</tbody></table>

修改目录权限：chmod -R 755 /usr/www

修改所有者和所属组：chown -R www:www /usr/www

查看磁盘剩余空间：`df -h`, df: disk free，h为human的意思，即人性化显示

	
查看文件里面匹配内容的上下文：
```
cat /workspace/log/test.log | grep -5 'not connect to' # 查看前后5行
cat /workspace/log/test.log | grep -A 5 'not connect to' # 查看前面5行
cat /workspace/log/test.log | grep -B 5 'not connect to' # 查看后面5行
```

磁盘空间满了，怎样快速查看是哪个目录或文件占用空间大:
```
du -m --max-depth=1 /workspace/log/ | sort -nr
du -ah --max-depth=1 /
du -ah --max-depth=1 /workspace/data | grep 'G' | sort -nr
```

### 系统相关
```shell
$ uptime
02:34:03 up 2 days, 20:14, 1 user, load average: 0.63, 0.83, 0.88
```
后面三个数值为过去1分钟、5分钟、15分钟的平均负载，平均负载即活跃进程数。
比如当平均负载为2时，在只有2个CPU的系统上，意味着所有的CPU都刚好被完全占用。在4个CPU的系统上，意味着CPU有50%的空闲。

### 网络相关

查看3306端口状态：
```
ss -lp | grep 3306 # ss: socket status，查看socket状态
netstat -lnp | grep 80
```

添加PATH变量：
```
vi /etc/profile
export PATH=/usr/local/php/bin:$PATH
source /etc/profile
```