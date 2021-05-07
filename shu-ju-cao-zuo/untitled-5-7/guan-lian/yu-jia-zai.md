# 预加载

## 1. 预加载 <a id="&#x9884;&#x52A0;&#x8F7D;"></a>

### 1.1. 预加载 <a id="&#x9884;&#x52A0;&#x8F7D;_1"></a>

```text
db.Preload("Orders").Find(&users)
//// SELECT * FROM users;
//// SELECT * FROM orders WHERE user_id IN (1,2,3,4);

db.Preload("Orders", "state NOT IN (?)", "cancelled").Find(&users)
//// SELECT * FROM users;
//// SELECT * FROM orders WHERE user_id IN (1,2,3,4) AND state NOT IN ('cancelled');

db.Where("state = ?", "active").Preload("Orders", "state NOT IN (?)", "cancelled").Find(&users)
//// SELECT * FROM users WHERE state = 'active';
//// SELECT * FROM orders WHERE user_id IN (1,2) AND state NOT IN ('cancelled');

db.Preload("Orders").Preload("Profile").Preload("Role").Find(&users)
//// SELECT * FROM users;
//// SELECT * FROM orders WHERE user_id IN (1,2,3,4); // has many
//// SELECT * FROM profiles WHERE user_id IN (1,2,3,4); // has one
//// SELECT * FROM roles WHERE id IN (4,5,6); // belongs to
```

### 1.2. 自动预加载 <a id="&#x81EA;&#x52A8;&#x9884;&#x52A0;&#x8F7D;"></a>

始终自动预加载的关联

```text
type User struct {
  gorm.Model
  Name       string
  CompanyID  uint
  Company    Company `gorm:"PRELOAD:false"` //没有预加载
  Role       Role                           // 已经预加载
}

db.Set("gorm:auto_preload", true).Find(&users)
```

### 1.3. 嵌套预加载 <a id="&#x5D4C;&#x5957;&#x9884;&#x52A0;&#x8F7D;"></a>

```text
db.Preload("Orders.OrderItems").Find(&users)
db.Preload("Orders", "state = ?", "paid").Preload("Orders.OrderItems").Find(&users)
```

### 1.4. 自定义预加载 SQL <a id="&#x81EA;&#x5B9A;&#x4E49;&#x9884;&#x52A0;&#x8F7D;-sql"></a>

您可以通过传入`func（db * gorm.DB）* gorm.DB`来自定义预加载SQL，例如：

```text
db.Preload("Orders", func(db *gorm.DB) *gorm.DB {
    return db.Order("orders.amount DESC")
}).Find(&users)
//// SELECT * FROM users;
//// SELECT * FROM orders WHERE user_id IN (1,2,3,4) order by orders.amount DESC;
```

