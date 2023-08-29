# Volatile in C
## How do compilers work ?
### References:
- https://medium.com/skydome-academy/how-do-compilers-work-8a1ffddf0667
  
<p align="center">
    <img src="./Images/Compiler.png" width="600px" alt="">
</p>

### Example: Optimize code 
~~~cpp
#include <stdio.h>

int main() {
    int x = 10;
    int y = 20;
    
    if (x > y) {
        printf("x is greater than y.\n");
    } else {
        printf("x is not greater than y.\n");
    }
    return 0;
}
Output: x is not greater than y.
~~~
Đoạn chương trình chỉ xảy ra ở `else` và không xảy ra ở phần `if(x > y)` được nên `compiler` sẽ tự động xóa đi đoạn code if(x > y) để tối ưu `tốc độ` và `bộ nhớ`.

## What is a Volatile Keyword in C ?
- Từ khóa `Volatile` nó thông báo cho `compiler` biết là giá trị của biến có thể thay đổi bất cứ lúc nào, ngăn chặn việc tối ưu hóa chương trình của compiler.

## Syntax
- To declare a variable volatile, include the keyword volatile before or after the data type in the variable definition.
~~~cpp
volatile int foo;
int volatile foo;

volatile int * foo;
int volatile * foo;

int * volatile foo;

int volatile * volatile foo;
~~~

## When To Use Volatile in C/C++ ?
- Memory-mapped peripheral registers
- Global variables modified by an interrupt service routine
- Global variables within a multi-threaded application

## Example
- Example 1: Giả sử bạn đang phát triển một ứng dụng điều khiển một cảm biến nhiệt độ. Cảm biến này liên tục gửi dữ liệu về nhiệt độ đo được tới biến trong mã của bạn. Bạn muốn chắc chắn rằng dữ liệu nhiệt độ luôn được cập nhật chính xác trong biến, ngay cả khi trình biên dịch có thể thực hiện các tối ưu hóa.
~~~cpp
#include <stdio.h>
#include <stdbool.h>

volatile int temperature; // Biến nhiệt độ

void readTemperatureFromSensor() {
    // Giả định hàm này đọc dữ liệu nhiệt độ từ cảm biến và gán vào biến temperature
    // Vì nó thay đổi từ bên ngoài, nên chúng ta sử dụng volatile để đảm bảo tính nhất quán.
    temperature = /* ... */;
}

int main() {
    while (true) {
        readTemperatureFromSensor();
        printf("Current temperature: %d\n", temperature);
    }
    return 0;
}
~~~
`NOTE`: Nếu không sử dụng `volatile`, trình biên dịch có thể tối ưu hóa và giữ giá trị nhiệt độ trong thanh ghi hoặc cache, không cập nhật giá trị liên tục từ cảm biến.
