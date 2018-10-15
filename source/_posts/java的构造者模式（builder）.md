
---
title: java的构造者模式
date: 2018-10-12 16:40:52
tags:
categories: java代码
---

&emsp;&emsp;我们在工作中写代码会遇到一种情况，就是设置一个对象属性值，通常方式有两种：

 1. Animal animal=new Animal("3岁",20kg,"牛奶");
 2. Animal animal=new Animal();
    animal.setAge("3岁");
    animal.setWeight("20kg");
    animal.setFood("牛奶");

**第一种方式:**
&emsp;&emsp;相当于在构造函数里传递参数，但这样加入参数的时候，不能明确的知道往这个对象里加入了什么属性的内容。

**第二种方式:**
&emsp;&emsp;虽然可以根据set函数名看到将要设置的值是什么值，但是这种写发，略显冗余。



&emsp;&emsp;**在设计模式中有构造者模式(builder),在类的构造器或静态工厂具有多个参数。设计这种类时，builder模式就是个不错的选择。Demo代码如下：**

 public class Purchase {
 private final String shipNo;
 private final String menuId;
 private final String menuName;
 private final Double price;

 public static class Builder {
     private final String shipNo;
     private String menuId;
     private String menuName;
     private final Double price = 0.0;

     public Builder(String shipNo) {
         this.shipNo = shipNo;
     }

     public Builder menuId(String val) {
         menuId = val;
         return this;
     }

     public Builder menuName(String val){
         menuName=val;
         return this;
     }

     public Purchase build() {
         return new Purchase(this);
     }
 }

 private Purchase(Builder builder) {
     shipNo = builder.shipNo;
     menuId = builder.menuId;
     menuName = builder.menuName;
     price = builder.price;
 }
}


其结果就是这种设置属性是多个方法连接的方式：

```
public static void main(String[] args) {
Purchase p=new Builder("S0001").menuId("11").menuName("宫保").build();
System.out.println(p.menuName);
}
```

也有另一种方式，省去代码的builder写法，就是使用lombok：


```
import lombok.Builder;
import lombok.Data;

@Builder
@Data
public class LomBokDemo {

 private String name;
 private Integer age;
 public static void main(String[] args) {
     LomBokDemo wyz = LomBokDemo.builder().age(12).name("张三").build();
     System.out.println(wyz);

 }   
}

```
