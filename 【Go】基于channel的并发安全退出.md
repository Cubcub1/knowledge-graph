### 基于select 和default 分支实现Goroutine的退出控制

```golang
package main

import (
	"fmt"
	"time"
)

func worker(cancel chan bool) {
	for {
		select {
		default:
			fmt.Println("hello")
		case <-cancel:
			return
		}
	}
}

func main() {
	cancel := make(chan bool)
	for i := 0; i < 10; i++ {
		go worker(cancel)
	}

	time.Sleep(time.Second)
	close(cancel)
}

```

### 优化：一个channel 广播n个Goroutine的退出

```golang
package main

import (
	"fmt"
	"time"
)

func worker(cancel chan bool) {
	for {
		select {
		default:
			fmt.Println("hello")
		case <-cancel:
			return
		}
	}
}

func main() {
	cancel := make(chan bool)
	for i := 0; i < 10; i++ {
		go worker(cancel)
	}

	time.Sleep(time.Second)
	close(cancel) // 关闭管道，接受处会收到一个零值和一个可选的失败标志
}

```

### 优化：解决等待N个Goroutine安全退出

```golang
package main

import (
	"fmt"
	"sync"
	"time"
)

func worker(wg *sync.WaitGroup, cancel chan bool) {
	defer wg.Done()
	for {
		select {
		default:
			fmt.Println("hello")
		case <-cancel:
			return
		}
	}
}

func main() {
	var wg sync.WaitGroup
	cancel := make(chan bool)
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go worker(&wg, cancel)
	}

	time.Sleep(time.Second)
	close(cancel)
	wg.Wait()
}

```

### context包实现安全退出/超时退出

```golang
func worker(ctx context.Context, wg *sync.WaitGroup) error {
    defer wg.Done()

    for {
        select {
        default:
            fmt.Println("hello")
        case <-ctx.Done():
            return ctx.Err()
        }
    }
}

func main() {
    ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)

    var wg sync.WaitGroup
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go worker(ctx, &wg)
    }

    time.Sleep(time.Second)
    cancel()

    wg.Wait()
}

```