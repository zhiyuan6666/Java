# 什么是框架?
> 　　框架就是软件半成品，使用的时候根据需求进行填空　
　　需要建立特定位置和名称的配置文件,在文件中添加变量
　　使用了xml解析和反射的技术
  
##### 1.MyBatis是数据访问层的框架(底层是对JDBC的封装)
##### 2.mybatis.xml文件
> 帮助文档中的简单示例

    	<?xml version="1.0" encoding="UTF-8" ?>
    	<!-- 加载DTD-->
    	<!DOCTYPE configuration
    	  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
    	  "http://mybatis.org/dtd/mybatis-3-config.dtd">
    	<configuration>
    		<!-- default属性表示引用的环境-->
    		<environments default="development">
    		<!-- 可以有多个environment标签 -->
    		<environment id="development">
				<!-- 使用原生JDBC事务 -->
				<transactionManager type="JDBC"></transactionManager>
				<!-- 数据库连接池技术 -->
				<dataSource type="POOLED">
					<!-- 连接JDBC的属性 -->
					<property name="driver" value="${driver}"></property>
					<property name="url" value="${url}"></property>
					<property name="username" value="${username}"></property>
					<property name="password" value="${password}"></property>
				</dataSource>
    		</environment>
    	  </environments>
    	  <!-- 加载mapper.xml文件 -->
    	  <mappers>
    		<mapper resource="org/mybatis/example/BlogMapper.xml"></mapper>
    	  </mappers> 
    	</configuration>
> 测试类中

    	String resource = "org/mybatis/example/mybatis-config.xml";
    	// 加载配置文件
    	InputStream inputStream = Resources.getResourceAsStream(resource);
    	// 通过SqlSessionFactoryBuilder创建一个工厂
    	sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
    	// 通过sqlsessionfactor工厂 创建sqlSession
    	// sqlsession相当于一次数据库会话，打开连接，执行sql，关闭链接
    	
    	SqlSession session = sqlSessionFactory.openSession();
    	try {
    		// 查询 并返回结果							方法名,								参数
    	  Blog blog = (Blog) session.selectOne("org.mybatis.example.BlogMapper.selectBlog", 101);
    	} finally {
    		// 释放资源
    	  session.close();
    	}
    
    	mapper.xml文件配置
    		<?xml version="1.0" encoding="UTF-8"?>
    		<!DOCTYPE mapper
    		PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    		"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    		
    		<!-- namesapce:理解成实现类的全路径(包名+类名) -->
    		<mapper namespace="com.wzy.mapper.PojoMapper" >
    			<!-- id:方法名 parameterType:定义参数类型 resultType:返回值类型.
    			如果方法返回值是list,在resultType中写List的泛型, 因为mybatis
    			对jdbc封装,一行一行读取数据
    			-->
    			<select id="selAll"
    			resultType="com.wzy.pojo.Pojo"> select * from Pojo
    			</select>
    		</mapper>

##### 3.log4j支持
	在mybatis.xml文件中配置
	注意：在src下要有log4j.properties文件
	注意 setings标签的位置(dtd的语法要求)
	<settings>
		<setting name="logImpl" value="LOG4J"/> 
	</settings>
##### 4.别名
> 在mybatis.xml文件中配置

	<typeAliases> 
		<package name="com.wzy.pojo" /> 
	</typeAliases>
	引用的时候直接写类名就行 不用写全路径
	
##### 5.parameterType设置参数类型

##### 6.resultType设置返回类型

