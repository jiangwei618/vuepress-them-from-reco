# 观察者模式在java中的实现
## 1. 主要类
java.util.Observable（主题）

java.util.Observer(观察者)

## 2. 实现
```
package com.gzmu.observer.observable;

import java.util.Observable;

public class Publisher extends Observable {

private String magazineName;


public String getMagazineName() {
return magazineName;
    }

public void publish(String magazineName) {
this.magazineName = magazineName;
        setChanged();
        notifyObservers(this);
    }

}
```

---

```
package com.gzmu.observer.observer;

import java.util.Observable;
import java.util.Observer;

import com.gzmu.observer.observable.Publisher;

public class Reader implements Observer {

    @Override
public void update(Observable o, Object arg) {
        Publisher p = (Publisher) o;
        System.out.println("我要订阅" + p.getMagazineName());
    }

}
```

---

```java
package com.gzmu.observer.test;


import org.junit.Test;

import com.gzmu.observer.observable.Publisher;

public class TestCase {

    @Test
public void register() {

        Publisher publisher = new Publisher();
        publisher.publish("Kent.Kuan的技术空间");

    }

}
```