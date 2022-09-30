---
layout: post
title: "GORM Go 类型转换"
date: 2022-09-29 22:00:00 +0800
tags: Database Go MySQL
---

![Type](/assets/images/2022-09-29-gorm_type_convention_1.png)

使用 GORM 时会遇到 MySQL 与 Go 数据类型不对应的情况：

- timestamp 类型在 GORM 中支持较好，但是想在 Go 中直接操作 time.Time 类型
- 在 MySQL 中直接存储 string，但是希望在 Go 中直接使用 struct/slice

可以利用 GORM 提供的接口实现序列化、反序列化时的转换：

```Go
// from MySQL to Go
type Scanner interface {
	Scan(src any) error
}

// from Go to MySQL
type Valuer interface {
	Value() (Value, error)
}
```

## 示例

```Go
import (
	"database/sql/driver"
	"fmt"
	"strings"
	"time"
)

// 想用","分割的方式在数据库某字段中存储多个字符串值(值中无','符号)
type Strs []string

func (p *Strs) Scan(val interface{}) error {
	switch v := val.(type) {
	case []uint8:
		*p = strings.Split(string(v), ",")
	default:
		return fmt.Errorf("can not parse %T %v to string slice", val, val)
	}

	return nil
}

func (p Strs) Value() (driver.Value, error) {
	return strings.Join(p, ","), nil
}


// 想在 MySQL 中以 int64 的方式存储毫秒级时间戳，在 Go 中以 time.Time 类型使用
type Time struct {
	time.Time
}

func (p *Time) Scan(val interface{}) error {
	switch v := val.(type) {
	case int64:
		p.Time = time.Unix(v/1000, v*1000*1000)
	default:
		return fmt.Errorf("can not parse %T %v to time", val, val)
	}

	return nil
}

func (p Time) Value() (driver.Value, error) {
	if p == nil {
		return int64(0), nil
	} else if p.IsZero() {
		return int64(0), nil
	}

	return p.UnixMilli(), nil
}

type MySQLEntity struct {
  Strings    Strs `gorm:"type:varchar(longtext);not null"`
  CreateTime Time `gorm:"autoCreateTime:milli;not null"`
}
```

- 如果恰好符合基本类型，如 slice 等，可以用重定义类型的方式解决
- 如果是符合某种结构体，则以 embed 方式实现更好
- 上面`Strs`的实现方案对原本只有一个字符串的情况有较好的兼容，可以在升级为多个字符串存储时平滑过渡