##### 7.Mybatis接口绑定 使用了动态代理和反射
	创建接口
		接口包名和接口名要与mapper.xml的namespace一致
		
	在 mybatis.xml <mappers>标签中使用<package>进行扫描接口和 mapper.xml
	<mappers>
		<package name="com.wzy.mapper"/> 
	</mappers>
	// 此时<mappers>中就不需要引入
	<mapper resource="org/mybatis/example/BlogMapper.xml"/>
	
	实现了接口绑定之后，就可以进行多参数传递了
	进行多参数传递时，parameterType就可以不用写	
	在参数前加@param("key1")
		例: selByParam1Param2(@param("key1")String string1,@param("key2")String string2);
		然后在mapper.xml文件中
		<select id="" resultType="pojo">
			select * from table
			where param1=#{key1} and param2=#{key2}
		</select>
		如果没有添加@param("key1") 这个注解在mapper.xml中
		只能是
		<select id="" resultType="pojo">
			select * from table
			where param1=#{0} and param2=#{1}
		</select>
		或者
		<select id="" resultType="pojo">
			select * from table
			where param1=#{param1} and param2=#{param2}
		</select>
		原理:Mybatis底层把参数转换成了map了 
##### 8.动态SQL
	<if test="">
		and XXX
	</if>
	注意：where后面记得接1==1 因为可能每个if都不成立
		<select id="selByAccInAccOut" resultType="Log">
			select * from log where 1=1
			// OGNL表达式，直接写key或者对象的属性 ,不需要添加任何特定字符串符号
			<if test="accIn!=null and accIn!='' "> 
			// &为关键字 xml文件不允许写& 因此使用and  再编译得时候会将and 转换为&符号
				and accIn=#{accIn}
			</if>
			<if test="accOut!=null and accOut!=''">
				and accOut=#{accOut}
			</if>
		</select>
		
	<where>
		<if test="">
			and XXX
		</if>
		<if test="">
			and XXX
		</if>
	</where>
	当内容中第一个是and mybatis会自动去掉第一个and
		<select id="selByAccInAccOut" resultType="Log">
			select * from log 
			<where>
				// OGNL表达式，直接写key或者对象的属性 ,不需要添加任何特定字符串符号
				<if test="accIn!=null and accIn!='' "> 
				// &为关键字 xml文件不允许写& 因此使用and  再编译得时候会将and 转换为&符号
					and accIn=#{accIn}
				</if>
				<if test="accOut!=null and accOut!=''">
					and accOut=#{accOut}
				</if>
			</where>
		</select>
		
		
	<where> <choose> <when> <otherwise>	
	<where>
		<choose>
			<when test="">
				and 
			</when>
		</choose>
	</where>
	
	注意：只有一个条件成立
		where标签可以去掉前面and
		<select id="selByAccInAccOut" resultType="Log">
			select * from log 
			<where>
				<choose>
					<when test="accOut!=null and accOut!=''">and accOut=#{accOut}</when>
				</choose>
			</where>
		</select>
		
		
	<set>
		id=#{id},
		<if>
			XXX,
		</if>
		<if>
			XXX,
		</if>
	</set>
	注意：set标签会去掉最后一个逗号 ","
	使用id=#{id},   的目的是防止set中没有内容 sql语句会出错
	当set中有内容的时候，会生成set关键字，没有就不生成
		<update id="updLog" parameterType="Log">
			update Log 
			<set>
				id=#{id},
				<if test="accIn!=null and accIn!='' ">
				accIn=#{accIn},
				</if>
				<if test="accOut!=null and accOut!='' ">
				accIn=#{accOut},
				</if>
			</set>
			where id=#{id}
		</update>
		<select id="selByLog" parameterType="Log" resultType="Log">
			select * from Log
			<trim prefix="where" prefixOverrides="and">
				and accIn=#{accIn}
			</trim>
		</select>
		
	<trim></trim>
		prefix 在前面添加内容
		prefixOverrides 去掉前面内容
		suffix 在后面添加内容
		suffixOverrieds 去掉后面内容
		执行顺序去掉内容后添加内容
		<update id="upd" parameterType="log"> 
			update log 
			<trim prefix="set" suffixOverrides=",">
				a=a, 
			</trim> 
			where id=100 
		</update>
		
	<bind/>
	作用:给参数重新赋值
		模糊查询
		在原内容前或后添加内容
		<select id="selByLog" parameterType="log"  resultType="log"> 
			// 相当于在accin前面和后面加上 %   注意:是字符串的拼接
			<bind name="accin" value="'%'+accin+'%'"/>#{money} 
		</select>
		
	<foreach></foreach>
	注意：效率低
		collectino=”” 要遍历的集合
		item 迭代变量,#{迭代变量名}获取内容
		open 循环后左侧添加的内容
		close 循环后右侧添加的内容
		separator 每次循环时,元素之间的分隔符
		<select id="selIn" parameterType="list" resultType="log"> 
			select * from log where id in 
			<foreach collection="list" item="abc" open="(" close=")" separator=","> 
				#{abc} 
			</foreach> 
		</select>

