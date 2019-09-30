Environment对象

通过@PropertySource加载的properties文件，会以键值对的形式，封装到Environment对象中

在spring的bean对象里，可以实现EnvironmentAware，或者直接@Autowired注入Environment对象



Environment对象里，存储了环境变量，包括系统的环境变量，还有properties文件里的键值对，以及激活的profiles，默认的profiles等信息

里面还包含了一个PropertyResolver，会解析`${}`占位符，可以用@Value("${db.user}") 这种形式进行值的注入