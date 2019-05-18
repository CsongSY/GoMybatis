# 高性能 高开发效率 功能全面 的数据库orm框架
* 中文
* [English](README-en.md) 

[![Go Report Card](https://goreportcard.com/badge/github.com/zhuxiujia/GoMybatis)](https://goreportcard.com/report/github.com/zhuxiujia/GoMybatis)
[![Build Status](https://travis-ci.com/zhuxiujia/GoMybatis.svg?branch=master)](https://travis-ci.com/zhuxiujia/GoMybatis)
[![GoDoc](https://godoc.org/github.com/zhuxiujia/GoMybatis?status.svg)](https://godoc.org/github.com/zhuxiujia/GoMybatis)
[![Coverage Status](https://coveralls.io/repos/github/zhuxiujia/GoMybatis/badge.svg?branch=master)](https://coveralls.io/github/zhuxiujia/GoMybatis?branch=master)
[![codecov](https://codecov.io/gh/zhuxiujia/GoMybatis/branch/master/graph/badge.svg)](https://codecov.io/gh/zhuxiujia/GoMybatis)


![Image text](https://zhuxiujia.github.io/gomybatis.io/assets/vuetify.png)
### 使用教程请仔细阅读文档网站 https://zhuxiujia.github.io/gomybatis.io/info.html
# 优势
* <a href="https://zhuxiujia.github.io/gomybatis.io/info.html">高性能</a>，单机每秒事务数最高可达456621Tps/s,总耗时0.22s （测试环境 返回模拟的sql数据，并发1000，总数100000，6核16GB win10）<br>
* <a href="https://zhuxiujia.github.io/gomybatis.io/info.html">事务</a>，session灵活插拔，兼容过渡期微服务<br>
* <a href="https://zhuxiujia.github.io/gomybatis.io/info.html">异步日志</a>异步消息队列日,框架内sql日志使用带缓存的channel实现 消息队列异步记录日志<br>
* <a href="https://zhuxiujia.github.io/gomybatis.io/info.html">动态SQL</a>，在xml中`<select>,<update>,<insert>,<delete>,<trim>,<if>,<set>,<where>,<foreach>,<resultMap>,<bind>,<choose><when><otherwise>,<sql><include>`等等java框架Mybatis包含的15种实用功能<br>
* <a href="https://zhuxiujia.github.io/gomybatis.io/info.html">多数据库</a>Mysql,Postgres,Tidb,SQLite,Oracle....等等更多<br>
* <a href="https://zhuxiujia.github.io/gomybatis.io/info.html">无依赖</a>基于反射动态代理,无需go generate生成*.go等中间代码，xml读取后可直接调用函数<br>
* <a href="https://zhuxiujia.github.io/gomybatis.io/info.html">智能表达式</a>`#{foo.Bar}``#{arg+1}``#{arg*1}``#{arg/1}``#{arg-1}`不但可以处理简单判断和计算任务，同时在取值时 假如foo.Bar 这个属性是指针,那么调用 foo.Bar 则会取指针指向的实际值,完全避免解引用操作<br>
* <a href="https://zhuxiujia.github.io/gomybatis.io/info.html">动态数据源</a>可以使用路由engine.SetDataSourceRouter自定义多数据源规则<br>
* <a href="https://zhuxiujia.github.io/gomybatis.io/info.html">模板标签</a>一行代码实现增删改查，逻辑删除，乐观锁（基于版本号更新）极大减轻CRUD操作的心智负担<br>
* <a href="https://zhuxiujia.github.io/gomybatis.io/info.html">乐观锁</a>`<updateTemplete>`支持通过修改版本号实现的乐观锁<br>
* <a href="https://zhuxiujia.github.io/gomybatis.io/info.html">逻辑删除</a>`<insertTemplete>``<updateTemplete>``<deleteTemplete>``<selectTemplete>`均支持逻辑删除<br>
* <a href="https://zhuxiujia.github.io/gomybatis.io/info.html">8种事务传播行为</a>复刻Spring MVC的事务传播行为功能<br>
* <a href="https://zhuxiujia.github.io/gomybatis.io/info.html">定制easyrpc 基于rpc/jsonrpc</a>让服务完美支持RPC（减少参数限制）,动态代理，事务订阅，易于微服务集成和扩展 详情请点击链接https://github.com/zhuxiujia/easyrpc<br>



## 使用教程

> 示例源码https://github.com/zhuxiujia/GoMybatis/tree/master/example

设置好GoPath,用go get 命令下载GoMybatis和对应的数据库驱动
```
go get github.com/zhuxiujia/GoMybatis
go get github.com/go-sql-driver/mysql
```
实际使用mapper
```
import (
	_ "github.com/go-sql-driver/mysql" //导入mysql驱动
	"GoMybatis"
	"fmt"
	"time"
)

//定义xml内容，建议以*Mapper.xml文件存于项目目录中,在编辑xml时就可享受GoLand等IDE渲染和智能提示。生产环境可以使用statikFS把xml文件打包进程序里

var xmlBytes = []byte(`
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"https://raw.githubusercontent.com/zhuxiujia/GoMybatis/master/mybatis-3-mapper.dtd">
<mapper namespace="ActivityMapperImpl">
    <!--SelectAll(result *[]Activity)error-->
    <select id="selectAll">
        select * from biz_activity where delete_flag=1 order by create_time desc
    </select>
</mapper>
`)

type ExampleActivityMapperImpl struct {
     SelectAll  func() ([]Activity, error)
}

func main() {
    var engine = GoMybatis.GoMybatisEngine{}.New()
	//Mysql链接格式 用户名:密码@(数据库链接地址:端口)/数据库名称,如root:123456@(***.com:3306)/test
	err := engine.Open("mysql", "*?charset=utf8&parseTime=True&loc=Local")
	if err != nil {
	   panic(err)
	}
	var exampleActivityMapperImpl ExampleActivityMapperImpl
	
	//加载xml实现逻辑到ExampleActivityMapperImpl
	engine.WriteMapperPtr(&exampleActivityMapperImpl, xmlBytes)

	//使用mapper
	result, err := exampleActivityMapperImpl.SelectAll(&result)
        if err != nil {
	   panic(err)
	}
	fmt.Println(result)
}
```
## 动态数据源
```
        //添加第二个mysql数据库,请把MysqlUri改成你的第二个数据源链接
	GoMybatis.Open("mysql", MysqlUri)
	//动态数据源路由
	var router = GoMybatis.GoMybatisDataSourceRouter{}.New(func(mapperName string) *string {
		//根据包名路由指向数据源
		if strings.Contains(mapperName, "example.") {
			var url = MysqlUri//第二个mysql数据库,请把MysqlUri改成你的第二个数据源链接
			fmt.Println(url)
			return &url
		}
		return nil
	})
```
## 自定义日志输出
```
	engine.SetLogEnable(true)
	engine.SetLog(&GoMybatis.LogStandard{
		PrintlnFunc: func(messages []byte) {
		},
	})
```
## 异步日志-基于消息队列日志
![Image text](https://zhuxiujia.github.io/gomybatis.io/assets/log_system.png)
## 多数据库支持-驱动列表
```
 Mysql: github.com/go-sql-driver/mysql
 MyMysql: github.com/ziutek/mymysql/godrv
 Postgres: github.com/lib/pq
 Tidb: github.com/pingcap/tidb
 SQLite: github.com/mattn/go-sqlite3
 MsSql: github.com/denisenkom/go-mssqldb
 MsSql: github.com/lunny/godbc
 Oracle: github.com/mattn/go-oci8
 CockroachDB(Postgres): github.com/lib/pq
 ```
 
 ## 嵌套事务-事务传播行为
 <table>
 <thead>
 <tr><th>事务类型</th>
 <th>说明</th>
 </tr>
 </thead>
 <tbody><tr><td>PROPAGATION_REQUIRED</td><td>表示如果当前事务存在，则支持当前事务。否则，会启动一个新的事务。默认事务类型。</td></tr>
 <tr><td>PROPAGATION_SUPPORTS</td><td>表示如果当前事务存在，则支持当前事务，如果当前没有事务，就以非事务方式执行。</td></tr>
 <tr><td>PROPAGATION_MANDATORY</td><td>表示如果当前事务存在，则支持当前事务，如果当前没有事务，则返回事务嵌套错误。</td></tr>
 <tr><td>PROPAGATION_REQUIRES_NEW</td><td>表示新建一个全新Session开启一个全新事务，如果当前存在事务，则把当前事务挂起。</td></tr>
 <tr><td>PROPAGATION_NOT_SUPPORTED</td><td>表示以非事务方式执行操作，如果当前存在事务，则新建一个Session以非事务方式执行操作，把当前事务挂起。</td></tr>
 <tr><td>PROPAGATION_NEVER</td><td>表示以非事务方式执行操作，如果当前存在事务，则返回事务嵌套错误。</td></tr>
 <tr><td>PROPAGATION_NESTED</td><td>表示如果当前事务存在，则在嵌套事务内执行，如嵌套事务回滚，则只会在嵌套事务内回滚，不会影响当前事务。如果当前没有事务，则进行与PROPAGATION_REQUIRED类似的操作。</td></tr>
 <tr><td>PROPAGATION_NOT_REQUIRED</td><td>表示如果当前没有事务，就新建一个事务,否则返回错误。</td></tr></tbody>
 </table>
 
 ```
 //嵌套事务的服务
type TestService struct {
	exampleActivityMapper *ExampleActivityMapper //服务包含一个mapper操作数据库，类似java spring mvc
	UpdateName   func(id string, name string) error   `tx:"" rollback:"error"`
	UpdateRemark func(id string, remark string) error `tx:"" rollback:"error"`
}
func main()  {
	var testService TestService
	testService = TestService{
		exampleActivityMapper: &exampleActivityMapper,
		UpdateRemark: func(id string, remark string) error {
			testService.exampleActivityMapper.SelectByIds([]string{id})
			panic(errors.New("业务异常")) // panic 触发事务回滚策略
			return nil                   // rollback:"error"指定了返回error类型 且不为nil 就会触发事务回滚策略
		},
		UpdateName: func(id string, name string) error {
			testService.exampleActivityMapper.SelectByIds([]string{id})
			return nil
		},
	}
	GoMybatis.AopProxyService(&testService, &engine)//必须使用AOP代理服务的func
	testService.UpdateRemark("1","remark")
}
```
 
 
 
 
 
  ## 内置xml生成工具- 根据用户定义的struct结构体生成对应的 mapper.xml
```
  //step1 定义你的数据库模型,必须包含 json注解（默认为数据库字段）, gm:""注解指定 值是否为 id,version乐观锁,logic逻辑软删除
  type UserAddress struct {
	Id            string `json:"id" gm:"id"`
	UserId        string `json:"user_id"`
	RealName      string `json:"real_name"`
	Phone         string `json:"phone"`
	AddressDetail string `json:"address_detail"`

	Version    int       `json:"version" gm:"version"`
	CreateTime time.Time `json:"create_time"`
	DeleteFlag int       `json:"delete_flag" gm:"logic"`
}

//第二步，在你项目main 目录下建立一个 XmlCreateTool.go 内容如下
func main() {
	var bean = UserAddress{} //此处只是举例，应该替换为你自己的数据库模型
	GoMybatis.OutPutXml(reflect.TypeOf(bean).Name()+"Mapper.xml", GoMybatis.CreateXml("biz_"+GoMybatis.StructToSnakeString(bean), bean))
}

//第三步，执行命令，在当前目录下得到 UserAddressMapper.xml文件
go run XmlCreateTool.go

//以下是自动生成的xml文件内容
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://raw.githubusercontent.com/zhuxiujia/GoMybatis/master/mybatis-3-mapper.dtd">
<mapper>
    <!--logic_enable 逻辑删除字段-->
    <!--logic_deleted 逻辑删除已删除字段-->
    <!--logic_undelete 逻辑删除 未删除字段-->
    <!--version_enable 乐观锁版本字段,支持int,int8,int16,int32,int64-->
    <resultMap id="BaseResultMap" tables="biz_user_address">
    <id column="id" property="id"/>
	<result column="id" property="id" langType="string"   />
	<result column="user_id" property="user_id" langType="string"   />
	<result column="real_name" property="real_name" langType="string"   />
	<result column="phone" property="phone" langType="string"   />
	<result column="address_detail" property="address_detail" langType="string"   />
	<result column="version" property="version" langType="int" version_enable="true"  />
	<result column="create_time" property="create_time" langType="Time"   />
	<result column="delete_flag" property="delete_flag" langType="int"  logic_enable="true" logic_undelete="1" logic_deleted="0" />
    </resultMap>
</mapper>
```
 
 
 
 
 
 


## 配套生态(RPC,JSONRPC,Consul)-搭配GoMybatis
* https://github.com/zhuxiujia/easy_mvc //mvc,极大简化开发流程
* https://github.com/zhuxiujia/easyrpc  //easyrpc（基于标准库的RPC）吸收GoMybatis的概念，类似标准库的api，定义服务没有标准库的要求那么严格（可选不传参数，或者只有一个参数，只有一个返回值）
* https://github.com/zhuxiujia/easyrpc_discovery  //基于easyrpc定制微服务发现，支持动态代理，支持GoMybatis事务，AOP代理，事务嵌套，tag定义事务，自带负载均衡算法（随机，加权轮询，源地址哈希法）
![Image text](https://zhuxiujia.github.io/gomybatis.io/assets/easy_consul.png)










## 请及时关注版本，及时升级版本(新的功能，bug修复)
## TODO 未来新特性（可能会更改）

## 喜欢的老铁欢迎在右上角点下 star 关注和支持我们哈