##### 10.缓存

	应用程序和数据库交互过程是一个相对比较耗时的过程
	让应用程序减少对数据的访问，提升程序运行效率
	mybatis 默认sqlsession缓存开启
		1.同一个sqlsession对象调用同一个<select>时，只有第一次访问数据库，第一次之后把查询结果缓存到sqlsession的缓存区(内存)中
		2.缓存的是statement对象
			一个<select>对应一个statement对象
		3.必须是同一个sqlsession对象
	缓存的流程
		先去缓存区找是否存在statement
		不存在去数据库获取数据，将数据放到缓存区中
		存在返回数据即可
	sqlsessionFactory缓存(二级缓存)
		有效范围：同一个factory内的sqlsession
		当数据频繁被使用，很少被修改
		在mapper.xml中添加
			<cache readOnly="true" />
		当close()或commit()是会把sqlsession缓存刷到sqlsessionfactor缓存区中

##### 11.mybatis实现多表查询  2X3=6种方式
	多表查询的方式
		1.业务装配：对两个表写单表查询语句，然后在service中把查询的结果进行关联.
		2.使用AutoMapping特性，在实现两表联合查询时通过别名来完成映射.
		3.使用mybatis<resultMap>属性进行实现.
	
	多表查询时，类中包含另一个类的对象
		1.单个对象
		2.集合对象
	数据库的设计
		student: id  name age tid
		teacher: id  name
##### 12.resultMap属性
	1.编写在mapper.xml中，由程序员控制SQL查询结果与实体类的映射关系
		使用AutoMapping(自动映射)特性
	2.使用resultMap属性的时候,select标签中就不用写resultType属性，而是使用resultMap属性引用resultMap标签
		手动映射
		
##### 13.使用resultMap实现单表映射关系(表字段与pojo类属性 名称不一样)
	数据库设计 id name
	pojo类属性 id1 name1
	
	在mapper.xml进行映射
	<resultMap type="teacher" id="mymap">
		<!-- 主键使用id标签配置映射关系 -->
		<id column="id" property="id1" />
		<!-- 其他列使用result标签配置映射关系 -->
		<result column="name" property="name1"/>
	</resultMap>

	<select id="selAll" resultMap="mymap">
		select * from teacher
	</select>

##### 14.类中包含一个其他对象 (N+1方式)例如(在查询学生的时候查询出老师-->即在学生对象中包含一个老师对象)
	1.N+1方式 先查询出某个表的全部信息,再根据这个表的信息查出另一个表的全部信息.
		使用association属性
	<resultMap type="student" id="stuMap">
		<id column="id" property="id"  />
		<result column="name" property="name"/> 
		<result column="age" property="age"/>
		<result column="tid" property="tid"/>
		<!-- 如果关联一个对象 -->
		<!-- 在teachermapper.xml写一个根据教师selById查询的方法  -->
		<!-- 将selById方法(参数为tid)查询的结果映射到pojo类的teacher属性中 -->
		<association property="teacher" select="com.wzy.mapper.TeacherMapper.selById" column="tid"></association>
	</resultMap>
	
	<!-- 查询全部学生 -->
	<select id="selAll" resultMap="stuMap">
		select * from student
	</select>

	2.联合查询方式
		只需要写一个sql查询
		把<association>当作一个小的resultMap来看待
		javaType 属性:<association/>专配完后返回一个什么类型的对象.取值是一个类(或类的别名) 
	
	<resultMap type="Student" id="stuMap1"> 
		<id column="sid" property="id"/> 
		<result column="sname" property="name"/> 
		<result column="age" property="age"/> 
		<result column="tid" property="tid"/> 
		
		<association property="teacher" javaType="Teacher" > 
			<id column="tid" property="id"/> 
			<result column="tname" property="name"/> 
		</association> 
	</resultMap> 
	
	<select id="selAll1" resultMap="stuMap1"> 
			select s.id sid,s.name sname,age age,t.id tid,t.name tname FROM student s left outer join teacher t on s.tid=t.id 
	</select>

