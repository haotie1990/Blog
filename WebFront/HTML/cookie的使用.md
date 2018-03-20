# cookie的使用

### 什么是Cookie？

``HTTP Cookie``是服务器发送到用户浏览器并保存在本地的一小块数据，它会在浏览器下次向服务器再次发起请求时被携带并发送到服务器上。通常，它用于告知服务端两个请求是否来自同一浏览器，如保持用户的登录状态。``Cookie``使基于无状态和HTTP协议记录稳定的状态信息成为了可能。

**Cookie主要用于以下三个方面：**

* 会话状态管理（如用户登录状态，购物车等其他需要记录的信息）
* 个性化设置（如用户自定义的设置、主题等）
* 浏览器行为追踪（如跟踪分析用户行为等）

### Cookie的属性？

* name：cookie的名称，一经创建便不可更改
* value：cookie的值，如果值为unicode字符，需要为字符编码，如果是二进制数据，则需要使用base64编码
* domain：指定哪些主机可以接受``cookie``。如果不指定，则默认为当前页面``hostname``。非顶级域名只能设置``domain``为顶级域名或其本身，不能设置其他非顶级域名。顶级域名只能设置``domain``为顶级域名，不能设置为非顶级域名（二级或三级域名）。而非顶级域名可以读取``domain``为顶级域名或其自身的``cookie``，因此要在多个域名中共享，则设置``domain``为顶级域名。
* path：指定服务器主机下哪些路径可以接受``cookie``。默认是网页根目录``/``
* expires/max-age：``expires``设置``cookie``什么时间失效。``expires``即是``cookie``的失效日期。``expires``必须是``GMT``格式的时间，如``expires=Thu, 25 Feb 2016 04:18:00 GMT``表示``cookie``在2016年2月25日4:18分之后失效，对于失效的``cookie``浏览器会清空。``max-age``是以秒为单位的时间段，因此``cookie失效时刻=创建时间+max-age``。若不设置``expires/max-age``，则``cookie``为会话``cookie``，其在浏览器窗口关闭后即被删除。``max-age``的默认值是-1(即有效期为``session``)，其值有三种：负数、0、正数。负数有效其为``session``，0：删除``cookie``，正数：有效期为创建时刻+max-age
* size：``cookie``大小
* http：``cookie``的``httpOnly``属性，正常情况下客户端可以通过``document.cookie``访问浏览器的``cookie``，若设置``httpOnly``属性， 则客户端无法通过``document.cookie``获取``cookie``（包括修改，删除）。
* secure：若设置了``secure``，则此``cookie``只能通过https协议发到服务器

### 客户端（浏览器）如何设置、删除、更新cookie

``cookie``既可以通过客户端通过``document.cookie``设置也可以有服务器通过``set-cookie``设置。

* ``document.cookie``和``set-cookie``每次只能设置一个``cookie``
* 客户端``document.cookie``不能设置``httpOnly``，而服务器``set-cookie``可以设置``cookie``的所有属性

**设置``cookie``**

通过``document.cookie``设置

**修改``cookie``**

修改一个``cookie``只需要重新赋值即可，d但需要注意的是，必须保证``domain/path``是一致的，否则只会生成一个新的``cookie``

**删除 ``cookie``**

删除一个``cookie``只要设置其有效期为过去的时间点或设置``max-age``为0。

### 跨越请求如何携带cookie？

跨域请求是默认不携带``cookie``信息的。如果要把``cookie``发到服务器，一方面必须服务器同意，即指定``Access-Control-Allow-Credentials``是``true``。此外还需要客户端发发送ajax请求的时候打开``withCredentials``。

同时``Access-Control-Allow-Origin``就不能设置为星号，必须指定明确的、与请求网页一致的域名。同时，``cookie``也同样遵循同源策略。

### Cookie的安全问题？

### 参考

* [聊一聊 cookie](https://segmentfault.com/a/1190000004556040)
* [cookie小结](http://www.cnblogs.com/xianyulaodi/p/6476991.html)
* [HTTP cookies](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies)