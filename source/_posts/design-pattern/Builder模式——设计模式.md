---
title: Builder模式——设计模式
tag:
	-- 设计模式
	-- Design Pattern
	-- Java
---

### Builder模式

#### 前言

构建对象时，如果碰到类有很多参数——其中很多参数类型相同而且很多参数可以为空时，推荐Builder模式来完成。当参数数量不多、类型不同而且都是必须出现时，通过增加代码实现Builder往往无法体现它的优势。在这种情况下，理想的方法是调用传统的构造函数。再者，如果不需要保持不变，那么就使用无参构造函数调用相应的set方法吧。

#### 普通方式，通过构造方法创建实例

```java
package com.ubuntuvim.pattern.builder;

/**
 * Builder 模式 创建类实例方式:<br>
 * 1. 普通方式，通过构造方法创建 <br>
 * 2. JavaBean方式创建<br>
 * 3. Builder 模式
 * 
 * @author ubuntuvim
 */
public class Car {
    /**
     * 必需属性
     */
    private String carBody;//车身
    private String tyre;//轮胎
    private String engine;//发动机
    private String aimingCircle;//方向盘
    /**
     * 可选属性
     */
    private String decoration;//车内装饰品
    
    /**
     * 必备属性创建实例
     * @param carBody
     * @param tyre
     * @param engine
     * @param aimingCircle
     */
    public Car(String carBody, String tyre, String engine, String aimingCircle) {
        super();
        this.carBody = carBody;
        this.tyre = tyre;
        this.engine = engine;
        this.aimingCircle = aimingCircle;
    }

    /**
     * 增加可选属性创建实例
     * @param carBody
     * @param tyre
     * @param engine
     * @param aimingCircle
     */
    public Car(String carBody, String tyre, String engine, String aimingCircle, String decoration) {
        super();
        this.carBody = carBody;
        this.tyre = tyre;
        this.engine = engine;
        this.aimingCircle = aimingCircle;
        this.decoration = decoration;
    }

    @Override
    public String toString() {
        return "Car [carBody=" + carBody + ", tyre=" + tyre + ", engine=" + engine + ", aimingCircle=" + aimingCircle
                + ", decoration=" + decoration + "]";
    }
}
```

这是最常见的、最普通的方式——通过构造方法传递属性，并创建实例。

简单使用main方法模拟调用。

```java
package com.ubuntuvim.pattern.builder;

public class Client {
    public static void main(String[] args) {
        // 通过不同方式获取实例
        
        // 1. 普通方式，通过构造方法初始化属性，弊端：参数多，而且更重要的是参数的次序不能错
        Car c = new Car("carBody", "tyre", "engine", "aimingCircle");
        System.out.println(c);
       
    }
}
```

这种方式简单，但是参数多的情况传参麻烦，并且参数次序不能错，否则得到的实例不是自己想要的。

#### JavaBean方式创建

```java
package com.ubuntuvim.pattern.builder;

/**
 * Builder 模式 创建类实例方式:<br>
 * 1. 普通方式，通过构造方法创建 <br>
 * 2. JavaBean方式创建<br>
 * 3. Builder 模式
 * 
 * @author ubuntuvim
 */
public class Car2 {
    /**
     * 必需属性
     */
    private String carBody;// 车身
    private String tyre;// 轮胎
    private String engine;// 发动机
    private String aimingCircle;// 方向盘
    /**
     * 可选属性
     */
    private String decoration;// 车内装饰品

    public void setCarBody(String carBody) {
        this.carBody = carBody;
    }

    public void setTyre(String tyre) {
        this.tyre = tyre;
    }

    public void setEngine(String engine) {
        this.engine = engine;
    }

    public void setAimingCircle(String aimingCircle) {
        this.aimingCircle = aimingCircle;
    }

    public void setDecoration(String decoration) {
        this.decoration = decoration;
    }

    @Override
    public String toString() {
        return "Car2 [carBody=" + carBody + ", tyre=" + tyre + ", engine=" + engine + ", aimingCircle=" + aimingCircle
                + ", decoration=" + decoration + "]";
    }
}
```

调用

```java
package com.ubuntuvim.pattern.builder;

public class Client {
    public static void main(String[] args) {
        // 通过不同方式获取实例
        
        // 1. 普通方式，通过构造方法初始化属性，弊端：参数多，而且更重要的是参数的次序不能错
        Car c = new Car("carBody", "tyre", "engine", "aimingCircle");
        System.out.println(c);
        
        // 2. javabean方式，通过setter方法初始化属性
        Car2 c2 = new Car2();
        c2.setCarBody("carBody2");
        c2.setTyre("tyre2");
        c2.setEngine("engine2");
        c2.setAimingCircle("aimingCircle2");
        System.out.println(c2);

    }
}
```

提供无参的构造函数，暴露一些公共的方法让用户自己去设置对象属性，这种方法较之第一种似乎增强了灵活度，用户可以根据自己的需要随意去设置属性。但是这种方法自身存在严重的缺点：
1. 因为构造过程被分到了几个调用中，在构造中 JavaBean 可能处于不一致的状态。类无法仅仅通过判断构造器参数的有效性来保证一致性。
2. 参数无法校验，本例子中`carBody`、`tyre`、`engine`、`aimingCircle`这几个属性是必须的。Javabean的方式无法校验。

#### Builder模式

```java
package com.ubuntuvim.pattern.builder;

/**
 * Builder 模式 创建类实例方式:<br>
 * 1. 普通方式，通过构造方法创建 <br>
 * 2. JavaBean方式创建<br>
 * 3. Builder 模式
 * 
 * @author ubuntuvim
 */
public final class Car3 {
    /**
     * 必需属性
     */
    private String carBody;// 车身
    private String tyre;// 轮胎
    private String engine;// 发动机
    private String aimingCircle;// 方向盘
    /**
     * 可选属性
     */
    private String decoration;// 车内装饰品

//  public void setCarBody(String carBody) {
//      this.carBody = carBody;
//  }
//
//  public void setTyre(String tyre) {
//      this.tyre = tyre;
//  }
//
//  public void setEngine(String engine) {
//      this.engine = engine;
//  }
//
//  public void setAimingCircle(String aimingCircle) {
//      this.aimingCircle = aimingCircle;
//  }
//
//  public void setDecoration(String decoration) {
//      this.decoration = decoration;
//  }

    /**
     * 必须的方法否则后面的`build()`方法无法把属性传递进来，<br>
     * 汽车指定某个制造商才能制造汽车，
     * @param builder
     */
    public Car3(Builder builder) {
        this.aimingCircle = builder.aimingCircle;
        this.carBody = builder.carBody;
        this.decoration = builder.decoration;
        this.engine = builder.engine;
        this.tyre = builder.tyre;
    }


    /**
     * 汽车不会自己生产自己，通常是汽车制造商Builder生产机车，
     * 汽车制造商要制造汽车，它必须拿到汽车一样的属性。
     */
    public static final class Builder {
        /**
         * 必需属性
         */
        private String carBody;// 车身
        private String tyre;// 轮胎
        private String engine;// 发动机
        private String aimingCircle;// 方向盘
        /**
         * 可选属性
         */
        private String decoration;// 车内装饰品

        // 制造商默认生产的汽车
        public Builder() {
        }
        
        // 为了能够链式调用，每个setter方法都返回this
        public Builder setCarBody(String carBody) {
            this.carBody = carBody;
            return this;
        }
        public Builder setTyre(String tyre) {
            this.tyre = tyre;
            return this;
        }
        public Builder setEngine(String engine) {
            this.engine = engine;
            return this;
        }
        public Builder setAimingCircle(String aimingCircle) {
            this.aimingCircle = aimingCircle;
            return this;
        }
        public Builder setDecoration(String decoration) {
            this.decoration = decoration;
            return this;
        }
        
        /**
         * 制造商返回制造出来的汽车
         * @return
         * @throws Exception 
         */
        public Car3 build() throws Exception {
            if (this.carBody == null)
                throw new Exception("carBody不允许为空。");
            if (this.aimingCircle == null)
                throw new Exception("aimingCircle不允许为空。");
            if (this.tyre == null)
                throw new Exception("tyre不允许为空。");
            // this就是制造出来的汽车，默认情况下都是宝马
            // 可以通过Builder提供的setter方法改变汽车的属性
            return new Car3(this);
        }
    }
    
    @Override
    public String toString() {
        return "Car3 [carBody=" + carBody + ", tyre=" + tyre + ", engine=" + engine + ", aimingCircle=" + aimingCircle
                + ", decoration=" + decoration + "]";
    }
}
```

调用。

```java
package com.ubuntuvim.pattern.builder;

public class Client {
    public static void main(String[] args) throws Exception {
        // 通过不同方式获取实例
        
        // 1. 普通方式，通过构造方法初始化属性，弊端：参数多，而且更重要的是参数的次序不能错
        Car c = new Car("carBody", "tyre", "engine", "aimingCircle");
        System.out.println(c);
        
        // 2. javabean方式，通过setter方法初始化属性
        Car2 c2 = new Car2();
        c2.setCarBody("carBody2");
        c2.setTyre("tyre2");
        c2.setEngine("engine2");
        c2.setAimingCircle("aimingCircle2");
        System.out.println(c2);
        
        // 3. Builder模式，设置汽车必须属性
        Car3 c32 = new Car3.Builder()
                .setAimingCircle("奔驰")
                .setCarBody("奔驰")
                .setEngine("奔驰")
                .setTyre("奔驰")
                .build();
        System.out.println(c32);

        // 3. Builder模式，设置汽车必须属性+可选属性
        Car3 c33 = new Car3.Builder()
                .setAimingCircle("奔驰")
                .setCarBody("奔驰")
                .setEngine("奔驰")
                .setTyre("奔驰")
                .setDecoration("可选属性：高级音响")
                .build();
        System.out.println(c33);
        

        // 3. Builder模式，没有设置任何属性，属性校验不通过
        Car3 c3 = new Car3.Builder().build();
        System.out.println(c3);
        
    }
}
```

首先参数的设置包含了javabean的特点，不需要关注参数的次序并且可以链式设置，代码更优雅；其次，可以校验一些必须的参数。

以上就是Builder模式。
