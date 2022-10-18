# gone

> 这是一个上天堂的故事  
> 逝者被埋葬后，在天堂永生，直到天崩地裂。  
> 天国管理着一片神圣的墓园，安葬在这片墓园的逝者会被升入天国。只有神圣的牧师才知道天国墓园的所在地，...

这是gone框架的第二版，第一版在[这里](https://gitlab.openviewtech.com/gone/gone#gone)

## 概念

> gone的意思是 `走了，去了，没了，死了`，那么gone框架管理都就是goner(逝者)

- Goner: 逝者 💀
- Tomb: 坟墓 ⚰️
- Cemetery: 墓园 🪦
- Cemetery.Bury:  安葬
- Priest: 神父✝️，负责给Goner下葬
- Cemetery.revive: 复活Goner，将其升入天国
- Heaven: 天国 🕊☁️
- Heaven.Start: 天国开始运行；Goner永生，直到天崩地裂
- Heaven.Stop:  天国崩塌，停止运行
- Angel: 天使 𓆩♡𓆪 ，实现了`Start(gone.Cemetery) error` 和 `Stop(gone.Cemetery) error`方法的Goner，升入天国后被变成天使
- Angel.Start: 天使开始工作；能力越大责任越大，天使是要工作的
- Angel.Stop: 天使停止工作；
- Vampire: 吸血鬼 🧛🏻‍，实现了`Suck(conf string, v reflect.Value) gone.SuckError`
  方法的是吸血鬼；吸血鬼是一个邪恶的存在，他可能毁掉整个天国。理论上吸血行为可以制造Goner，但是这可能会导致循环依赖，从而破坏系统。
- Vampire.Suck: 吸血鬼"吸血行为"
- Lucifer: 如果某个Goner既有Angel和Vampire的特征，那么他就是堕落天使。（这个概念在系统中没有定义，只是一种推导）

### 三种Goner

- 普通Goner
  > 普通Goner，可以用于抽象App中的Service、Controller、Client等常见的组件。
- 天使Angel
  > 天使会在天国承担一定的职责：启动阶段，天使的`Start`方法会被调用；停止阶段，天使的`Stop`方法会被调用；所以天使适合抽象"
  需要启停控制"的组件。
- 吸血鬼Vampire
  > 吸血鬼，具有吸血的能力，可以通过`Suck`方法去读取/写入被标记的字段；可以抽象需要控制其他组件某个属性的行为。

## 注入配置

## 普通Goner下葬

```go
package goner_demo

import "github.com/gone-io/gone"

type XGoner struct {
	gone.GonerFlag
}

type Demo struct {
	gone.GonerFlag
	a  *XGoner     `gone:"x-goner"` // x-goner 是 GonerId; 支持使用非导出属性
	A  XGoner      `gone:"x-goner"` // x-goner 是 GonerId; 支持结构体；⚠️尽量不要这样使用，由于结构体是值拷贝，会导致不能深度复制的问题
	A1 *XGoner     `gone:"x-goner"` // x-goner 是 GonerId; 支持结构体的指针
	A2 interface{} `gone:"x-goner"` // x-goner 是 GonerId; 支持接口

	B  *XGoner       `gone:"*"` //  支持匿名注入
	B1 []interface{} `gone:"*"` // 支持匿名注入数组
}
```

## 对吸血鬼下葬，被下葬的是Goner是一个Vampire

> 吸血鬼是一种邪恶的生物，他可以读取/吸入被注入的Goner的属性

```go
package goner_demo

import (
	"github.com/gone-io/gone"
	"github.com/magiconair/properties/assert"
	"reflect"
)

type ConfigVampire struct {
	gone.GonerFlag
}

func (*ConfigVampire) Suck(conf string, v reflect.Value) gone.SuckError {
	// conf = abc.dex,xxx|xxx
	// v = Demo.a 的 reflect.Value

	return nil
}

const ConfigVampireId = "x-config"

type Demo struct {
	// 吸血鬼不会被注入到属性中，而是会在属性上调用`Vampire.Suck`函数完成吸血，吸血鬼可以读取、写入属性的值
	a int `gone:"x-config,abc.dex,xxx|xxx"` //普通Goner会忽略GonerId(x-config)后面的字符串`abc.dex,xxx|xxx`; 而吸血鬼会用来进行"吸血"
}

func Priest(cemetery gone.Cemetery) error {
	cemetery.Bury(&ConfigVampire{}, ConfigVampireId)
	cemetery.Bury(&Demo{})
	return nil
}

func run() {
	gone.Run(Priest)
}
```

## 使用

- 启动

```go
package main

import "github.com/gone-io/gone"

func main() {
	gone.Run(func(cemetery gone.Cemetery) error {
		//下葬Goner
		return nil
	})
}

```

## 📢注意

- 尽量不用使用 struct（结构体）作为 gone 标记的字段，由于struct在golang中是值拷贝，可能导致相关依赖注入失败的情况