#前后端分离开发总结

## * RESTful API定义不规范
![不规范的例子](https://github.com/Summer-Zhang/RESTful/blob/master/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-07-07%20%E4%B8%8B%E5%8D%8812.46.42.png)

### 1. 什么是REST（Representational State Transfer）
REST的名称"表现层状态转化"，省略了主语。"表现层"其实指的是"资源"（Resources）的"表现层"。

	* Resource
		网络上的一个具体信息。它可以是一段文本、一张图片、一首歌曲、一种服务，总之就是一个具体的实在。你可以用一个URI（统一资源定位符）指向它，每种资源对应一个特定的URI。
	* Representation
		"资源"是一种信息实体，它可以有多种外在表现形式, 文本可以用txt格式表现，也可以用HTML格式、XML格式、JSON格式表现，甚至可以采用二进制格式；图片可以用JPG格式表现，也可以用PNG格式表现。
	* State Transfer
		互联网通信协议HTTP协议，是一个无状态协议。由客户端发出请求到服务端，最终返回自己想要的数据，当前请求不会受到上次请求的影响。
	* HTTP 动作
		HTTP协议里面，四个表示操作方式的动词：GET、POST、PUT、DELETE。它们分别对应四种基本操作：GET用来获取资源，POST用来新建资源（也可以用于更新资源），PUT用来更新资源，DELETE用来删除资源。
**综述：  
 什么是RESTful架构：** 

	1. 每一个URI代表一种资源； 
	2. 客户端和服务器之间，传递这种资源的某种表现层；
	3. 客户端通过HTTP动词，对服务器端资源进行操作，实现"表现层状态转化"。

**RESTful API 例子：** 

	GET /zoos：列出所有动物园
	POST /zoos：新建一个动物园
	GET /zoos/ID：获取某个指定动物园的信息
	PUT /zoos/ID：更新某个指定动物园的信息（提供该动物园的全部信息）
	PATCH /zoos/ID：更新某个指定动物园的信息（提供该动物园的部分信息）
	DELETE /zoos/ID：删除某个动物园
	GET /zoos/ID/animals：列出某个指定动物园的所有动物
	DELETE /zoos/ID/animals/ID：删除某个指定动物园的指定动物

------

![不规范的例子](https://github.com/Summer-Zhang/RESTful/blob/master/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-07-07%20%E4%B8%8B%E5%8D%8812.48.26.png)

### 2. RESTful API设计原则
*  **协议**  

	API与用户的通信协议，总是使用HTTPs协议。
*  **域名** 

	应该尽量将API部署在专用域名之下。
	` https://api.example.com `
	
	如果确定API很简单，不会有进一步扩展，可以考虑放在主域名下。
	` https://example.org/api/ `
	
*  **版本(Versioning)**  
	应该将API的版本号放入URL。  
	` https://api.example.com/v1/ `  
	也可以将版本号放在HTTP头信息中：
	`　Accept: vnd.example-com.foo+json; version=1.0 `
*  **路径(Endpoint)**  
	在RESTful架构中，每个网址代表一种资源（resource），所以网址中不能有动词，只能有名词，而且所用的名词往往与数据库的表格名对应。一般来说，数据库中的表都是同种记录的"集合"（collection），所以API中的名词也应该使用复数。

*  **HTTP动词**

	```
	GET（SELECT）：从服务器取出资源（一项或多项）。  
	POST（CREATE）：在服务器新建一个资源。  
	PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）。  
	PATCH（UPDATE）：在服务器更新资源（客户端提供改变的属性）。  
	DELETE（DELETE）：从服务器删除资源。  
	```
	
*  **过滤信息（Filtering）**

	API应该提供参数，过滤返回结果:
	
	```
	?limit=10：指定返回记录的数量
	?offset=10：指定返回记录的开始位置。
	?page=2&per_page=100：指定第几页，以及每页的记录数。
	?sortby=name&order=asc：指定返回结果按照哪个属性排序，以及排序顺序。
	?animal_type_id=1：指定筛选条件
	```
*  **状态码（Status Codes）**

	服务器向用户返回的状态码和提示信息，常见的有以下一些（方括号中是该状态码对应的HTTP动词）.

	```
	200 OK - [GET]：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。
	201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功。
	202 Accepted - [*]：表示一个请求已经进入后台排队（异步任务）
	204 NO CONTENT - [DELETE]：用户删除数据成功。
	400 INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。
	401 Unauthorized - [*]：表示用户没有权限（令牌、用户名、密码错误）。
	403 Forbidden - [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的。
	404 NOT FOUND - [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
	406 Not Acceptable - [GET]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。
	410 Gone -[GET]：用户请求的资源被永久删除，且不会再得到的。
	422 Unprocesable entity - [POST/PUT/PATCH] 当创建一个对象时，发生一个验证错误。
	500 INTERNAL SERVER ERROR - [*]：服务器发生错误，用户将无法判断发出的请求是否成功。
	```
*  **错误处理（Error handling）**

返回的信息中将error作为键名，出错信息作为键值即可。

```
{
    error: "Invalid API key"
}
```
*  **返回结果**

针对不同操作，服务器向用户返回的结果应该符合以下规范。

```
GET /collection：返回资源对象的列表（数组）
GET /collection/resource：返回单个资源对象
POST /collection：返回新生成的资源对象
PUT /collection/resource：返回完整的资源对象
PATCH /collection/resource：返回完整的资源对象
DELETE /collection/resource：返回一个空文档
```

	
## * 已定义的API,字段改动随意
**就是一开始定API时没考虑清楚，后面做起来的时候没依据API改的随意了。 这是病得治！！！**
## * 参考文档
[Principles of good RESTful API Design](https://codeplanet.io/principles-good-restful-api-design/)  
[Versioning REST Services](http://www.informit.com/articles/article.aspx?p=1566460)  
[前后端分离-Rest Api设计](http://bbear.me/qian-hou-duan-fen-chi-tr/)  
[RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)  
