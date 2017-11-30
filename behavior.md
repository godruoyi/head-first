## 策略模式

> 定义了算法族，分别封装起来，让他们之间可以相互调用，此模式让算法的变化独立与调用算法的客户端。

## 设计谜题

一个冒险游戏，有多个游戏角色可以使用不同的武器，游戏中角色可以自由的切换武器,每个角色一次只能使用一样武器.

类图设计如下

![](/assets/策略模式.png)

```php
interface WeaponBehavior
{
    /**
     * 设置具体使用什么样的武器
     *
     */
    public function useWeapon();
}
```

武器的具体实现类

```php
class KnifeBehavior implements WeaponBehavior
{
    public function useWeapon()
    {
        //使用小刀
        return 1;
    }
}

class SwordBehavior implements WeaponBehavior
{
    public function useWeapon()
    {
        //使用大保健
        return 1024;
    }
}

//...
```

各种游戏角色均继承自`Character`超类。

```php
abstract public class Character
{
    protected $weapon;

    public function setWeapon(WeaponBehavior $weapon)
    {
        $this->weapon = $weapon;
    }

    /**
     * 获取角色战斗力
     */
    public function fightPower()
    {
        return $this->weapon->useWeapon()
    }
}
```

角色的具体实现.

```php
class King extends Character
{
    public function __construct()
    {
        //国王使用斧头
        $this->setWeapon(new SwordBehavior);
    }
}

class Queen extends Character
{
    public function __construct()
    {
        //王后使用匕首
        $this->setWeapon(new KnifeBehavior);
    }
}
```

## 战斗

```php
(new King)->fightPower(); //1024;
(new Queen)->fightPower(); //1;
```



