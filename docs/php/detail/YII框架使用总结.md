## 命名空间与自动加载

### 命名空间

php的命名空间只是为了解决重命名的问题，为了区分开同名的类，但不在一个命名空间下即可。有了命名空间，在使用相应命名空间下的类时，并不会自动的加载相应的类，也必须require引入之后才能使用。

---

### Yii的自动加载

yii框架中在BaseYii.php中有autoload()自动加载函数，这其中会加载yii框架中相关的类，因为所有的核心类都已经写在了classMap中了，根据类名即可找到映射的类文件，如`yii\base\Action' => YII2_PATH . '/base/Action.php'`等，而在BaseYii.php中已经定义了相关的路径常量如`YII2_PATH`。

另外不是yii框架的核心类，在base\Application.php中的preInit()函数中会初始化各种别名路径等，如@app其实就是项目所在根目录，即model,controller所在的目录，如`\Users\work\nginx\html\yiidemo`等，非核心类的查找会根据命名空间加类名在加上相应的别名目录，即可找到相应的类文件，并require进来，如app\library\base\CAction等，以app开头的，用@app的路径加上类命名空间等即可找到

## Yii下config配置项的加载与使用

### Yii配置的加载

在Yii框架中BaseYii类封装了基本的方法，其中有个`configure`方法用来设置配置项的，而最终的调用是在base/Object里调用，其中的继承关系是`Application->\yii\base\Application->Module->ServiceLocator->Component->Object`，所以在构造函数里传入配置文件config时，最终会调用父类Object的构造函数，而在Object构造函数里调用了`Yii::configure()`方法来设置配置项，如下所示：

```php
 public function __construct($config = [])
    {
        if (!empty($config)) {
            Yii::configure($this, $config);
        }
        $this->init();
    }
```

---

### Yii配置项数据的调用

由上面可知，配置数据会加载成应用的全局属性，因为在`Yii::configure($this,$config)`里，传入的是$this即Application应用对象本身，而这个函数会把所有的配置项做该应用对象的属性进行赋值，如下所示：

```php
public static function configure($object, $properties)
{
    foreach ($properties as $name => $value) {
        $object->$name = $value;
    }

    return $object;
}
```

而在_base\Application_的构造函数里又会把Application对象本身赋值给Yii，如下所示：

```php
public function __construct($config = [])
{
    Yii::$app = $this;
    static::setInstance($this);
    $this->state = self::STATE_BEGIN;
    $this->preInit($config);
    $this->registerErrorHandler($config);
    Component::__construct($config);
}
```

所以要调用全局配置数据即这样调用即可`Yii::$app->params,Yii::$app->db,Yii::$app->memcache`获取相应配置项的数据

---

### Yii框架配置项的限制

Yii框架在config目录中并不能随便添加自定义配置项，如在web.php中$config中添加相应的配置项xxx => xxx等，只能在$config中添加Application中所定义出现的属性，如`id,basePath,defaultRoute,params`等，如果添加没有的属性，则会报错，如下所示：

```php
public function __set($name, $value)
{
    $setter = 'set' . $name;
    if (method_exists($this, $setter)) {
        // set property
        $this->$setter($value);

        return;
    } elseif (strncmp($name, 'on ', 3) === 0) {
        // on event: attach event handler
        $this->on(trim(substr($name, 3)), $value);

        return;
    } elseif (strncmp($name, 'as ', 3) === 0) {
        // as behavior: attach behavior
        $name = trim(substr($name, 3));
        $this->attachBehavior($name, $value instanceof Behavior ? $value : Yii::createObject($value));

        return;
    } else {
        // behavior property
        $this->ensureBehaviors();
        foreach ($this->_behaviors as $behavior) {
            if ($behavior->canSetProperty($name)) {
                $behavior->$name = $value;

                return;
            }
        }
    }
    if (method_exists($this, 'get' . $name)) {
        throw new InvalidCallException('Setting read-only property: ' . get_class($this) . '::' . $name);
    } else {
        throw new UnknownPropertyException('Setting unknown property: ' . get_class($this) . '::' . $name);
    }
}
```

> 注：所以只能在params或其他已有改改配置项下面添加自己的配置，即需要注意层级关系，不能在$config外层添加配置，如：

```php
##config/params.php
<?php

$redisConf = require(__DIR__ . '/redis.conf.php');
$memcacheConf = require(__DIR__ . '/memcache.conf.php');
return [
    'adminEmail' => 'admin@example.com',
    'redisConf' => $redisConf,
    'memcacheConf' => $memcacheConf,
];
```

## Yii框架的事件、行为及过虑器机制

我理解的所有的事件、行为、过虑器都是发生在应用处理过程中的一个插曲，就像java里面的切面编程一样，而最终这三个东西都会在runAction之前即beforeAction方法里执行，有的可能会在beforeFilter方法里执行，而beforeFilter里再执行相应的beforeAction等方法

### Yii框架里的访问控制AccessControl

一般的权限控制估计都会使用这个类，而可以用附加行为的方式$component->attachBehavior进行添加访问控制，如

