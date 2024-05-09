# 如何在Java代码中使用不同Redis DB
 工作中可能因为要和别的团队做一些兼容，所以有时候会需要读取别人的Redsis，但是又不在同一个DB怎么办

## knife4j-redis-lettuce
* 基于SpringBoot2.X + lettuce 实现Redis集成、多库实时切换操作
* 基于源码可以自己做改动，实现业务
* gitee地址: https://gitee.com/hywang2017/knife4j-redis-lettuce
* 项目原地址：htps://github.com/it235/knife4j-redis-lettuce.git


### 如何快速接入
- Maven依赖集成

  ```xml
          <dependency>
              <groupId>com.github.it235</groupId>
              <artifactId>knife4j-redis-lettuce-stater</artifactId>
              <version>${currentVersion}</version>
          </dependency>
  ```

- `application.yml`或`properties`如下配置

  ```yml
  # 此处Key不可改变
  knife4j:
    redis:
      # 是否采用json序列化发方式，若不采用则为JDK序列化
      #jsonSerialType: 'Fastjson'
      host: localhost
      password: knife # 根据自己设定进行修改
      port: 6379
      databases: [0,1,2,3,4,5,6] # 要使用的库，会根据此处填写的库生成redisTemplate
      timeout: 60s
      lettuce: # lettuce基于netty，线程安全，支持并发
        pool:
          max-active: 50
          max-wait: -1ms
          max-idle: 8
          min-idle: 0
  ```