##### 15.使用<resultMap>查询关联集合对象
	1.N+1方式
	TeacherMapper.xml中
		<resultMap type="teacher" id="mymap"> 
			<id column="id" property="id"/> 
			<result column="name" property="name"/> 
			<!--集合使用collection属性 -->
			<collection property="list" select="com.wzy.mapper.StudentMapper.selByTid" column="id">
			</collection> 
		</resultMap> 
		
		<select id="selAll" resultMap="mymap"> 
			select * from teacher 
		</select>
	
	StudentMapper.xml中
		<select id="selByTid" parameterType="int" resultType="student"> 
			select * from student where tid=#{0} 
		</select>
	
	2.联合查询方式
		<resultMap type="teacher" id="mymap1"> 
			<id column="tid" property="id"/> 
			<result column="tname" property="name"/>
			<collection property="list" ofType="student" > 
				<id column="sid" property="id"/> 
				<result column="sname" property="name"/> 
				<result column="age" property="age"/> 
				<result column="tid" property="tid"/> 
			</collection> 
		</resultMap> 
		<select id="selAll" resultMap="mymap1"> 
			select t.id tid,t.name tname,s.id sid,s.name sname,age,tid from teacher t LEFT JOIN student s on t.id=s.tid; 
		</select>

##### 16.使用AutoMapping(自动映射--查询出的列名和属性名要相同)结合别名实现多表查询.
	注意：
		1. 只适用 类中包含一个其他对象(不是集合)
		2. 字符点. 在sql中是关键字，要是有反单引号(`)(tab按键上面的)
	<select id="selAll" resultType="student">
		select t.id `teacher.id`,t.name `teacher.name`,s.id id,s.name name,age,tid from student s LEFT JOIN teacher t on t.id=s.tid 
	</select>

##### 17.mybatis注解
	目的：简化mapper.xml配置文件
	注意：
		设计动态sql需要使用mapper.xml文件
	
	1.单表的增删改查
	@Select("select * from student")
	List<Student> selAll();
		
	@Update(update studen set name=#{name},age=#{age}where id=#{id})
	int updById(Student s);
	
	@Delete("delete from student where id=#{0}")
	int delById(int id);

	@Insert("insert into student values(default,#{name},#{age})")
	int insStudent(Student s);
	
	2.多表的增删改查
	例子：查询学生的时候查询出老师   一个对象中包含另一个对象
	@Select("select s.id id,s.name name ,s.age age,t.id `teacher.id`,t.name `teacher.name` from student s left join teacher t on s.tid=t.id")
	List<Student> selAllStudent();
	
	例子：查询老师的时候查询出学生  一个对象中包含另一个集合
	
	在StudentMapper接口中添加
	@Select("select * from student where tid=#{0}")
	List<Student> selByTid(int tid);
	
	在TeacherMapper接口中添加
	@Results(value={
			@Result(id=true,property="id",column="id"),
			@Result(property="name",column="name"),
			@Result(property="list",column="id",many=@Many(select="com.wzy.mapper.StudentMapper.selByTid"))
		})
	@Select("select * from teacher") 	
	List<Teacher> selAllTeacher();





	

#### MyBatis的问题：

##### 1.当实体类中的属性名和表中的字段名不一致时，使用MyBatis进行查询操作时无法查询出相应的结果的问题以及针对问题采用的两种办法：

> 	解决办法一: 通过在查询的sql语句中定义字段名的别名，让字段名的别名和实体类的属性名一致，这样就可以表的字段名和实体类的属性名一一对应上了，这种方式是通过在sql语句中定义别名来解决字段名和属性名的映射关系的。

> 	解决办法二: 通过<resultMap>来映射字段名和实体类属性名的一一对应关系。这种方式是使用MyBatis提供的解决方式来解决字段名和属性名的映射关系的。

##### 2.数据库连接池技术
	在内存中开辟一块空间，存放多个数据库连接对象
	原理
		正常情况下：
			每一次web请求都需要建立一次数据库连接(费时，费内存)且每次连接都需要关闭。
		数据库连接池：
			当查询完数据之后不关闭连接，供下一次请求使用
			减少了数据库连接和断开的时间消耗
##### 3.#{}和${}的区别
	#{}使用占位符
	${}使用字符串拼接
##### 4.mybatis配置文件详解
###### 下面为一个基本的mybatis.xml的配置文件
	第一步：我们需要导入dtd 因为不导入dtd没有提示
	第二步：配置<environments>标签，我们所需要连接数据库的信息
		这个有一个default=""属性 表示使用我们配置的哪一个环境（引用下面我们配置的<environment>标签的id）
	第三步：在<environments>标签下配置一个或者多个<environment>标签
		<environment>标签属性
			id="" 
	第四步：在<environment>标签下配置我们需要的数据源信息
		<transactionManager> 事务管理机制  
			type="JDBC"   表示使用jdbc的事务管理机制
		<dataSource>配置数据源信息
			type="POOLED" 表示使用数据库连接词技术
			<property> 表示连接数据库需要的信息
			<property name="driver" value="com.mysql.jdbc.Driver"/>
			<property name="url" value="jdbc:mysql://localhost:3306/test"/>
			<property name="username" value="root"/>
			<property name="password" value=""/>
	第五步：配置<mappers>标签 告诉mybatis去哪里寻找映射文件，从而找到sql语句
		<mappers>
			<package name="com.wzy.mapper"/>  // 表示这个包中有映射文件
			<mapper resource="xml的文件路径">
			<mapper class="Mapper接口的类路径" />
		</mappers>
	第六步：为了输出sql语句查询信息(注意set标签的位置在environments标签的上面)
	<!-- log4j功能注意顺序 鼠标放在根标签-->
	<settings>
		<setting name="logImpl" value="LOG4J"/>
	</settings>
	第七步：添加别名
	<!-- 增加别名 -->
	<typeAliases>
		<package name="com.wzy.pojo"/>
	</typeAliases>
##### mybatis配置文件
```xml
<?xml version="1.0" encoding="UTF-8"?>

<!--导入DTD  XML的语法检查器-->
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>

	<!-- log4j功能注意顺序 鼠标放在根标签-->
	<settings>
		<setting name="logImpl" value="LOG4J"/>
	</settings>

	<!-- 增加别名 -->
	<typeAliases>
		<package name="com.wzy.pojo"/>
	</typeAliases>


	<!--default 引用environment的id，表示当前所用的环境  -->
	<environments default="default">
		<!--声明可能使用的环境  -->
		<environment id="default">
			<!--使用原生JDBC事物  -->
			<transactionManager type="JDBC"></transactionManager>
			<dataSource type="POOLED">  <!--数据库连接池  -->
				<property name="driver" value="com.mysql.jdbc.Driver"/>
				<property name="url" value="jdbc:mysql://localhost:3306/test"/>
				<property name="username" value="root"/>
				<property name="password" value=""/>
			</dataSource>
		</environment>
	</environments>
	<mappers>
		<package name="com.wzy.mapper"/>
	</mappers>
</configuration>
```