1. 프로세스와 스레드

   - 프로세스(Process)와 스레드(Thread)의 차이점을 설명해주세요.
     - 프로세스 → 실행 중인 프로그램의 인스턴스
       - 자원: 프로세스는 독립적인 메모리 공간(Code, Data, Stack, Heap)
       - 통신: 프로세스 간 통신(IPC)은 복잡하고 비용이 높음
     - 스레드 → 프로세스 내에서 실행되는 단위
       - 자원: 프로세스의 자원을 공유(Code, Data, Heap), 독립적인 Stack
       - 통신: 같은 프로세스 내에서 실행되므로 스레드 간 통신이 빠름
       - 하나의 스레드 오류가 같은 프로세스의 다른 스레드에 영향을 미침
   - 멀티프로세싱(Multiprocessing)과 멀티스레딩(Multithreading)의 차이점은 무엇인가요?

     멀티프로세싱:

     - 정의: 여러 프로세스를 생성하여 작업을 병렬로 처리.
     - 특징:
       - 각 프로세스는 독립적인 메모리 공간 존재
       - CPU 코어를 최대한 활용 가능
       - 프로세스 간 통신(IPC)이 필요하므로 구현이 복잡
     - 장점:
       - 독립적이므로 안정성이 높음
       - CPU 집약적인 작업에 적합
     - 단점:
       - 생성과 관리 비용이 높으며, 자원 소모가 큼

     멀티스레딩:

     - 정의: 하나의 프로세스 내에서 여러 스레드가 실행.
     - 특징:
       - 같은 메모리 공간을 공유하여 실행.
       - CPU와 I/O 작업을 효율적으로 처리.
     - 장점:
       - 자원 소모가 적고, 빠른 통신이 가능.
       - I/O 바운드 작업에 적합.
     - 단점:
       - 데이터 공유로 인한 동기화 문제가 발생
       - 하나의 스레드 오류가 프로세스 전체에 영향을 줌

   - 프로세스 상태(Process State)의 변화를 설명해보세요.
     - New → Ready: 프로세스가 생성되어 준비 상태로 진입.
     - Ready → Running: 스케줄러에 의해 CPU가 할당됨.
     - Running → Waiting: I/O 요청 등으로 대기 상태로 전환.
     - Waiting → Ready: I/O 작업이 완료되어 다시 준비 상태로 전환.
     - Running → Terminated: 작업이 완료되어 종료.
   - 문맥 전환(Context Switching)은 언제 발생하며, 왜 필요한가요?

     - 문맥 전환은 CPU가 실행 중인 프로세스를 중단하고, 다른 프로세스로 전환하는 작업
     - CPU 스케줄링, I/O 요청, 시스템 호출, 인터럽트 발생
     - 필요성 :
       - CPU 자원을 효율적으로 사용하기 위해 프로세스를 스케줄링.
       - 다중 작업 환경에서 응답성과 공정성을 보장.

   - thread-safe 하다는 의미와 설계하는 법을 설명해보세요.
     - 동기화(Synchronization):
       - Mutex: 한 번에 하나의 스레드만 공유 데이터에 접근.
       - Semaphore: 리소스의 개수를 제한하여 동시 접근 제어.
     - 불변 데이터: 데이터 변경을 방지하여 스레드 간 충돌을 피함
     - Thread-local Storage: 스레드마다 독립적인 데이터 저장소를 사용
     - Reentrant Code: 상태를 저장하지 않고, 호출할 때마다 새로운 값을 반환
     - Atomic Operations: 공유 데이터에 대한 작업을 원자적으로 수행하여 데이터 경합 방지
     - 라이브러리 사용: Thread-safe 라이브러리나 프레임워크 사용

### 번외.

언어에서는 스레드세이프 하기 위한 방법들

---

### **Java에서 스레드 세이프를 위한 방법**

Java는 멀티스레드 프로그래밍을 기본적으로 지원

### (1) **동기화(Synchronization)**

Java는 `synchronized` 키워드를 사용하여 공유 자원에 대한 접근을 동기화함

```java
public class Counter {
    private int count = 0;

    public synchronized void increment() {
        count++;
    }

    public synchronized int getCount() {
        return count;
    }
}

```

- `synchronized` 메서드는 한 번에 하나의 스레드만 접근 가능
- 객체 수준의 락이 걸리므로, Race Condition(경쟁 상태)을 방지

### (2) **Lock 객체**

Java의 `java.util.concurrent.locks` 패키지는 더 세밀한 락 제어를 제공

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Counter {
    private int count = 0;
    private final Lock lock = new ReentrantLock();

    public void increment() {
        lock.lock();
        try {
            count++;
        } finally {
            lock.unlock();
        }
    }

    public int getCount() {
        lock.lock();
        try {
            return count;
        } finally {
            lock.unlock();
        }
    }
}

```

- 락을 걸거나 해제할 시점을 직접 제어할 수 있어 유연성 up
- 조건 객체(Condition)를 활용해 스레드 간 신호 전달도 가능

### (3) **원자적 연산(Atomic Operations)**

Java는 `java.util.concurrent.atomic` 패키지를 통해 원자적 연산을 제공

```java
import java.util.concurrent.atomic.AtomicInteger;

