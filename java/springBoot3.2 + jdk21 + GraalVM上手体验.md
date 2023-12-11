# springBoot3.2 + jdk21 + GraalVM上手体验
SpringBoot2.x官方已经停止维护了，jdk8这次真的得换了🤣


>- 可以参考官方文章进行体验：https://spring.io/blog/2023/09/09/all-together-now-spring-boot-3-2-graalvm-native-images-java-21-and-virtual
>
>- 通过官方快速得到一个基于jdk21的项目：https://start.spring.io/

## 快速体验（二进制部署）

```java
@RestController
@SpringBootApplication
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}

	@GetMapping("/customers")
	Collection<Customer> customers() {
		return Set.of(new Customer(1, "A"), new Customer(2, "B"), new Customer(3, "C"));
	}

	record Customer(Integer id, String name) {
	}
}
```

启动非常快，秒启动

![image-20231201173556211](https://img-blog.csdnimg.cn/img_convert/6177751a91e71fbf42a294d5c0d712e0.png)

压测环境内存占用大概`70MB`左右，空闲时在`20MB`左右（由于直接打成二进制文件了，不能再使用jconsole、arthas之类的进行监控了），性能上由于不需要JVM预热，性能启动即巅峰。

```shell
$ ab -c 50 -n 10000 http://localhost:8080/customers
Server Software:
Server Hostname:        localhost
Server Port:            8080

Document Path:          /customers
Document Length:        61 bytes

Concurrency Level:      50
Time taken for tests:   1.413 seconds
Complete requests:      10000
Failed requests:        0
Total transferred:      1660000 bytes
HTML transferred:       610000 bytes
Requests per second:    7076.39 [#/sec] (mean)
Time per request:       7.066 [ms] (mean)
Time per request:       0.141 [ms] (mean, across all concurrent requests)
Transfer rate:          1147.15 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    2   8.0      2     144
Processing:     1    5   6.7      4     147
Waiting:        0    4   5.6      3     145
Total:          1    7  10.4      6     149
```

![image-20231201173732084](https://img-blog.csdnimg.cn/img_convert/1ce803696e706e607860c9c1ed3a8a16.png)

## 快速体验（jar部署）

jar包占用只有19MB，已经不能算是小胖jar了😊

![image-20231201175815773](https://img-blog.csdnimg.cn/img_convert/c5234d582cfb7b3185f87db7661f7d03.png)

内存占用在压测时大概在200MB左右，空闲时在160MB左右。性能显然也不是启动即巅峰，可以看出其实还是需要进行JVM预热才能达到性能巅峰的

```shell
$ ab -c 50 -n 10000 http://localhost:8080/customers
Server Software:
Server Hostname:        localhost
Server Port:            8080

Document Path:          /customers
Document Length:        61 bytes

Concurrency Level:      50
Time taken for tests:   17.930 seconds
Complete requests:      10000
Failed requests:        0
Total transferred:      1660000 bytes
HTML transferred:       610000 bytes
Requests per second:    557.72 [#/sec] (mean)
Time per request:       89.651 [ms] (mean)
Time per request:       1.793 [ms] (mean, across all concurrent requests)
Transfer rate:          90.41 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0   38 430.2      2    7004
Processing:     0   14  90.4      8    1773
Waiting:        0   12  88.7      6    1771
Total:          1   53 439.0     10    7011
```

![image-20231201180038447](https://img-blog.csdnimg.cn/img_convert/3dedb4ca21d5c5212ba2cb343571f02d.png)



## 对比golang

```go
package main

import (
	"encoding/json"
	"flag"
	"fmt"
	"net/http"
)

var port = flag.String("p", "8080", "please input port")

func main() {
	http.HandleFunc("/customers", func(writer http.ResponseWriter, request *http.Request) {
		data, _ := json.Marshal(request.URL)
		writer.Write(data)
	})
	e := make(chan error)
	go func() {
		e <- fmt.Errorf("error[%v]", http.ListenAndServe(":"+*port, nil))
	}()
	fmt.Println("http 服务器启动...")
	fmt.Println(<-e)
}
```

这里golang没有使用框架，仅使用标准库，所以内存占用较低，仅`10MB`左右，不过即使使用Gin之类的web框架，内存也不会超过20MB

```shell
$ ab -c 50 -n 10000 http://localhost:8080/customers
Server Software:
Server Hostname:        localhost
Server Port:            8080

Document Path:          /customers
Document Length:        161 bytes

Concurrency Level:      50
Time taken for tests:   1.380 seconds
Complete requests:      10000
Failed requests:        0
Total transferred:      2790000 bytes
HTML transferred:       1610000 bytes
Requests per second:    7247.68 [#/sec] (mean)
Time per request:       6.899 [ms] (mean)
Time per request:       0.138 [ms] (mean, across all concurrent requests)
Transfer rate:          1974.71 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    2  16.5      2     459
Processing:     0    4  27.9      2     460
Waiting:        0    2  10.5      2     459
Total:          1    7  32.3      4     462
```

![image-20231201174441704](https://img-blog.csdnimg.cn/img_convert/87713526e30e27ca4d410070daafb7ee.png)

## 对比Rust

```toml
[dependencies]
actix-web = "4"
```

```rust
use actix_web::{get, App, HttpRequest, HttpResponse, HttpServer, Responder};

#[get("/customers")]
async fn echo(req: HttpRequest) -> impl Responder {
    let url = req.uri().to_string();
    HttpResponse::Ok().body(url)
}


#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .service(echo)
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

Actix-web空闲时内存占用大概`3MB`左右，压测时占用大概`6MB`左右

```rust
$ ab -c 50 -n 10000 http://localhost:8080/customers
Server Software:
Server Hostname:        127.0.0.1
Server Port:            8080

Document Path:          /customers
Document Length:        10 bytes

Concurrency Level:      50
Time taken for tests:   1.091 seconds
Complete requests:      10000
Failed requests:        0
Total transferred:      860000 bytes
HTML transferred:       100000 bytes
Requests per second:    9163.48 [#/sec] (mean)
Time per request:       5.456 [ms] (mean)
Time per request:       0.109 [ms] (mean, across all concurrent requests)
Transfer rate:          769.59 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    2  11.0      2     189
Processing:     0    3   7.0      3     190
Waiting:        0    2   7.0      2     189
Total:          2    5  13.1      4     193
```

![image-20231204115913574](https://cdn.fengxianhub.top/resources-master/image-20231204115913574.png)

rust虽然有非常厉害的`零成本抽象`，但作为代价其编译时间会比较长（在实际项目中真的特别长😢）

```shell
$ time cargo build
cargo build  213.00s user 23.08s system 258% cpu 1:31.39 total
```

## 结论

AOT-processed已经相对成熟，效果可以说非常惊艳，解决了`JVM`启动慢、需要预热、内存占用大等问题。

美中不足的是编译速度非常慢，笔者电脑是2017款mac book pro编译花费大概15分钟左右

```shell
Finished generating 'demo' in 14m 33s.
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  15:45 min
[INFO] Finished at: 2023-12-01T17:00:21+08:00
[INFO] ------------------------------------------------------------------------
```

------------
[INFO] Total time:  15:45 min
[INFO] Finished at: 2023-12-01T17:00:21+08:00
[INFO] ------------------------------------------------------------------------
```

可以看出java在云原生大环境下已经取得了不错的进步的