```php
$component->attachBehavior('access',[
    'class' => yii\filters\AccessControl::className(),
    'rules' => [
        [
            'allow' => true,
            'roles' => ['@'],
        ],
        [
            'controllers' => ['api'],
            'allow' => true,
            'roles' => ['?'],
        ]
        [
            'allow' => false,
            'roles' => ['?'],
            'denyCallback' => function($rule, $action){
                return true;
            }
        ]
    ],
]); 
```

而AccessControl会需要yii\web\User类做用户检验，而当User类的enableSession=>true时，会自动去session里拿相应的用户信息，如果没有则会从指定的User里的identityClass里通过findIdentity函数去拿用户信息，这里的identityClass一般是自定义且实现yii\web\IdentityInterface接口的一个Model，如：

> 验证身份

```php
## AccessControl里的方法
public function beforeAction($action)
    {
        $user = $this->user;
        $request = Yii::$app->getRequest();
        /* @var $rule AccessRule */
        foreach ($this->rules as $rule) {
            if ($allow = $rule->allows($action, $user, $request)) {
                return true;
            } elseif ($allow === false) {
                if (isset($rule->denyCallback)) {
                    call_user_func($rule->denyCallback, $rule, $action);
                } elseif ($this->denyCallback !== null) {
                    call_user_func($this->denyCallback, $rule, $action);
                } else {
                    $this->denyAccess($user);
                }
                return false;
            }
        }
        if ($this->denyCallback !== null) {
            call_user_func($this->denyCallback, null, $action);
        } else {
            $this->denyAccess($user);
        }
        return false;
    }
````

> 获取用户信息

```php
public function getIdentity($autoRenew = true)
{
    if ($this->_identity === false) {
        if ($this->enableSession && $autoRenew) {
            $this->_identity = null;
            $this->renewAuthStatus();
        } else {
            return null;
        }
    }

    return $this->_identity;
}
##拿用户信息并赋值给identity
protected function renewAuthStatus()
{
    $session = Yii::$app->getSession();
    $id = $session->getHasSessionId() || $session->getIsActive() ? $session->get($this->idParam) : null;

    if ($id === null) {
        $identity = null;
    } else {
        /* @var $class IdentityInterface */
        $class = $this->identityClass;
        $identity = $class::findIdentity($id);
    }

    $this->setIdentity($identity);

    if ($identity !== null && ($this->authTimeout !== null || $this->absoluteAuthTimeout !== null)) {
        $expire = $this->authTimeout !== null ? $session->get($this->authTimeoutParam) : null;
        $expireAbsolute = $this->absoluteAuthTimeout !== null ? $session->get($this->absoluteAuthTimeoutParam) : null;
        if ($expire !== null && $expire < time() || $expireAbsolute !== null && $expireAbsolute < time()) {
            $this->logout(false);
        } elseif ($this->authTimeout !== null) {
            $session->set($this->authTimeoutParam, time() + $this->authTimeout);
        }
    }

    if ($this->enableAutoLogin) {
        if ($this->getIsGuest()) {
            $this->loginByCookie();
        } elseif ($this->autoRenewCookie) {
            $this->renewIdentityCookie();
        }
    }
}
```

> 自定义用户身份类

```php
#app\models\User

public static function findIdentity($id)
{
    $user = new User();
    $user->id = $id;
    $user->name = $buc_info['name'];
    return $user;
}
```

### 过虑器及事件

在application执行run的过程中，会执行父类Module的runAction，如下所示创建相应的Controller并执行runAction方法：

```php
public function runAction($route, $params = [])
    {
        $parts = $this->createController($route);
        if (is_array($parts)) {
            /* @var $controller Controller */
            list($controller, $actionID) = $parts;
            $oldController = Yii::$app->controller;
            Yii::$app->controller = $controller;
            $result = $controller->runAction($actionID, $params);
            Yii::$app->controller = $oldController;

            return $result;
        } else {
            $id = $this->getUniqueId();
            throw new InvalidRouteException('Unable to resolve the request "' . ($id === '' ? $route : $id . '/' . $route) . '".');
        }
    }
```

Controller里的runAction方法会创建相应的Action，并先执行相关Module的beforeActoin方法，如下所示：

```php
public function runAction($id, $params = [])
    {
        $action = $this->createAction($id);
        if ($action === null) {
            throw new InvalidRouteException('Unable to resolve the request: ' . $this->getUniqueId() . '/' . $id);
        }

        Yii::trace('Route to run: ' . $action->getUniqueId(), __METHOD__);

        if (Yii::$app->requestedAction === null) {
            Yii::$app->requestedAction = $action;
        }

        $oldAction = $this->action;
        $this->action = $action;

        $modules = [];
        $runAction = true;

        // call beforeAction on modules
        foreach ($this->getModules() as $module) {
            if ($module->beforeAction($action)) {
                array_unshift($modules, $module);
            } else {
                $runAction = false;
                break;
            }
        }

        $result = null;

        if ($runAction && $this->beforeAction($action)) {
            // run the action
            $result = $action->runWithParams($params);

            $result = $this->afterAction($action, $result);

            // call afterAction on modules
            foreach ($modules as $module) {
                /* @var $module Module */
                $result = $module->afterAction($action, $result);
            }
        }

        $this->action = $oldAction;

        return $result;
    }
```

所以最终的事件及过虑器都会在beforeAction方法里即执行相应的Action之前执行，以便做事先相应的处理