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
- Keywords `Volatile` nó thông báo cho `compiler` biết là giá trị của biến có thể thay đổi bất cứ lúc nào, ngăn chặn việc `tối ưu` hóa chương trình của compiler.
  
=> Sử dụng `volatile` đảm bảo rằng giá trị biến sẽ không bị `optimization` do `compiler`, luôn được đọc và ghi trực tiếp từ `memory`.

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
- Memory-mapped peripheral registers.
- Global variables modified by an interrupt service routine.
- Global variables within a multi-threaded application.

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

- Example 2: Memory-mapped peripheral registers.
  
  Giả sử ta đang viết mã cho một vi điều khiển nhúng và  muốn đọc giá trị từ thanh ghi ngoại vi điều khiển nút nhấn (button) để biết xem nút đã được nhấn hay chưa.
~~~cpp
#include <stdio.h>

volatile unsigned int *button_register = (volatile unsigned int *)0x20000000;

int main() {
    while (1) {
        if (*button_register & 0x01) {
            // Nếu bit thấp nhất của thanh ghi bằng 1, tức là nút đã được nhấn.
            // Thực hiện các hành động tương ứng ở đây.
        }
    }
    return 0;
}
~~~
`Note:` Nếu không sử dụng `Volatile`, compiler có thể quyết định tối ưu hóa việc truy cập đến biến này bằng cách `lưu trữ giá trị` của biến trong `thanh ghi CPU` và `bộ nhớ cache` thay vì đọc giá trị từ bộ nhớ thực sự => dẫn tới kết quả không mong muốn.

- Example 3: Global variables within a multi-threaded application.

Giả sử bạn đang viết một ứng dụng đa luồng để quản lý tài khoản ngân hàng. Bạn có một global variable `balance` để lưu số dư trong tài khoản.
~~~cpp
#include <stdio.h>
#include <pthread.h>

int balance = 1000;

void* withdraw(void* arg) {
    int amount = *(int*)arg;
    if (balance >= amount) {
        balance -= amount;
        printf("Withdrawn %d. New balance: %d\n", amount, balance);
    } else {
        printf("Insufficient balance!\n");
    }
    return NULL;
}

int main() {
    pthread_t thread1, thread2;
    int amount1 = 800, amount2 = 600;

    pthread_create(&thread1, NULL, withdraw, &amount1);
    pthread_create(&thread2, NULL, withdraw, &amount2);

    pthread_join(thread1, NULL);
    pthread_join(thread2, NULL);

    printf("Final balance: %d\n", balance);
    return 0;
}
~~~
Trong ví dụ này: 
 - Hàm `withdraw` là một hàm chạy trong thread đồng thời. Nó rút tiền từ tài khoản nếu số dư đủ.
 - Trong hàm `main`, bạn tạo hai thread để rút tiền từ tài khoản với hai số tiền khác nhau.

`Vấn đề:` Tuy có vẻ như chương trình trên hoạt động đúng, nhưng thực tế có thể xảy ra vấn đề do các `thread` hoạt động đồng thời và thay đổi `balance`:
 - `Thread 1` và `thread 2` có thể đồng thời kiểm tra điều kiện `balance >= amount`, và cả hai đều thấy số dư đủ để rút.
 - Sau đó, cả hai thread `cùng lúc` cập nhật balance bằng cách trừ amount từ balance.
`Kết quả:` balance bị giảm quá mức do việc cập nhật không đồng bộ => blance = -400.
`Giải quyết:`
 - Do `blance` là global variable -> Không có cơ chế đồng bộ hóa.
 - Dùng các thuật toán đồng bộ hóa như mutex, sermaphone,...
