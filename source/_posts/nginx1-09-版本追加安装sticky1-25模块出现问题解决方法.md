---
title: nginx1.09.*+版本追加安装sticky1.25模块出现问题解决方法
date: 2017-11-21 15:15:10
category:
	- 服务器技术
tags: 
	- Nginx
 	- sticky
	- 反向代理
---
## 问题一：
大致出现问题是因为MD5报错找不到：

```
ngx_http_sticky_misc.c: In function 「ngx_http_sticky_misc_md5」:
ngx_http_sticky_misc.c:152:15: ERROR：「MD5_DIGEST_LENGTH」 undeclared (first use in this function)
   u_char hash[MD5_DIGEST_LENGTH];
   
```

解决方式:
    修改在你下载解压缩之后的sticky模块文件夹中的ngx_http_sticky_misc.c文件
将这两个模块 <openssl/sha.h> and <openssl/md5.h>包含到文件ngx_http_sticky_misc.c
下面"+"标注de地方:
```
#include <nginx.h>
#include <ngx_config.h>
#include <ngx_core.h>
#include <ngx_http.h>
#include <ngx_md5.h>
#include <ngx_sha1.h>

+#include <openssl/sha.h>
+#include <openssl/md5.h>

#include "ngx_http_sticky_misc.h"
```

## 问题二：
该问题大致是因为nginx1.9.0以上版本API的改变，
ngx_http_upstream_rr_peer_data_t 中类型定义从
ngx_uint_t 变为 ngx_http_upstream_rr_peer_t*，
导致sticky出现类型不匹配错误，提示如下：
```
/root/nginx-goodies-nginx-sticky-module-ng-1e96371de59f/ngx_http_sticky_module.c: In function ‘ngx_http_get_sticky_peer’: /tmp/nginx-goodies-nginx-sticky-module-ng-1e96371de59f/ngx_http_sticky_module.c:340:21: error: assignment makes pointer from integer without a cast [-Werror] iphp->rrp.current = iphp->selected_peer; ^ cc1: all warnings being treated as errors make[1]: [objs/addon/nginx-goodies-nginx-sticky-module-ng-1e96371de59f/ngx_http_sticky_module.o] Error 1 make[1]: Leaving directory `/tmp/nginx-1.9.0' make: [build] Error 2

```
解决方法：
        修改在你下载解压缩之后的sticky模块文件夹中的ngx_http_sticky_module.c文件，具体如下（“+”标注）：
添加头文件：

```
+#include <nginx.h>
 #include <ngx_config.h>
 #include <ngx_core.h>
 #include <ngx_http.h>
 
 #include "ngx_http_sticky_misc.h"
```
修改code,大概在340行左右：
```

 	/* we have a valid peer, tell the upstream module to use it */
 	if (peer && selected_peer >= 0) {
 		ngx_log_debug(NGX_LOG_DEBUG_HTTP, pc->log, 0, "[sticky/get_sticky_peer] peer found at index %i", selected_peer);
 
+#if defined(nginx_version) && nginx_version >= 1009000
+		iphp->rrp.current = peer;
+#else
 		iphp->rrp.current = iphp->selected_peer;
+#endif		
+		
 		pc->cached = 0;
 		pc->connection = NULL;
 		pc->sockaddr = peer->sockaddr;
 		pc->socklen = peer->socklen;
 		pc->name = &peer->name;
 
 		iphp->rrp.tried[n] |= m;
 
 	} else {
 		ngx_log_debug(NGX_LOG_DEBUG_HTTP, pc->log, 0, "[sticky/get_sticky_peer] no sticky peer selected, switch back to classic rr");
 
 		if (iphp->no_fallback) {
 			ngx_log_error(NGX_LOG_NOTICE, pc->log, 0, "[sticky/get_sticky_peer] No fallback in action !");

```
