# 1.属性复制（mapstruct）

## 常用工具

在项目一般会用到各种对象PO、BO、VO、DTO来回转换。常用方法有

* org.apache.commons.beanutils.BeanUtils.copyProperties
* org.apache.commons.beanutils.PropertyUtils.copyProperties
* org.springframework.beans.BeanUtils.copyProperties
* org.springframework.cglib.beans.BeanCopier.copy
* org.mapstruct

> BeanUtils和PropertyUtils都是根据属性名称进行复制，如果属性名相同，类型不同，BeanUtils会直接转换，PropertyUtils会抛出异常。
>
> Apache 主要集中了各种丰富的功能(日志、转换、解析等等),导致性能变差。
>
> Spring BeanUtils则是直接通过反射来读取和写入
>
> BeanCopier动态生成一个要代理类的子类,其实就是通过字节码方式转换成性能最好的get和set方式,只需考虑创建BeanCopier的开销

## mapstruct使用

###  依赖引入

```xml
<org.mapstruct.version>1.4.1.Final</org.mapstruct.version>
<org.projectlombok.version>1.18.16</org.projectlombok.version>
<lombok-mapstruct-binding.version>0.2.0</lombok-mapstruct-binding.version>

<dependency>
     <groupId>org.mapstruct</groupId>
     <artifactId>mapstruct</artifactId>
     <version>${org.mapstruct.version}</version>
 </dependency>
<dependency>
     <groupId>org.projectlombok</groupId>
     <artifactId>lombok</artifactId>
     <version>${org.projectlombok.version}</version>
     <scope>provided</scope>
</dependency>

 <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <annotationProcessorPaths>
                        <path>
                            <groupId>org.mapstruct</groupId>
                            <artifactId>mapstruct-processor</artifactId>
                            <version>${org.mapstruct.version}</version>
                        </path>
                        <path>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                            <version>${org.projectlombok.version}</version>
                        </path>
                        <path>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok-mapstruct-binding</artifactId>
                            <version>${lombok-mapstruct-binding.version}</version>
                        </path>
                    </annotationProcessorPaths>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

### 准备两个实体类

```java
package com.hywang.sca.openapi.vo;

import lombok.Data;

import java.io.Serializable;
import java.time.LocalDateTime;

/**
 * @author ywang
 * @version 1.0.0
 * @className UserVO.java
 * @description 用户
 * @createTime 2020年12月18日 14:15:00
 */
@Data
public class UserVO implements Serializable {
    private static final long serialVersionUID = 8581557595007972659L;
    /**
     * 主键
     */
    private Integer id;

    /**
     * 身份证
     */
    private String idCard;

    /**
     * 姓名
     */
    private String name;

    /**
     * 手机号
     */
    private String mobile;

    /**
     * 联系人手机号
     */
    private LocalDateTime birthday;

    /**
     * 性别
     */
    private Integer sex;

    /**
     * 创建时间
     */
    private LocalDateTime createTime;
}

```

```
package com.hywang.sca.dto;

import lombok.Data;

import java.io.Serializable;
import java.time.LocalDateTime;

@Data
public class UserDTO implements Serializable {

    private static final long serialVersionUID = 8113127982327468078L;
    /**
     * 主键
     */
    private Integer id;

    /**
     * 身份证
     */
    private String idCard;

    /**
     * 姓名
     */
    private String name;

    /**
     * 手机号
     */
    private String mobile;

    /**
     * 联系人手机号
     */
    private LocalDateTime birthday;

    /**
     * 性别
     */
    private Integer sex;

    /**
     * 创建时间
     */
    private LocalDateTime createTime;

}

```

### 通用的转换接口

```java
package com.hywang.sca.openapi.mapstruct;

import java.util.List;


/**
 * Mapper文件基类
 *
 * @param <D>目标对象，一般为DTO对象
 * @param <V>源对象，一般为需要转换的对象
 * @author -c
 */
public interface BaseDTO2VOMapper<D , V> {


    /**
     * 将源对象转换为目标对象
     *
     * @param v
     * @return D
     */
    D toDTO(V v);

    /**
     * 将源对象集合转换为目标对象集合
     *
     * @param es
     * @return List<D>
     */
    List<D> toDTO(List<V> es);


    /**
     * 将目标对象转换为源对象
     *
     * @param d
     * @return E
     */
    V toVO(D d);

    /**
     * 将目标对象集合转换为源对象集合
     *
     * @param ds
     * @return List<E>
     */
    List<V> toVO(List<D> ds);



}

```

### 具体对象转换接口

```java
package com.hywang.sca.openapi.mapstruct;

import com.hywang.sca.dto.UserDTO;
import com.hywang.sca.openapi.vo.UserVO;
import org.mapstruct.Mapper;
import org.mapstruct.factory.Mappers;

/**
 * @author hywang
 * @version 1.0.0
 * @className UserDTO2VO.java
 * @description
 * @createTime 2021年01月25日 16:50:00
 */
@Mapper(componentModel = "spring")
public interface UserDTO2VOMapper extends BaseDTO2VOMapper<UserDTO, UserVO> {
    UserDTO2VOMapper INSTANCE = Mappers.getMapper(UserDTO2VOMapper.class);
}

```

### 使用

```java
     @Test
    public void userTest() {
        UserDTO userDTO = new UserDTO();
        userDTO.setName("hywang");
        UserVO userVO = UserDTO2VOMapper.INSTANCE.toVO(userDTO);
        System.out.println(userVO.getName());
    }

```


```JAVA
@Mapper
public interface StorageMapper {

    StorageMapper INSTANCE = Mappers.getMapper( StorageMapper.class );

    @ToEntity
    @Mapping( target = "weightLimit", source = "maxWeight")
    ShelveEntity map(ShelveDto source);

    @ToEntity
    @Mapping( target = "label", source = "designation")
    BoxEntity map(BoxDto source);
}
```
```JAVA
@Mapper
public interface AddressMapper {

    @Mapping(target = "description", source = "person.description")
    @Mapping(target = "houseNumber", source = "address.houseNo")
    DeliveryAddressDto personAndAddressToDeliveryAddressDto(Person person, Address address);
}
```
```JAVA
@Mapper
public interface AddressMapper {

    @Mapping(target = "description", source = "person.description")
    @Mapping(target = "houseNumber", source = "hn")
    DeliveryAddressDto personAndAddressToDeliveryAddressDto(Person person, Integer hn);
}
```
```JAVA
 @Mapper
 public interface CustomerMapper {

     @Mapping( target = "name", source = "record.name" )
     @Mapping( target = ".", source = "record" )
     @Mapping( target = ".", source = "account" )
     Customer customerDtoToCustomer(CustomerDto customerDto);
 }
```
```JAVA
@Mapper
public interface CarMapper {

    @Mapping(source = "price", numberFormat = "$#.00")
    CarDto carToCarDto(Car car);

    @IterableMapping(numberFormat = "$#.00")
    List<String> prices(List<Integer> prices);
}
```

## 高级用法

https://mapstruct.org/documentation/stable/reference/html/