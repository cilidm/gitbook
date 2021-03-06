# 验证器

## 1. 验证器 <a id="&#x9A8C;&#x8BC1;&#x5668;"></a>

### 1.1.1. 描述 <a id="&#x63CF;&#x8FF0;"></a>

ozzo-validation是一个Go软件包，提供可配置和可扩展的数据验证功能。它具有以下功能：

* 使用常规的编程构造而不是容易出错的构造标记来指定应如何验证数据。
* 可以验证不同类型的数据，例如结构，字符串，字节片，片，映射，数组。
* 只要实现Validatable接口，就可以验证自定义数据类型。
* 可以验证实现该sql.Valuer接口的数据类型（例如sql.NullString）。
* 可自定义且格式正确的验证错误。
* 立即提供丰富的验证规则集。
* 创建和使用自定义验证规则非常容易。
* github地址： [https://github.com/go-ozzo/ozzo-validation](https://github.com/go-ozzo/ozzo-validation)

### 1.1.2. 要求 <a id="&#x8981;&#x6C42;"></a>

达到1.13或更高。

### 1.1.3. 入门 <a id="&#x5165;&#x95E8;"></a>

ozzo验证包主要包括一组验证规则和两种验证方法。您使用验证规则来描述应如何将值视为有效，然后调用validation.Validate\(\) 或validation.ValidateStruct\(\)验证值。

### 1.1.4. 安装 <a id="&#x5B89;&#x88C5;"></a>

运行以下命令以安装软件包：

```text
    go get github.com/go-ozzo/ozzo-validation/v3
    go get github.com/go-ozzo/ozzo-validation/v3/is
```

### 1.1.5. 验证简单值 <a id="&#x9A8C;&#x8BC1;&#x7B80;&#x5355;&#x503C;"></a>

对于简单的值（例如字符串或整数），可以使用validation.Validate\(\)它进行验证。例如，

```text
package main

import (
    "fmt"

    "github.com/go-ozzo/ozzo-validation/v3"
    "github.com/go-ozzo/ozzo-validation/v3/is"
)

func main() {
    data := "example"
    err := validation.Validate(data,
        validation.Required,       
        validation.Length(5, 100), 
        is.URL,                    
    )
    fmt.Println(err)
    
    
}
```

该方法validation.Validate\(\)将按照列出的顺序运行规则。如果规则未能通过验证，则该方法将返回相应的错误，并跳过其余规则。如果值通过所有验证规则，则该方法将返回nil。

### 1.1.6. 验证结构 <a id="&#x9A8C;&#x8BC1;&#x7ED3;&#x6784;"></a>

对于结构值，通常需要检查其字段是否有效。例如，在RESTful应用程序中，您可以将请求有效内容解组为结构，然后验证结构字段。如果一个或多个字段无效，则可能需要获取描述哪些字段无效的错误。您可以使用validation.ValidateStruct\(\) 以实现此目的。单个结构可以具有多个字段的规则，并且一个字段可以与多个规则关联。例如，

```text
package main

import (
    "fmt"
    "regexp"

    "github.com/go-ozzo/ozzo-validation/v3"
    "github.com/go-ozzo/ozzo-validation/v3/is"
)

type Address struct {
    Street string
    City   string
    State  string
    Zip    string
}

func (a Address) Validate() error {
    return validation.ValidateStruct(&a,
        
        validation.Field(&a.Street, validation.Required, validation.Length(5, 50)),
        
        validation.Field(&a.City, validation.Required, validation.Length(5, 50)),
        
        validation.Field(&a.State, validation.Required, validation.Match(regexp.MustCompile("^[A-Z]{2}$"))),
        
        validation.Field(&a.Zip, validation.Required, validation.Match(regexp.MustCompile("^[0-9]{5}$"))),
    )
}

func main() {
    a := Address{
        Street: "123",
        City:   "Unknown",
        State:  "Virginia",
        Zip:    "12345",
    }

    err := a.Validate()
    fmt.Println(err)
    
    
}
```

请注意，在调用validation.ValidateStruct以验证结构时，应将指向结构的指针（而不是结构本身）传递给方法。同样，在调用validation.Field以指定结构字段的规则时，应使用指向结构字段的指针。

执行结构验证时，将按照在中指定的顺序验证字段ValidateStruct。并且，当每个字段都经过验证时，其规则也会按照与该字段关联的顺序进行评估。如果规则失败，则会为该字段记录一个错误，并且验证将在下一个字段继续。

### 1.1.7. 验证错误 <a id="&#x9A8C;&#x8BC1;&#x9519;&#x8BEF;"></a>

该validation.ValidateStruct方法返回在结构字段中发现的验证错误，这些验证错误validation.Errors 是字段及其对应错误的映射。如果验证通过，则返回Nil。

默认情况下，validation.Errors使用命名的struct标记json来确定应使用哪些名称来表示无效字段。该类型还实现了json.Marshaler接口，以便可以将其编组为适当的JSON对象。例如，

```text
type Address struct {
    Street string `json:"street"`
    City   string `json:"city"`
    State  string `json:"state"`
    Zip    string `json:"zip"`
}



err := a.Validate()
b, _ := json.Marshal(err)
fmt.Println(string(b))


```

您可以修改validation.ErrorTag以使用其他结构标记名称。

如果您不喜欢ValidateStruct根据结构字段名称或相应的标记值确定错误键的魔术，则可以使用以下替代方法：

```text
c := Customer{
    Name:  "Qiang Xue",
    Email: "q",
    Address: Address{
        State:  "Virginia",
    },
}

err := validation.Errors{
    "name": validation.Validate(c.Name, validation.Required, validation.Length(5, 20)),
    "email": validation.Validate(c.Name, validation.Required, is.Email),
    "zip": validation.Validate(c.Address.Zip, validation.Required, validation.Match(regexp.MustCompile("^[0-9]{5}$"))),
}.Filter()
fmt.Println(err)


```

在上面的示例中，我们validation.Errors通过名称列表和相应的验证结果来构建。最后，我们要求Errors.Filter\(\)从Errors所有与成功验证结果相对应的nil中删除。如果Errors为空，该方法将返回nil 。

上面的方法非常灵活，因为它允许您自由地建立验证错误结构。您可以使用它来验证结构和非结构值。与用于ValidateStruct验证结构相比，它的缺点是您必须冗余地指定错误密钥，同时ValidateStruct可以自动找出它们。

### 1.1.8. 内部错误 <a id="&#x5185;&#x90E8;&#x9519;&#x8BEF;"></a>

内部错误与验证错误的不同之处在于，内部错误是由代码故障（例如，验证器在远程服务中断时进行远程调用以验证某些数据）引起的，而不是由正在验证的数据引起的。在数据验证期间发生内部错误时，您可以允许用户重新提交相同的数据以再次执行验证，以希望程序恢复运行。另一方面，如果由于数据错误导致数据验证失败，则用户通常不应再次重新提交相同的数据。

为了将内部错误与验证错误区分开，当验证程序中发生内部错误时，请validation.InternalError通过调用将其包装validation.NewInternalError\(\)。然后，验证者的用户可以检查返回的错误是否是内部错误。例如，

```text
if err := a.Validate(); err != nil {
    if e, ok := err.(validation.InternalError); ok {
        
        fmt.Println(e.InternalError())
    }
}
```

### 1.1.9. 可验证的类型 <a id="&#x53EF;&#x9A8C;&#x8BC1;&#x7684;&#x7C7B;&#x578B;"></a>

如果类型实现了validation.Validatable接口，则该类型是有效的。

当validation.Validate用于验证有效值时，如果在给定的验证规则中未发现任何错误，它将进一步调用该值的Validate\(\)方法。

同样，在validation.ValidateStruct验证类型可验证的结构字段时，它将Validate在通过列出的规则后调用该字段的方法。

> 注意：在实现时validation.Validatable，请勿调用validation.Validate\(\)验证其原始类型的值，因为这将导致无限循环。例如，如果你定义一个新类型MyString作为string 和实施validation.Validatable为MyString，在内部Validate\(\)功能，你应该值转换为string调用之前先validation.Validate\(\)对其进行验证。

在以下示例中，由于实现 ，的Address字段Customer是有效的。因此，在使用验证结构时，验证将“潜入”该领域。Addressvalidation.ValidatableCustomervalidation.ValidateStructAddress

```text
type Customer struct {
    Name    string
    Gender  string
    Email   string
    Address Address
}

func (c Customer) Validate() error {
    return validation.ValidateStruct(&c,
        
        validation.Field(&c.Name, validation.Required, validation.Length(5, 20)),
        
        validation.Field(&c.Gender, validation.In("Female", "Male")),
        
        validation.Field(&c.Email, validation.Required, is.Email),
        
        validation.Field(&c.Address),
    )
}

c := Customer{
    Name:  "Qiang Xue",
    Email: "q",
    Address: Address{
        Street: "123 Main Street",
        City:   "Unknown",
        State:  "Virginia",
        Zip:    "12345",
    },
}

err := c.Validate()
fmt.Println(err)


```

有时，您可能要跳过对类型Validate方法的调用。为此，只需将validation.Skip规则与要验证的值相关联即可。

### 1.1.10. Maps/Slices/Arrays 可验证 <a id="mapsslicesarrays-&#x53EF;&#x9A8C;&#x8BC1;"></a>

验证其元素类型实现validation.Validatable接口的可迭代（映射，切片或数组）时，该validation.Validate方法将调用Validate每个非null元素的方法。将返回元素的验证错误，因为validation.Errors它将无效元素的键映射到其相应的验证错误。例如，

```text
addresses := []Address{
    Address{State: "MD", Zip: "12345"},
    Address{Street: "123 Main St", City: "Vienna", State: "VA", Zip: "12345"},
    Address{City: "Unknown", State: "NC", Zip: "123"},
}
err := validation.Validate(addresses)
fmt.Println(err)


```

当validation.ValidateStruct用于验证结构时，以上验证过程也适用于那些结构字段，这些结构字段是可验证对象的映射/切片/数组。

Each

另一种选择是使用该validation.Each方法。此方法允许您为结构中的可迭代对象定义规则。

```text
type Customer struct {
    Name      string
    Emails    []string
}

func (c Customer) Validate() error {
    return validation.ValidateStruct(&c,
        
        validation.Field(&c.Name, validation.Required, validation.Length(5, 20)),
        
        validation.Field(&c.Emails, validation.Each(is.Email)),
    )
}

c := Customer{
    Name:   "Qiang Xue",
    Emails: []Email{
        "valid@example.com",
        "invalid",
    },
}

err := c.Validate()
fmt.Println(err)


```

### 1.1.11. 指针 <a id="&#x6307;&#x9488;"></a>

当要验证的值是指针时，大多数验证规则将验证指针指向的实际值。如果指针为零，则这些规则将跳过验证。

validation.Required和validation.NotNil规则是一个例外。当指针为零时，它们将报告验证错误。

### 1.1.12. 类型实施 sql.Valuer <a id="&#x7C7B;&#x578B;&#x5B9E;&#x65BD;-sqlvaluer"></a>

如果数据类型实现了sql.Valuer接口（例如sql.NullString），则内置的验证规则将正确处理该接口。特别是，当规则验证此类数据时，它将调用该Value\(\)方法并验证返回的值。

### 1.1.13. 必需vs.非零 <a id="&#x5FC5;&#x9700;vs&#x975E;&#x96F6;"></a>

验证输入值时，有两种有关检查是否提供输入值的方案。

在第一种情况下，如果未输入输入值或将其输入为零值（例如，空字符串，零整数），则认为输入值丢失。validation.Required在这种情况下，您可以使用规则。

在第二种情况下，仅当未输入输入值时，才将其视为丢失。通常在这种情况下使用指针字段，以便您可以通过检查指针是否为nil来检测是否输入了值。您可以使用validation.NotNil规则来确保输入值（即使它是零值）。

### 1.1.14. 嵌入式结构 <a id="&#x5D4C;&#x5165;&#x5F0F;&#x7ED3;&#x6784;"></a>

该validation.ValidateStruct方法将正确验证包含嵌入式结构的结构。特别是，将嵌入式结构的字段视为直接属于包含的结构。例如，

```text
type Employee struct {
    Name string
}

func ()

type Manager struct {
    Employee
    Level int
}

m := Manager{}
err := validation.ValidateStruct(&m,
    validation.Field(&m.Name, validation.Required),
    validation.Field(&m.Level, validation.Required),
)
fmt.Println(err)


```

在上面的代码中，我们用于&m.Name指定Name嵌入式struct 的字段的验证Employee。验证错误Name用作与该Name字段关联的错误的键，就好像Name一个字段直接属于Manager。

如果Employee实现该validation.Validatable接口，我们还可以使用以下代码来验证 Manager，它会生成相同的验证结果：

```text
func (e Employee) Validate() error {
    return validation.ValidateStruct(&e,
        validation.Field(&e.Name, validation.Required),
    )
}

err := validation.ValidateStruct(&m,
    validation.Field(&m.Employee),
    validation.Field(&m.Level, validation.Required),
)
fmt.Println(err)


```

### 1.1.15. 内置验证规则 <a id="&#x5185;&#x7F6E;&#x9A8C;&#x8BC1;&#x89C4;&#x5219;"></a>

validation软件包中提供了以下规则：

* `In(...interface{})：`检查是否可以在给定的值列表中找到一个值。
* `Length(min, max int)`：检查值的长度是否在指定范围内。此规则仅应用于验证字符串，切片，映射和数组。
* `RuneLength(min, max int)：`检查字符串的长度是否在指定范围内。此规则与Length其他规则类似，不同之处在于，当要验证的值是字符串时，它将检查其符文长度而不是字节长度。
* `Min(min interface{})和Max(max interface{})：`检查值是否在指定范围内。这两个规则仅应用于验证int，uint，float和time.Time类型。
* `Match(*regexp.Regexp)：`检查值是否与指定的正则表达式匹配。此规则仅应用于字符串和字节片。
* `Date(layout string)：`检查字符串值是否是布局指定格式的日期。通过调用Min\(\)和/或Max\(\)，您可以额外检查日期是否在指定范围内。
* `Required：`检查值是否不为空（nil也不为零）。
* `NotNil：`检查指针值是否不为nil。非指针值被视为有效。
* `NilOrNotEmpty：`检查值是nil指针还是非空值。这与Required将nil指针视为有效指针不同。
* `Skip：`这是一条特殊规则，用于指示应跳过其后的所有规则（包括嵌套规则）。
* `MultipleOf：`检查该值是否为指定范围的倍数。
* `Each(rules ...Rule)：`使用其他规则检查可迭代（映射/切片/数组）中的元素。 该is子包提供了可用于检查的常用字符串的验证规则列表，如果值满足一定要求的格式。请注意，这些规则仅处理字符串和字节片，并且如果字符串或字节片为空，则视为有效。您可以使用Required规则来确保值不为空。以下是该is软件包提供的规则的完整列表：
* `Email：`验证字符串是否为电子邮件
* `URL：`验证字符串是否为有效网址
* `RequestURL：`验证字符串是否为有效的请求网址
* `RequestURI：`验证字符串是否为有效的请求URI
* `Alpha：`验证字符串是否仅包含英文字母（a-zA-Z）
* `Digit：`验证字符串是否仅包含数字（0-9）
* `Alphanumeric：`验证字符串是否仅包含英文字母和数字（a-zA-Z0-9）
* `UTFLetter：`验证字符串是否仅包含unicode字母
* `UTFDigit：`验证字符串是否仅包含unicode十进制数字
* `UTFLetterNumeric：`验证字符串是否仅包含unicode字母和数字
* `UTFNumeric：`验证字符串是否仅包含unicode数字字符（类别N）
* `LowerCase：`验证字符串是否仅包含小写unicode字母
* `UpperCase：`验证字符串是否仅包含大写unicode字母
* `Hexadecimal：`验证字符串是否为有效的十六进制数字
* `HexColor：`验证字符串是否为有效的十六进制颜色代码
* `RGBColor：`验证字符串是否为rgb（R，G，B）形式的有效RGB颜色
* `Int：`验证字符串是否为有效的整数
* `Float：`验证字符串是否为浮点数
* `UUIDv3：`验证字符串是否为有效的版本3 UUID
* `UUIDv4：`验证字符串是否为有效的版本4 UUID
* `UUIDv5：`验证字符串是否为有效的版本5 UUID
* `UUID：`验证字符串是否为有效的UUID
* `CreditCard：`验证字符串是否为有效的信用卡号
* `ISBN10：`验证字符串是否为ISBN版本10
* `ISBN13：`验证字符串是否为ISBN版本13
* `ISBN：`验证字符串是否为ISBN（版本10或13）
* `JSON：`验证字符串是否为有效JSON格式
* `ASCII：`验证字符串是否仅包含ASCII字符
* `PrintableASCII：`验证字符串是否仅包含可打印的ASCII字符
* `Multibyte：`验证字符串是否包含多字节字符
* `FullWidth：`验证字符串是否包含全角字符
* `HalfWidth：`验证字符串是否包含半角字符
* `VariableWidth：`验证字符串是否同时包含全角和半角字符
* `Base64：`验证字符串是否在Base64中编码
* `DataURI：`验证字符串是否为有效的base64编码数据URI
* `E164：`验证字符串是否为有效的E164电话号码（+19251232233）
* `CountryCode2：`验证字符串是否为有效的ISO3166 Alpha 2国家/地区代码
* `CountryCode3：`验证字符串是否为有效的ISO3166 Alpha 3国家/地区代码
* `DialString：`验证字符串是否是可以传递给Dial（）的有效拨号字符串
* `MAC：`验证字符串是否为MAC地址
* `IP：`验证字符串是否为有效的IP地址（版本4或6）
* `IPv4：`验证字符串是否为有效的版本4 IP地址
* `IPv6：`验证字符串是否为有效的版本6 IP地址
* `Subdomain：`验证字符串是否为有效的子域
* `Domain：`验证字符串是否为有效域
* `DNSName：`验证字符串是否为有效的DNS名称
* `Host：`验证字符串是有效的IP（v4和v6）还是有效的DNS名称
* `Port：`验证字符串是否为有效的端口号
* `MongoID：`验证字符串是否为有效的Mongo ID
* `Latitude：`验证字符串是否为有效纬度
* `Longitude：`验证字符串是否为有效经度
* `SSN：`验证字符串是否为社会安全号码（SSN）
* `Semver：`验证字符串是否是有效的语义版本

### 1.1.16. 自定义错误消息 <a id="&#x81EA;&#x5B9A;&#x4E49;&#x9519;&#x8BEF;&#x6D88;&#x606F;"></a>

所有内置的验证规则均允许您自定义错误消息。为此，只需调用Error\(\)规则的方法。例如，

```text
data := "2123"
err := validation.Validate(data,
    validation.Required.Error("is required"),
    validation.Match(regexp.MustCompile("^[0-9]{5}$")).Error("must be a string with five digits"),
)
fmt.Println(err)


```

### 1.1.17. 创建自定义规则 <a id="&#x521B;&#x5EFA;&#x81EA;&#x5B9A;&#x4E49;&#x89C4;&#x5219;"></a>

创建自定义规则就像实现validation.Rule界面一样简单。接口包含如下所示的单个方法，该方法应验证值并返回验证错误（如果有）：

```text

Validate(value interface{}) error
```

如果您已经具有与上面所示相同的签名的函数，则可以调用validation.By\(\)将其转换为验证规则。例如，

```text
func checkAbc(value interface{}) error {
    s, _ := value.(string)
    if s != "abc" {
        return errors.New("must be abc")
    }
    return nil
}

err := validation.Validate("xyz", validation.By(checkAbc))
fmt.Println(err)

```

如果您的验证功能使用其他参数，则可以使用以下闭合技巧：

```text
func stringEquals(str string) validation.RuleFunc {
    return func(value interface{}) error {
        s, _ := value.(string)
        if s != str {
            return errors.New("unexpected string")
        }
        return nil
    }
}

err := validation.Validate("xyz", validation.By(stringEquals("abc")))
fmt.Println(err)

```

### 1.1.18. 规则组 <a id="&#x89C4;&#x5219;&#x7EC4;"></a>

在多个地方使用多个规则的组合时，可以使用以下技巧来创建规则组，以使代码更易于维护。

```text
var NameRule = []validation.Rule{
    validation.Required,
    validation.Length(5, 20),
}

type User struct {
    FirstName string
    LastName  string
}

func (u User) Validate() error {
    return validation.ValidateStruct(&u,
        validation.Field(&u.FirstName, NameRule...),
        validation.Field(&u.LastName, NameRule...),
    )
}
```

在上面的示例中，我们创建了一个NameRule包含两个验证规则的规则组。然后，我们使用此规则组同时验证FirstName和LastName。

### 1.1.19. 上下文感知验证 <a id="&#x4E0A;&#x4E0B;&#x6587;&#x611F;&#x77E5;&#x9A8C;&#x8BC1;"></a>

虽然大多数验证规则是独立的，但某些规则可能会动态依赖于上下文。规则可以实现 validation.RuleWithContext接口以支持所谓的上下文感知验证。

要使用上下文验证任意值，请调用validation.ValidateWithContext\(\)。该context.Conext参数将被传递下去，以实现这些规则validation.RuleWithContext。

要使用上下文验证结构的字段，请调用validation.ValidateStructWithContext\(\)。

您可以通过同时实现validation.Rule和来定义上下文感知规则validation.RuleWithContext。您还可以使用validation.WithContext\(\)将功能转换为上下文感知规则。例如，

```text
rule := validation.WithContext(func(ctx context.Context, value interface{}) error {
    if ctx.Value("secret") == value.(string) {
        return nil
    }
    return errors.New("value incorrect")
})
value := "xyz"
ctx := context.WithValue(context.Background(), "secret", "example")
err := validation.ValidateWithContext(ctx, value, rule)
fmt.Println(err)

```

执行上下文感知验证时，如果未实现规则validation.RuleWithContext， validation.Rule则将改用它。

