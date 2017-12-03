## 观察者模式

> 定义了对象之间的一对多依赖,当一个对象改变状态时,它的所有依赖者都将会收到通知并自动更新.

**观察者模式形容图**

image

## 设计谜题

有一个气象观察站,我们希望建立一个应用,有三种布告板\(用于显示不同的气象数据\),当气象站获取到最新的测量数据时,我们希望三种布告板能实时更新.

**类图设计**

image

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
```

测试

```php
$weatherData = new WeatherData();

$display = new CurrentConditionsDisplay($weatherData);//把当前布告栏注册成为观察者
//$other = new OthersDisplay($weatherData);//把当前布告栏注册成为观察者
//$other = new OtherDisplay($weatherData);//把当前布告栏注册成为观察者

$weatherData->youNeedChanged();//气象站数据更新了
//Current pressure: 33, Current temperature: 46


```



