## 观察者模式

> 定义了对象之间的一对多依赖,当一个对象改变状态时,它的所有依赖者都将会收到通知并自动更新.

**观察者模式形容图**![](/assets/观察者模式.png)

## 设计谜题

有一个气象观察站,我们希望建立一个应用,有三种布告板\(用于显示不同的气象数据\),当气象站获取到最新的测量数据时,我们希望三种布告板能实时更新.

**类图设计**![](/assets/observer-class.png)

其中 `WeatherData`用于获取气象站最新测量数据\(三个`get`方法\),当数据更新时,会调用`onChanged`方法\(不要管为什么,这是气象站内部逻辑\).

**代码实现**

主题接口

```php
interface Sublect
{
    public function registerObserver(Observer $observer);

    public function removeObserver();

    public function nitifyObservers();
}
```

主题对象 `WeatherData`

```php
class WeatherData implements Sublect
{
    protected $observers = [];

    protected $pressure, $temperature, $humidity;

    public function registerObserver(Observer $observer)
    {
        if (array_search($observer, $this->observers) === false) {
            $this->observers[] = $observer;
        }
    }

    public function removeObserver()
    {
        if (($index = array_search($observer, $this->observers)) !== false) {
            unset($this->observers[$index]);
        }
    }

    public function nitifyObservers()
    {
        foreach ($this->observers as $observer) {
            $observer->update($this->getPressure(), $this->getTemperature(), $this->getHumidity());
        }
    }

    public function onChanged()
    {
        $this->nitifyObservers();
    }

    //获取最新气压
    public function getPressure()
    {
        return $this->pressure;
    }

    //获取最新温度
    public function getTemperature()
    {
        return $this->temperature;
    }

    //获取最新湿度
    public function getHumidity()
    {
        return $this->humidity;
    }

    //测试
    public function youNeedChanged()
    {
        $this->pressure = mt_rand(1, 99);
        $this->temperature = mt_rand(1, 99);
        $this->humidity = mt_rand(1, 99);

        $this->onChanged();
    }
}
```

观察者接口

```php
interface Observer
{
    //气压/温度/湿度
    public function update($pressure, $temperature, $humidity);
}
```

显示面板接口

```php
interface DisplayElement
{
    public function display();
}
```

观察者对象集

```php
class CurrentConditionsDisplay implements Observer, DisplayElement
{
    protected $subject;

    protected $pressure, $temperature, $humidity;

    //这里为什么会保留 Subject 接口的引用是为了方便的 remove 及 registe
    public function __construct(Sublect $subject)
    {
        $this->subject = $subject;
        $this->subject->registerObserver($this);
    }

    public function update($pressure, $temperature, $humidity)
    {
        $this->pressure = $pressure;
        $this->temperature = $temperature;
        $this->humidity = $humidity;

        $this->display();
    }

    public function display()
    {
        echo "Current pressure: {$this->pressure}, Current temperature: {$this->temperature}";
    }
}

//其他两种布告板省略
```

测试

```php
$weatherData = new WeatherData();

$display = new CurrentConditionsDisplay($weatherData);//把当前布告栏注册成为观察者
//$other = new OthersDisplay($weatherData);//把当前布告栏注册成为观察者
//$other = new OtherDisplay($weatherData);//把当前布告栏注册成为观察者

$weatherData->youNeedChanged();//气象站数据更新了会导致布告板实时更新
//Current pressure: 33, Current temperature: 46
```

## 另一种形式的观察者模式

我们知道,观察者总是被动的接受主题对象的推送,但有些场景下,我们希望观察者能主动的去获取数据;毕竟观察者数量这么多,主题对象不可能事先知道每个观察者需要的状态,并且也不会导致明明只需要一点点数据,却被迫收到一堆.

我们来重写设计上面的问题.

类图基本保持不变,只是在`WeatherData`类新增了`setChanged`方法并改变了`Observer`接口`update`签名.

重构后的主题接口

```php
interface Sublect
{
    public function registerObserver(Observer $observer);
    public function removeObserver();
    public function nitifyObservers($args = null);
}

interface Observer
{
    public function update(Sublect $subject, $object = null);
}
```

重构后的主题对象

```php
class WeatherData implements Sublect
{
    protected $observers = [];

    protected $pressure, $temperature, $humidity, $changed;

    public function nitifyObservers($args = null)
    {
        if ($this->changed) {
            foreach ($this->observers as $observer) {
                $observer->update($this, $args);
            }
            $this->changed = false;
        }
    }

    public function onChanged()
    {
        $this->setChanged();

        $this->nitifyObservers([
            'pressure' => $this->pressure,
            'temperature' => $this->temperature,
            'humidity' => $this->humidity,
        ]);
    }

    public function setChanged()//新增方法
    {
        $this->changed = true;
    }

    //其他方法保持不变
}
```

重构后的布告板对象

```php
class CurrentConditionsDisplay implements Observer, DisplayElement
{
    protected $subject;

    protected $pressure, $temperature, $humidity;

    //这里为什么会保留 Subject 接口的引用是为了方便的 remove 及 registe
    public function __construct(Sublect $subject)
    {
        $this->subject = $subject;
        $this->subject->registerObserver($this);
    }

    public function update(Sublect $subject, $object = null)
    {
        if ($subject instanceof Sublect) {
            //你可以用 拉取 的形式获取最新数据
            $this->pressure = $subject->getPressure();
            $this->temperature = $subject->getTemperature();
            $this->humidity = $subject->getHumidity();

            //也可以从推送数据中获取
            $this->pressure = $object['pressure'];
            $this->temperature = $object['temperature'];
            $this->humidity = $object['humidity'];
        }


        $this->display();
    }

    public function display()
    {
        echo "Current pressure: {$this->pressure}, Current temperature: {$this->temperature}";
    }
}
```

为什么要加一个 `setChanged` 方法

setChanged 让你在更新观察者时,有更多的弹性,能更适当的通知观察者,比方说,如果没有setCanged方法,气象站温度变化十分之一度时,都会通知所有观察者,你肯定不想让这么频繁的更新吧.我们可以控制温度变化达到一度时,调用setChanged,进行有效的更新.

