
## @DisconfFile -- 配置文件注解 ##

作用：在客户端应用中生命需要在配置中心统一管理的配置文件

- filename 监管的配置文件名称

- env 配置文件对应的环境编码

- version 配置文件对应的版本号

- copy2TargetDirPath 需要拷贝文件到指定的目录
	

## @DisconfFileItem -- 配置文件属性注解 ##

作用：分布式的配置文件中的ITEM对应的注解


- name 配置文件里的KEY的名字

- associateField 所关联的域

## @DisconfItem -- 配置项注解 ##

作用：独立分布式的配置项注解


- key 配置项名称

- associateField 所关联的域

- env 配置文件对应的环境编码

- version 配置文件对应的版本号


## @DisconfUpdateService -- 回调服务注解 ##

作用：标识配置更新时需要进行更新的服务,需要指定它影响的配置数据，可以是配置文件或者是配置项

- classes 配置文件对应的Bean的Clazz

- confFileKeys 配置文件名称

- itemKeys 配置项名称