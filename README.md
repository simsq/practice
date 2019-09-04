#  服务化架构实践
使用NetCore构建三高服务化架构并部署到K8S
### 技术点
NetCore2.2：所有的基础服务代码</br>
EFCore2.2：用来做底层数据访问的CRUD</br>
Polly：NET下的一个故障处理库（我们主要用到重试功能，例如消息发送失败将根据策略重试）</br>
Golang：目前使用go做一些系统边缘的业务，后期会把全部业务代码使用Go语言重构</br>
Mysql：业务数据存储</br>
Vue：前端使用vue框架</br>
SLB：使用阿里云SLB做内部服务的负责均衡及健康检查等</br>
Nginx：作为所有基础服务的入口</br>
APIGetway：阿里云api网关负责转发不同客户的请求（内部业务策略，因客户多数是B客户 多数接口为定制化）</br>
Kong：Api网关（内置IP白名单，限流 熔断 认证等功能）</br>
Redis：用来保存用户菜单权限数据信息并且结合redis内置策略过期时间做用户token授权数据平滑过期策略。</br>
RabiitMQ：标准的消息队列中间件。用来发送短信 邮箱 微信 等通知（以及内部业务订单数据和业务解耦）</br>
MongoDB：日志存储</br>
Docker：打包NetCore服务镜像</br>
K8s：所有服务均部署在k8s中（我们用到了k8s中的 自动扩容，自动重启，自动伸缩容器数量这些功能）</br>
gitlab：代码托管，以及CI/CD。</br>

### 服务化拆分规则
1.我们遵循内部业务便捷化最佳实践。（核心业务主要拆分为四个模块：用户 商品  订单 积分 ）</br>
2.服务化后最大的难题在于分布式事务，所有我们尽量的将相关的业务写在一个耦合在一起中。</br>
3.我们将涉及到高并发的业务单独部署。</br>
4.演进式拆分，根据性能监控平台的数据，对服务进行不断优化或拆分。</br>

### 配置中心
最早选型的时候决定使用携程的Apollo配置中心（支持docker 集群部署），后来因为我们面对的客户多数为B客户，在应对B客户多样化需求的时候apollo就显的比较吃力。
最后决定参考Apollo的思想与内部业务自研配置中心，现在自研的配置中心除了支持常见的配置管理外，更能支持我们内部业务代码的执行顺序（管道思想）

### 服务
1.服务之间统一由授权平台（基于OAuth2.0）颁发token进行授权通信</br>
2.服务之间通过http+json的方式进行数据通信。</br>
3.应用层服务提供标准的restful风格的api+json与前端进行数据传输</br>

### 链路追踪
阿里云链路追踪为我们提供了分布式应用中整套调用链（服务之间的调用）的关系看板 接口请求量 应用依赖分析等，对我们最大的提升就是帮助我们分析分布式系统中的性能瓶颈。

### CI/CD
b客户需求多样化，公司发展太快客户也越来越多，我们版本的发布也非常频繁，传统人肉发布的方式会给我们很痛苦，所有我们引用runner，通过在gitlab对提交的版本打标签的方式来构建镜像，镜像构建成功后自动推送到k8s中，k8s会根据我们配置的策略自动升级。
###### gitlab-ci.yml 脚本
https://github.com/simsq/Software-Architecture-in-Practics/blob/master/.gitlab-ci.yml
###### k8s.yml 脚本
https://github.com/simsq/Software-Architecture-in-Practics/blob/master/k8sPod.yml
