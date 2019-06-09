高端系统maven打包以及ejb调用梳理



### 1.本地开发环境下

用的Profile是jetty，没有直接调ejb，而是将impl模块打包进来，并且gsms.application.context引用的是spring/gsms-spring.xml，详情见  mqreceive模块下的pom.xml

```xml
<profile>
			<id>jetty</id>
			<activation>
				<activeByDefault>false</activeByDefault>
			</activation>
			<build>
				<finalName>gsms</finalName>
				<defaultGoal>jetty:run</defaultGoal>
				<resources>
					<resource>
						<filtering>true</filtering>
						<directory>src/main/resources</directory>
					</resource>
					<resource>
						<filtering>true</filtering>
						<directory>src/test/resources</directory>
					</resource>
				</resources>
				<!-- 去除了中间不重要的plugins标签 -->
			</build>
			<dependencies>
				<dependency>
					<groupId>${project.parent.groupId}</groupId>
					<artifactId>csair-gsms-hvp-applicationImpl</artifactId>
					<version>${project.parent.version}</version>
				</dependency>
			</dependencies>
			<properties>
				<gsms.application.context>
				<![CDATA[ -->
				<import resource="classpath*:META-INF/gsms/spring/gsms-spring.xml" />
				<!-- ]]>
				</gsms.application.context>
			</properties>
		</profile>

<!-- build 标签下的properties标签内的变量，gsms.application.context，通过上面resource标签的filtering为true激活，并设置了替换的目录为src/main/resources ,那么该目录下所有文件内的  ${gsms.application.context} 都会被替换成这里配置的值，即  xxxx/gsms-spring.xml

下图中运行mq项目的Run Configuration中
jetty:run -Pjetty -Djetty.port=8848
其中  -P后面指定了选用哪一个profile
      -D设置了一些属性，即对pom.xml里的一些属性，用对应值替换
```

![1555913979433](C:\Users\yogurtbee\AppData\Roaming\Typora\typora-user-images\1555913979433.png)



### 2.默认情况

gsms.application.context引用的是 ejb-context.xml，即调用ejb（生产环境下是如此）

ejb-context.xml里配置了ejb的调用接口，并生成了id为jndiTemplate的bean



下面是ejb-context.xml

```xml
<bean id="jndiTemplate" class="org.springframework.jndi.JndiTemplate"
		lazy-init="true">
		<property name="environment">
			<props>
				<prop key="java.naming.provider.url">${gsms.hvp.ejb.jndi.url}</prop>
				<prop key="java.naming.factory.initial">org.jnp.interfaces.NamingContextFactory</prop>
				<prop key="java.naming.factory.url.pkgs">java.naming.factory.url.pkgs</prop>
			</props>
		</property>
	</bean>
<!--在打生产包时（如给25服务器打包），其中  ${gsms.hvp.ejb.jndi.url}  会被producat25.application.properties文件中的gsms.hvp.ejb.jndi.url键值对给替换


```



下面是product25.application.properties

```properties
gsms.hvp.ejb.jndi.url=jnp://10.72.149.25:6000

# \u5728\u7ebf\u67e5\u770b\u65e5\u5fd7json
servers.json.str={"app25"\:{"rootDir"\:"/usr/local/jboss-eap-4.3/jboss-as/server/%s/log","serverName"\:"app25_server","shortName"\:"app25","localIp"\:"10.72.149.25","invokeBean"\:"appFileExplorerImpl","instancesMap"\:{"ejb"\:"gsms-ejb-hvp","ws"\:"gsms-webservice-hvp"}},"web17"\:{"rootDir"\:"/usr/local/jboss-eap-4.3/jboss-as/server/%s/log","serverName"\:"web17_server","shortName"\:"web17","localIp"\:"172.27.46.17","invokeBean"\:"webFileExplorerImpl","instancesMap"\:{"web"\:"gsms-web-hvp"}}
                "app25":\
                {"rootDir":"/usr/local/jboss-eap-4.3/jboss-as/server/%s/log",\
                "serverName":"app25_server","shortName":"app25","localIp":"10.72.149.25",\
                "invokeBean":"appFileExplorerImpl",\
                "instancesMap":{"ejb":"gsms-ejb-hvp","ws":"gsms-webservice-hvp"}},\
                "web17":\
                {"rootDir":"/usr/local/jboss-eap-4.3/jboss-as/server/%s/log",\
                "serverName":"web17_server","shortName":"web17","localIp":"172.27.46.17",\
                "invokeBean":"webFileExplorerImpl",\
                "instancesMap":{"web":"gsms-web-hvp"}}\
                }
```



### 3打生产包的情况

```xml
<profile>
		<!-- 高端子系统 生产  10.72.149.25-->
			<id>gsmshvpproduction25</id>
			<activation>
				<activeByDefault>false</activeByDefault>
			</activation>
			
            <build>
                <filters>
                     <filter>${baseDir}/../../conf/product.common.application.properties</filter>
                     <filter>${baseDir}/../../conf/product.25.application.properties</filter>
                </filters>
                <resources>
                    <resource>
                           <directory>src/main/resources</directory>
                           <filtering>true</filtering>
                    </resource>
                </resources>
            </build>
		</profile>
		
		<profile>
<!--
通过配置filter标签，声明了可用于替换的properties文件
后面resource标签下通过directory指定了需要进行替换的文件目录
通过设置filtering为true，激活了filter，会用filter标签内的properties文件里的键值对，去替换directory目录下的所有文件里包含了${key}的字符串
```

