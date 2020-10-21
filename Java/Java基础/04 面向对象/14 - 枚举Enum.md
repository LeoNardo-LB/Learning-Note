# 枚举Enum

枚举：有限数量的值存放在一个枚举类中

在JDK1.5之前，需要程序员自己通过特殊的方式来定义枚举类型。
在JDK1.5之后，Java支持enum关键字来快速的定义枚举类型。

## 枚举类的创建

#### JDK5之前

1.  创建类

2.  构造器加private私有化

3.  本类内部创建一组常量对象，并添加public static修饰符，对外暴露这些常量对象

##### 示例代码

```java
// 该类为枚举类
class Season{
    private String seasonName;
    public static Season SPRING=new Season("春");
    public static Season SUMMER=new Season("夏");
    public static Season AUTUMN=new Season("秋");
    public static Season WINTER=new Season("冬");
    private Season(String name){
        this.seasonName=name;
    }
    @Override
    public String toString() {
        return seasonName;
    }
}

public class TestEnum {
    public static void main(String[] args) {
        Season spring = Season.SPRING;
        System.out.println(spring); // 打印春
    }
}
```



#### JDK5之后

使用enum关键字创建枚举类

```java
【修饰符】 enum 枚举类名{
    常量对象列表
}
【修饰符】 enum 枚举类名{
    
    常量对象列表;
    
    其他成员列表;
}
```

##### 示例代码：

```java
public class EnumDemo01 {
	
	public static void main(String[] args) {
		Gun ak47 = Gun.AK47;
		
		// 查看枚举类的类型
		System.out.println("ak47.getClass() = " + ak47.getClass());
		
		//Obj.name() 返回对象toString的实现(默认为对象名)，此方法可以省略。
		System.out.println("ak47 = " + ak47);   //默认情况
		System.out.println("ak47.name() = " + ak47.name());
		
		//Obj.ordinal() 获取对象的序号(根据声明顺序,从0开始)
		System.out.println("ak47.ordinal() = " + ak47.ordinal());
		
		// #enum.Valueof(String str) 根据字符串获取对应枚举对象(区分大小写)
		System.out.println("----------根据传入字符串获取枚举类对象----------");
		Gun m4a1 = Gun.valueOf("M4A1");
		System.out.println("M4A1 = " + m4a1);
		
		// #enum.values() 获取所有的枚举对象
		System.out.println("----------现在输出枚举对象----------");
		Gun[] guns = Gun.values();
		Arrays.stream(guns).forEach((o) -> {
			System.out.print(o.ordinal() == guns.length - 1 ? o : (o + " | "));
		});
	}
}


enum Gun {
	AK47,
	M4A1,
	AWP,
	DEALGE
}
```

##### 示例代码2

```java
enum Week {
	MONDAY(1, "周一"), TUESDAY(2, "周二"), WENDSDAY(3, "周三"), THURSDAY(4, "周四"), FRIDAY(5, "周五"), SATUDAY(6, "周六"), SUNDAY(7, "周天");
	
	private int value; //序号
	
	private String desc; //描述
	
	private Week(int value, String desc) {
		this.value = value;
		this.desc = desc;
	}
	
	@Override
	public String toString() {
		return "今天是" + desc + "，代号为：" + value + "，序号为：" + this.ordinal();
	}
}
```

几点说明：

-   枚举类的常量对象列表必须在枚举类的首行，因为是常量，所以建议大写。

-   如果常量对象列表后面没有其他代码，那么“；”可以省略，否则不可以省略“；”。

-   编译器给枚举类默认提供的是private的无参构造，如果枚举类需要的是无参构造，就不需要声明，写常量对象列表时也不用加参数，

-   如果枚举类需要的是有参构造，需要手动定义private的有参构造，调用有参构造的方法就是在常量对象名后面加(实参列表)就可以。

-   枚举类默认继承的是java.lang.Enum类，因此不能再继承其他的类型。

-   JDK1.5之后switch，提供支持枚举类型，case后面可以写枚举常量名。

-   枚举类型如有其它属性，建议（不是必须）这些属性也声明为final的，因为常量对象在逻辑意义
    上应该不可变。

    

## 枚举类型常用方法

| 方法名 | 简介 |
| ------ | ---- |
|toString()| 默认返回的是常量名（对象名），可以继续手动重写该方法！（可省略）|
|name()|返回的是常量名（对象名） |
|ordinal()|返回常量的次序号，默认从0开始|
|values()|返回该枚举类的所有的常量对象，返回类型是当前枚举的数组类型，是一个静态方法|
|valueOf(String name)|根据枚举常量对象名称获取枚举对象|