public class Counter {
    private AtomicInteger count = new AtomicInteger();

    public void increment() {
        count.incrementAndGet();
    }

    public int getCount() {
        return count.get();
    }
}

```

- Atomic 클래스는 내부적으로 CAS(Compare-And-Swap)를 사용해 스레드 안전성을 보장
- 성능이 뛰어나며 락을 사용하지 않아 데드락이 발생 X

### (4) **Thread-safe Collections**

Java는 `java.util.concurrent` 패키지에서 스레드 세이프한 컬렉션을 제공

- **예제**:

  ```java
  import java.util.concurrent.ConcurrentHashMap;

  public class Example {
      private ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

      public void putValue(String key, Integer value) {
          map.put(key, value);
      }

      public Integer getValue(String key) {
          return map.get(key);
      }
  }

  ```

- 락 분할(Lock Striping)과 같은 최적화를 통해 성능이 좋음

---

### **JavaScript에서 스레드 세이프를 위한 방법**

JavaScript는 기본적으로 싱글 스레드 기반(이벤트 루프 모델)으로 동작

하지만 Web Workers 또는 Node.js의 Worker Threads를 사용해 멀티스레드 작업을 수행 가능 → 스레드 세이프를 달성하기 위해 JavaScript는 다음 방법을 사용

### (1) **Web Workers**

Web Workers를 사용해 멀티스레딩을 구현하며, 스레드 간 데이터를 공유하지 않고 메시지를 통해 통신

```jsx
// main.js
const worker = new Worker("worker.js");

worker.postMessage({ task: "increment", value: 1 });

worker.onmessage = (event) => {
  console.log(`Result: ${event.data}`);
};

// worker.js
let count = 0;

onmessage = (event) => {
  if (event.data.task === "increment") {
    count += event.data.value;
    postMessage(count);
  }
};
```

- Web Workers는 메모리를 공유하지 않으므로, Race Condition이 발생X
- 메시지 패싱으로 작업을 분리하여 스레드 안전성을 유지

### (2) **Node.js Worker Threads**

Node.js 환경에서 Worker Threads를 사용하여 멀티스레드 작업을 수행 가능

```jsx
const {
  Worker,
  isMainThread,
  parentPort,
  workerData,
} = require("worker_threads");

if (isMainThread) {
  const worker = new Worker(__filename, { workerData: 5 });

  worker.on("message", (result) => {
    console.log(`Result: ${result}`);
  });
} else {
  const result = workerData + 10;
  parentPort.postMessage(result);
}
```

- Worker Threads는 별도의 스레드에서 작업을 수행하며, 메시지 패싱을 통해 데이터를 주고 받음

### (3) **Atomic 데이터 구조**

JavaScript에서 `SharedArrayBuffer`와 `Atomics`를 사용해 스레드 세이프한 메모리 접근을 구현 가능

```jsx
const buffer = new SharedArrayBuffer(4);
const view = new Int32Array(buffer);

Atomics.add(view, 0, 1); // Atomic increment
console.log(Atomics.load(view, 0)); // 1
```

- `Atomics`는 여러 스레드가 공유 메모리에 접근할 때 원자적 연산을 보장

### (4) **Promise와 Async/Await**

JavaScript의 비동기 처리 방식(Promise, Async/Await)은 단일 이벤트 루프를 기반으로 동작하여 Race Condition이 발생하지 않도록 설계 가능

```jsx
let count = 0;

async function increment() {
  const current = count;
  await new Promise((resolve) => setTimeout(resolve, 100));
  count = current + 1;
  console.log(count);
}

increment();
increment();
```

위 코드에서는 Race Condition이 발생할 수 있으므로, `Atomics`나 추가 동기화를 통해 해결

---

### **3. Java와 JavaScript 스레드 세이프 방법 비교**

| 특징                       | Java                                    | JavaScript                        |
| -------------------------- | --------------------------------------- | --------------------------------- |
| **스레드 모델**            | 멀티스레드 기반                         | 싱글 스레드 기반, 멀티스레드 가능 |
| **동기화 도구**            | `synchronized`, `Lock`                  | Web Workers, Worker Threads       |
| **원자적 연산**            | `AtomicInteger`, `AtomicLong`           | `Atomics`                         |
| **메모리 공유 방식**       | 공유 메모리 기반                        | 메시지 패싱, `SharedArrayBuffer`  |
| **스레드 세이프한 컬렉션** | `ConcurrentHashMap`, `CopyOnWriteArray` | 없음                              |

---

### 결론

- **Java**는 멀티스레드 환경을 기본으로 설계되어 다양한 동기화 메커니즘과 라이브러리를 제공`synchronized`, `Lock`, `Atomic` 클래스를 주로 사용
  - Java는 전통적인 멀티스레드 환경에서 강력한 지원
- **JavaScript**는 기본적으로 싱글 스레드로 동작하므로 스레드 세이프 문제를 피하지만, 멀티스레드 작업(Web Workers, Node.js Worker Threads)에서는 메시지 패싱과 `Atomics`를 활용
  - JavaScript는 비동기적 처리와 메시지 패싱으로 안전한 멀티스레딩을 구현함
