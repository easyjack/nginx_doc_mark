1. proxy_bind -> 与upstream通讯中所使用的IP地址

2. proxy_buffer_size -> 设置upstream的first response buffer size，取值建议与response header相关。一般来说与一内存分页值相等(即 4k 或者 8k)

3. proxy_buffering -> ( on / off )  ->  缓存开关

4. proxy_buffers -> 为每个连接设置的缓冲区数量以及缓冲区大小

5. proxy_busy_buffer_size -> 在开启缓冲后，设置buffers阈值以一边缓冲一边response to client【默认是proxy_buffer_size 和 proxy_buffers的两倍】

6. proxy_cache 定义用于页面的缓存共享内存

7. proxy_cache_background_update -> (v1.11.10+ 可用)设置是否允许开启一个后台请求以更新过期的缓存，配合 proxy_cache_use_stale 指令使用

8. proxy_cache_bypass -> 设置什么情况下不从 cache 中读取响应，可以与 proxy_no_cache 指令配合使用,当值中的任何一个或一个以上的参数不为空且不为0，那么该response不从 cache 中读取响应

9. proxy_cache_convert_head -> 设置对于HEAD方法的请求是否转换为GET请求，当disabled的时候与 proxy_cache_key 指令有关联

10. proxy_cache_key -> 定义如何缓存的键所包含的内容,一般该值类似为 $scheme$proxy_host$uri$is_args$args

11. proxy_cache_lock -> ( on / off ) ( v1.1.12+可用 )，设置同样的请求(判断请求是否同样与proxy_cache_key有关)在同样的时间是否发起多个代理请求，开启后与proxy_cache_lock_timeout 指令有关联

12. proxy_cache_lock_age -> 代理请求重发时间

13. proxy_cache_lock_timeout -> (v1.1.12+), 为proxy_cache_lock指令设置超时，超时后发起代理请求，但是该请求不会被缓存

14. proxy_cache_max_range_offset -> 仿佛是设置超过某个字节的请求会直接发到代理服务器并且不会被缓存

15. proxy_cache_methods -> 设置什么请求将会被缓存，与 proxy_no_cache 指令有关

16. proxy_cache_min_users -> 设置请求达到多少次后会被缓存

17. proxy_cache_path -> 为缓存设置目录路径以及其他参数, 缓存数据将会以文件的形式存储下来。文件名就是 MD5(proxy_cache_key)的结果。

其中level 参数决定了缓存的等级层次，取值是1-3，每个 levels 还可以接受值1或2。也就是levels 参数的取值形式是1:2 。一个缓存response会先被写入一个临时文件，然后该临时文件会被重命名。
从0.8.9版本开始，临时文件和缓存文件可以放置于不同的文件系统。然而，需要注意的是，放置于不同的文件系统可能会因为文件复制带来效率损耗问题。因此建议把放置缓存和临时文件的目录放在同样的文件系统上。
设置缓存文件文件夹的参数是 use_temp_path (v1.7.10+)。如果 use_temp_path 参数省略或者设置为 on ,那么临时文件的文件夹路径就会被 proxy_temp_path 指令设置。如果 use_temp_path的值是 off，那么临时文件就会被直接放置于与cache同一个文件夹下。
另外，所有可用的键值及缓存相关数据信息都会被存储在由key_zone参数及其name 和 size 值设置的共享内存中，1MB的空间大约可以存放8千个键。
缓存数据在参数 inactive 指定的时间内未被访问的话会被删除，不管它是否刚生成的。默认情况下， inactive的值是10分钟。

特殊的"cache manager"进程负责监视由 max_size 参数设置的最大cache值，当超过最大cache值时，他会删除最少使用的数据，数据配合 manager_files 和 manager_sleep 参数设置在迭代中被删除。在一次迭代删除中，最多 manager_files 个文件被删除(默认100)。每次迭代的持续时间受 manager_threshold 参数限制(默认 200 毫秒)。迭代之间的暂停时间由 manager_sleep 参数设置(默认50毫秒)。

