# 冒泡排序算法

## 1. 冒泡排序算法 <a id="&#x5192;&#x6CE1;&#x6392;&#x5E8F;&#x7B97;&#x6CD5;"></a>

算法描述：冒泡算法，数组中前一个元素和后一个元素进行比较如果大于或者小于 前者就进行交换，最终返回最大或者最小都冒到数组的最后序列时间复杂度为 **O\(n^2\)**.

### 算法步骤 <a id="&#x7B97;&#x6CD5;&#x6B65;&#x9AA4;"></a>

* 从数组开头选择相邻两个元素进行比较，并进行交换
* 不停向后移动

```text
package sort

import "fmt"


func main() {
    arr := []int{1, 9, 10, 30, 2, 5, 45, 8, 63, 234, 12}
    fmt.Println(GetMax(arr))
    fmt.Println(BubbleSort(arr))
}


func GetMax(arr []int) int {
    for j := 1; j < len(arr); j++ {
        if arr[j-1] > arr[j] {
            arr[j-1], arr[j] = arr[j], arr[j-1]
        }
    }
    return arr[len(arr)-1]
}


func BubbleSort(arr []int) []int {
    for i := 0; i < len(arr); i++ {
        for j := i + 1; j < len(arr); j++ {
            if arr[i] > arr[j] {
                arr[i], arr[j] = arr[j], arr[i]
            }
        }
    }
    return arr
}
```

