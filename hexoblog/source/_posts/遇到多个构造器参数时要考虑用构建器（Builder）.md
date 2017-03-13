---
title: 遇到多个构造器参数时要考虑用构建器（Builder）
date: 2017-03-13 15:40:28
categories:
- 读书笔记
tags:
- effective java
---
为了保证一些必要参数的存在，可以用构造器传参的方式来创建对象。可以提供第一个只含有必要参数的构造器，第二个有一个可选参数的构造器，第三个有多个可选参数的构造器等等。  
```
public class Person {
        private String name;    //required
        private String phoneNum;    //required
        private int sex;    //optional
        private String year;    //optional
    
        public Person(String name, String phoneNum){
            this.name = name;
            this.phoneNum = phoneNum;
        }
    
        public Person(String name, String phoneNum, int sex){
            this.name = name;
            this.phoneNum = phoneNum;
            this.sex = sex;
        }
    
        public Person(String name, String phoneNum, int sex, String year){
            this.name = name;
            this.phoneNum = phoneNum;
            this.sex = sex;
        }
}
//客户端代码
Person person1 = new Person("lwa","13813813813");
Person person2 = new Person("lwa","13813813813",1);
Person person3 = new Person("lwa","13813813813",1,"13");
```

当你创建实例是可以用参数列表中最短的构造器，但是列表中会有很多个构造器。这个对象中只有两个可选参数，随着可选参数的增多，创建实例会变得越来越麻烦，你不得不一个个的查看参数个数来确保你用的构造器是你想使用的。  
***重叠构造模式可行*，但是当有许多参数的时候，客户端代码会变得很难编写，并且仍然难以阅读**  
```
public class Persion {
    private String name;//required
    private String phoneNum;//required
    private int sex;//optional
    private String year;//optional

    public void setName(String name) {
        this.name = name;
    }

    public void setPhoneNum(String phoneNum) {
        this.phoneNum = phoneNum;
    }

    public void setSex(int sex) {
        this.sex = sex;
    }

    public void setYear(String year) {
        this.year = year;
    }
}

//客户端代码
Person persion = new PersonJavaBeans();
persion.setName("lwa");
persion.setPhoneNum("13813813813");
persion.setSex(1);
```
遇到许多构造器参数时也可以用第二种模式javaBeans模式，调用一个无参构造函数俩创建对象，在用setter方法来设置必要参数和可选参数。  
这种模式虽然创建实例和阅读代码都很容易但是javaBeans模式有很严重的缺点，因为构造过程被分成了几个调用中，***在构造过程中javaBean可能处于不一致的状态***。另外，***javaBean模式阻止了把类做成不可变的可能***，这需要程序员付出额外的努力来保证它的线程安全。  
有第三种模式，既保证了安全性同时也保证了可读性。
```
public class Person {
    private String name;//required
    private String phoneNum;//required
    private int sex;//optional
    private String year;//optional
    public static class Builder{
        private String name;//required
        private String phoneNum;//required
        private int sex;//optional
        private String year;//optional

        public Builder(String name, String phoneNum){
            this.name = name;
            this.phoneNum = phoneNum;
        }
        public Builder sex(int val){
            this.sex = val;
            return this;
        }

        public Builder year(String val){
            this.year = val;
            return this;
        }

        public Person build(){
            return new Person(this);
        }
    }
    private Person(Builder builder){
        this.name = builder.name;
        this.phoneNum = builder.phoneNum;
        this.sex = builder.sex;
        this.year = builder.year;
    }
}
//客户端代码
Person person = new Person.Builder("lwa","13813813812").sex(12).year("22").build();
```
创建builder 的步骤
1. 创建一个静态内部类
2. 给静态内部类创建一个含有必要参数的构造函数
3. 创建一个返回外部类的方法
4. 给其他可选参数创建赋值方法，返回静态内部类
5. 创建外部类的构造函数，参数为静态内部类并赋值参数  

缺点：为了创建对象必须创建他的构造器虽然性能开销在实践中可能不明显，但是在某些特别注重性能的情况下会产生影响  
简而言之：  如果类的构造器或者静态工厂中具有多个参数，设计者种类时，Builder模式是一种很不错的选择，特别是大多数参数时可选参数时。