在启动一分钟后，"cache loader"进程会被激活，他将之前存储在文件系统上的缓存数据信息加载到缓存区(cache zone)中。这个加载过程也是以迭代的方式完成的。一次迭代最多加载 loader_files 个项目(默认100)。此外，每次迭代的持续时间由 loader_threshold 参数限制(默认200毫秒)。迭代之间的暂停时间由 loader_sleep参数限制(默认50毫秒)。


18. proxy_cache_purge -> 定义触发清除缓存操作的特殊请求。如果被该参数定义的值其中一个不为空而且不为0，则对应的缓存会被删除，然后返回状态码204(无内容)。【官方文档此处有example configuration】(通过map定义变量) -- 需要重新编译安装模块才可以使用

19. proxy_cache_revalidate  on | off  ->  是否通过使用"If-Modified-Since" 和 "If-None-Match" header重新验证过期缓存

20. proxy_cache_use_stale -> 定义什么情况下可以使用过期的缓存。该指令参数与 proxy_next_upstream 指令的参数相匹配,设置 error 参数可在无法选择代理服务器来处理请求时使用旧的缓存响应。另外，设置 updating 参数允许在正在更新缓存的时候使用旧的缓存响应，这样可以在更新缓存数据的时候，将访问源服务器的次数最小化。
也可以通过在response header中使用设置header定义响应过期后指定秒数以直接使用旧的缓存回应，这种做法优先级比使用指令参数优先级低。
如：
○ The “stale-while-revalidate” extension of the “Cache-Control” header field permits using a stale cached response if it is currently being updated.
○ The “stale-if-error” extension of the “Cache-Control” header field permits using a stale cached response in case of an error.
使用proxy_cache_lock 指令可以使在生成新的缓存的时候将访问源服务器次数降到最低


21. proxy_cache_valid ->为不同的响应状态码设置不同的缓存时间，如
proxy_cache_valid 200 302 10m;
proxy_cache_valid 404 1m;
如果只设置时间参数 proxy_cache_valid 5m; 则只有200, 301 和 302 报文会被缓存
另外，设置 any 参数可用户缓存一切报文 proxy_cache_valid any 1m; 缓存参数同样可以直接通过 response header设置，通过 response header 设置的规则比使用指令设置的规则优先级高。
response header settings : 
- "X-Accel-Expires": 设置缓存的有效时间(以秒为单位),当设置为 0 的时候意味着不缓存。如果数值以 @ 开头，则为自纪元以来的绝对时间(以秒为单位),直到设置的时间为止，该响应都可以被缓存且缓存有效。
- 如果不包含 " X-Accel-Expires "字段，缓存参数有可能被 "Expires" 或 " Cache-Control"字段设置
- 包含 "Set-Cookie" 字段的报文不会被缓存
- 包含 "Vary" 字段且值为"*"的报文不会被缓存。如果 "Vary"字段的值是其他值，则这样的响应将考虑相应的请求header字段进行缓存
- 通过 proxy_ignore_header 指令可以忽略对这些header的处理


22. proxy_connect_timeout -> 定义与源服务器建立连接的最长时间，注意一般这个值不超过75秒

23. proxy_cookie_domain -> 对源服务器response Set-cookie 字段中的域名进行替换，大小写不敏感。在domain 参数或 replacement 参数以及domain 属性开头的 . 会被忽略。domain 参数 和 replacement 参数可以引入变量。指令参数可以使用正则表达式，通过 ~ 符号开头。正则表达式内容中可以包含命名以及位置捕获并在 replacement 参数中引用它们。示例：
proxy_cookie_domain localhost example.org;
proxy_cookie_domain ~\.([a-z]+\.[a-z]+)$ $1;
如果参数值为 off 则令当前所有 proxy_cookie_domain 指令不生效

24. proxy_cookie_path -> 对  Set-cookie 中的 path 值进行修改……同样off参数令所有 proxy_cookie_path 不生效

