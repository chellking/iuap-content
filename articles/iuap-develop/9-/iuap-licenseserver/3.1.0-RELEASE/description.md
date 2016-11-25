
iUAP平台采用LicenseServer作为License授权机制，实现对各业务场景下的License授权及管理。

本组件提供对"CPU个数"场景接口、"特征数"场景、""用户数"场景授权管理接口的封装，供其它产品或者项目简便的集成和管理授权。

iUAP LicenseServer组件实现了Http连接池和LicenseClient进行交互，使用非对称加密算法保证传输的安全性。 分布式架构保证了服务的高可用性，水平无限制集群扩展，持续提供可用服务。 同时，LicenseServer提供LicenseClient的maven构件方便基于maven的集成。