25. proxy_force_ranges -> 是否启用字节范围支持(与 header Accept-Rangs有关)

26. proxy_headers_hash_bucket_size -> 设置proxy_hide_header 和 proxt_set_headers指令所用的hash表的最大值

27. proxy_hide_header -> nginx 默认不传递源服务器的 "Date"，"Server", "X-Pad", "X-Accel-xxx" 等header到客户端。通过本指令设置更多的不被传递的字段。通过proxy_pass_header去额外设置需要传递到客户端的字段值。

28. proxy_http_version -> 设置与源服务器通讯所使用的 http协议版本，默认使用1.0版本。对于keepalive和 NTLM authentication 的连接建议使用1.1版本。

29. proxy_ignore_client_abort -> 定义当client在没有收到response便关闭了连接的时候，proxied server是否也需要关闭与源服务器之间的连接

30. proxy_ignore_headers -> 对某些特殊response header不进行处理。以下headers可以被忽略处理:
"X-Accel-Redirect", "X-Accel-Limit-Rate", "X-Accel-Buffering", "X-Accel-Charset", "Expires", "Cache-Control", "Set-Cookie", "Vary"

默认这些headers会有对应的特殊处理:
- "X-Accel-Expires", "Expires", "Cache-Control", "Set-Cookie" 和 "Vary" -> 响应缓存参数
- "X-Accel-Redirect" 指定的URI内部重定向
- "X-Accel-Limit-Rate" 设置向客户端传输response的速率限制
- "X-Accel-Buffering" 启用或者禁用响应缓存
- "X-Accel-Charset" 设置所需的响应字符集


31. proxy_intercept_errors -> on/off(default) => 定义当源服务器返回HTTP状态码为300或大于300的时候是否直接返回给client还是拦截并由nginx所定义的error_page指令处理

32. proxy_limit_rate -> 限制从源服务器接受报文的速率。参数值以字节每秒为单位。当参数值为0的时候则禁用速率限制(默认)。该限制是对每个请求设置的，所以假设nginx同时向源服务器发起两个连接，则总速率将达到限制速率的两倍。此限制只有在proxy_buffering 启用的时候有效

33. proxy_max_temp_file_size -> 当proxy_buffering 为 on 即开启缓存后，若 response 不能存放进 proxy_buffer_size 和 proxy_buffers 所设置的缓冲区，该response的部分数据将会被存到临时文件中。该指令设置临时文件的最大容量(默认1024m)，而每次写入临时文件中的数据量大小则由 proxy_temp_file_write_size 指令设置。当设置本指令值为0时意味着不缓存response到临时文件。【这个限制不适用于被缓存或存储到硬盘上的response】

34. proxy_method -> 指定与源服务器请求时所使用的 HTTP 方法，而不是直接使用客户端请求的方法。值可包含变量

35. prxy_next_upstream -> 定义在什么失败情况下请求应该被发送到下一个源服务器(默认: error timeout),需要记住的是，只有在还没有发送任何内容到客户端的情况下，才有可能将请求发送到下一个源服务器。即在响应传输的过程中，如果发生了错误或者超时，该问题无法解决。这个指令还定义了在什么情况下与服务器的尝试通讯是不成功的(http/ngx_http_upstream_module.html#max_fails)。连接错误，超时以及无效的header被定义为不成功的尝试通讯，即使这几种情况没有在指令中定义。

36. proxy_next_upstream_timeout -> 设置请求发送到下一个源服务器的超时时间。0则无限制

37. proxy_next_upstream_tries -> 设置尝试将请求转发到下一个源服务器的次数。0为不限制

38. proxy_no_cache -> 定义什么条件下的 response 不会被缓存。当值中的任何一个或一个以上的参数不为空且不为0，那么该response不会被缓存，可以与 proxy_cache_bypass 指令配合一同使用






不同的nginx变量

$request_method

$schema

$proxy_host

$request_uri

$host

$cookie_user
