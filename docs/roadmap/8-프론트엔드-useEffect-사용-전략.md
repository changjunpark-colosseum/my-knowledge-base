---
id: frontend-use-effect-strategy
title: 프론트엔드 useEffect 사용 전략
type: principle
status: accepted
scope: frontend
tags: [react, use-effect, side-effect, derived-state]
priority: high
updated: 2026-06-30
applies_to: [frontend, cgkr_oncall]
---

# 8. 프론트엔드 useEffect 사용 전략

## 목적

`useEffect`를 상태 동기화나 계산 로직의 기본 도구처럼 남발하지 않고, 진짜 부수효과를 관리하는 용도로 제한한다.

`useEffect`는 렌더링 결과를 외부 시스템과 동기화하기 위한 escape hatch다. 렌더링 중 계산할 수 있는 값, 사용자 이벤트에서 처리할 수 있는 로직, 서버 상태 조회는 기본적으로 `useEffect`의 책임이 아니다.

## 핵심 문장

`useEffect`는 부수효과를 관리하는 곳이다.

프론트엔드에서 부수효과란 React 렌더링 바깥의 외부 시스템과 상호작용하는 일이다. 네트워크 요청, 소켓 연결, DOM 직접 조작, timer, browser storage, 외부 라이브러리 구독처럼 React 상태만으로 설명되지 않는 작업이 여기에 해당한다.

## 프론트엔드에서의 부수효과

| 부수효과 | 예시 |
| --- | --- |
| Network | API 요청, polling |
| Subscription | WebSocket, SSE, event listener |
| Browser API | `localStorage`, `sessionStorage`, clipboard |
| Timer | `setTimeout`, `setInterval` |
| DOM 직접 조작 | focus, scroll, measurement |
| External Library | chart, map, printer SDK, barcode scanner SDK |
| Analytics/Monitoring | logging, tracking, Sentry context |

## 기본 원칙

1. 렌더링 중 계산 가능한 값은 `useEffect`로 만들지 않는다.
2. 사용자 이벤트에서 처리할 수 있는 로직은 event handler에서 처리한다.
3. 서버 상태 조회는 React Query를 우선 사용한다.
4. props/state를 다른 state로 복사하기 위해 `useEffect`를 쓰지 않는다.
5. 외부 시스템 구독은 cleanup을 반드시 작성한다.
6. effect 안에 비즈니스 로직을 길게 작성하지 않는다.
7. dependency array를 속이기 위해 eslint를 끄지 않는다.

## 1. Derived state를 useEffect로 만들지 않는다

기존 상태에서 계산 가능한 값은 state가 아니라 계산 결과다.

### Do

```tsx
const OrderSummary = ({ orders }: { orders: Order[] }) => {
  const completedCount = orders.filter((order) => order.status === "COMPLETED").length;
  const pendingCount = orders.filter((order) => order.status === "PENDING").length;

  return (
    <Summary
      completedCount={completedCount}
      pendingCount={pendingCount}
    />
  );
};
```

### Do Not

```tsx
const OrderSummary = ({ orders }: { orders: Order[] }) => {
  const [completedCount, setCompletedCount] = useState(0);

  useEffect(() => {
    setCompletedCount(
      orders.filter((order) => order.status === "COMPLETED").length,
    );
  }, [orders]);

  return <Summary completedCount={completedCount} />;
};
```

이 방식은 원본 상태와 파생 상태를 동시에 관리하게 만든다. 상태가 하나 더 생기고, 동기화 실패 가능성도 생긴다.

## 2. props를 state로 복사하지 않는다

props를 state로 복사하는 effect는 대부분 원본을 두 개 만드는 코드다.

### Do

```tsx
const OrderDetail = ({ order }: { order: Order }) => {
  return (
    <DetailPanel
      orderNo={order.orderNo}
      status={order.status.label}
    />
  );
};
```

### Do Not

```tsx
const OrderDetail = ({ order }: { order: Order }) => {
  const [currentOrder, setCurrentOrder] = useState(order);

  useEffect(() => {
    setCurrentOrder(order);
  }, [order]);

  return <DetailPanel orderNo={currentOrder.orderNo} />;
};
```

정말 로컬에서 수정 중인 draft가 필요하다면 `draftOrder`처럼 역할을 드러내고 초기화 타이밍을 명확히 한다.

## 3. 사용자 이벤트는 useEffect가 아니라 event handler에서 처리한다

사용자 클릭, submit, scan 같은 명령은 event handler나 mutation action에서 처리한다.

### Do

```tsx
const OrderSearchForm = ({ onSearch }: { onSearch: (values: SearchValues) => void }) => {
  const form = useForm<SearchValues>();

  return (
    <form onSubmit={form.handleSubmit(onSearch)}>
      <TextField {...form.register("keyword")} />
      <Button type="submit">검색</Button>
    </form>
  );
};
```

### Do Not

```tsx
const OrderSearchForm = ({ keyword }: { keyword: string }) => {
  useEffect(() => {
    searchOrders({ keyword });
  }, [keyword]);

  return <TextField value={keyword} />;
};
```

검색 버튼을 누른다는 사용자 명령을 state 변경 감지 effect로 우회하지 않는다.

## 4. 서버 상태 조회는 React Query를 우선 사용한다

API 요청은 loading, error, cache, retry, refetch, invalidate가 함께 따라온다. 이 책임은 `useEffect + useState`가 아니라 React Query가 맡는다.

### Do

```tsx
const useOrderList = (params: OrderListParams) => {
  return useQuery({
    queryKey: orderKeys.list(params),
    queryFn: () => fetchOrderList(params),
    keepPreviousData: true,
  });
};
```

### Do Not

```tsx
const OrderList = ({ params }: { params: OrderListParams }) => {
  const [orders, setOrders] = useState<Order[]>([]);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    setLoading(true);
    fetchOrderList(params)
      .then((response) => setOrders(response.data.orders))
      .finally(() => setLoading(false));
  }, [params]);

  return <OrderTable orders={orders} loading={loading} />;
};
```

이 방식은 서버 상태 생명주기를 컴포넌트가 직접 관리하게 만든다.

## 5. 외부 시스템 구독에는 cleanup이 필요하다

소켓, event listener, timer처럼 외부 시스템과 연결하는 effect는 cleanup을 반드시 작성한다.

### Do

```tsx
const useBarcodeScanner = (onScan: (barcode: string) => void) => {
  useEffect(() => {
    const unsubscribe = barcodeScanner.subscribe((barcode) => {
      onScan(barcode);
    });

    return () => {
      unsubscribe();
    };
  }, [onScan]);
};
```

```tsx
const useWindowResize = (onResize: () => void) => {
  useEffect(() => {
    window.addEventListener("resize", onResize);

    return () => {
      window.removeEventListener("resize", onResize);
    };
  }, [onResize]);
};
```

### Do Not

```tsx
useEffect(() => {
  window.addEventListener("resize", onResize);
}, []);
```

cleanup이 없으면 화면을 벗어난 뒤에도 listener가 남을 수 있다.

## 6. DOM 직접 조작은 좁게 격리한다

focus, scroll, measurement처럼 DOM이 필요한 작업은 effect를 사용할 수 있다. 다만 UI 정책과 비즈니스 로직이 섞이지 않게 좁게 유지한다.

### Do

```tsx
const BarcodeInput = ({ value, onChange }: Props) => {
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    inputRef.current?.focus();
  }, []);

  return (
    <input
      ref={inputRef}
      value={value}
      onChange={(event) => onChange(event.target.value)}
    />
  );
};
```

### Do Not

```tsx
useEffect(() => {
  inputRef.current?.focus();

  if (order.status === "READY" && scannedBarcode === order.barcode) {
    completePacking(order.id);
  }
}, [order, scannedBarcode]);
```

focus 같은 DOM 부수효과와 포장 완료 같은 업무 명령을 하나의 effect에 섞지 않는다.

## 7. storage 동기화는 전용 hook으로 격리한다

`localStorage`, `sessionStorage`는 React 바깥의 browser storage다. 직접 여러 컴포넌트에서 다루지 말고 전용 hook으로 격리한다.

### Do

```tsx
const [accountType, setAccountType] = useStorageState("accountType", "DIRECT");
```

### Do Not

```tsx
useEffect(() => {
  localStorage.setItem("accountType", accountType);
}, [accountType]);
```

storage 접근이 여러 컴포넌트에 흩어지면 초기값, serialize, browser 환경 예외 처리가 중복된다.

## 8. dependency array를 거짓말하지 않는다

dependency array는 effect가 어떤 값과 동기화되는지 설명한다.

의존성을 빼서 effect 실행을 막는 것은 버그를 숨기는 것이다.

### Do

```tsx
useEffect(() => {
  socket.subscribe(roomId);

  return () => {
    socket.unsubscribe(roomId);
  };
}, [roomId]);
```

### Do Not

```tsx
useEffect(() => {
  socket.subscribe(roomId);

  return () => {
    socket.unsubscribe(roomId);
  };
  // eslint-disable-next-line react-hooks/exhaustive-deps
}, []);
```

의존성을 넣으면 문제가 생긴다면 effect 안의 책임이 너무 많거나, 안정적인 callback/hook 경계가 필요한 신호다.

## 9. effect가 필요해 보이면 먼저 질문한다

```md
1. 이 값은 렌더링 중 계산할 수 있는가?
   -> 그렇다면 effect가 아니다.

2. 사용자의 명령에 반응하는 로직인가?
   -> event handler나 mutation action에서 처리한다.

3. 서버 상태 조회인가?
   -> React Query를 사용한다.

4. React 바깥의 외부 시스템과 동기화하는가?
   -> effect를 사용할 수 있다.

5. 구독, timer, event listener인가?
   -> cleanup을 반드시 작성한다.

6. effect 안에 비즈니스 정책이 길게 들어가는가?
   -> domain helper나 use-case hook으로 분리한다.
```

## 금지 패턴 요약

### 계산 가능한 값을 effect로 state화

```tsx
useEffect(() => {
  setTotalCount(rows.length);
}, [rows]);
```

### props를 state로 복사

```tsx
useEffect(() => {
  setValue(props.value);
}, [props.value]);
```

### API 조회를 useEffect로 직접 관리

```tsx
useEffect(() => {
  fetchList().then(setRows);
}, []);
```

### 사용자 명령을 state 변경 감지로 우회

```tsx
useEffect(() => {
  if (submitted) {
    submitForm();
  }
}, [submitted]);
```

### cleanup 없는 구독

```tsx
useEffect(() => {
  socket.on("message", onMessage);
}, []);
```

## 결론

`useEffect`는 상태 계산 도구가 아니라 외부 시스템과 React를 동기화하는 도구다.

렌더링 중 계산 가능한 값은 계산하고, 사용자 명령은 event handler에서 처리하고, 서버 상태는 React Query로 관리한다.

`useEffect`는 네트워크, 소켓, DOM, timer, storage, 외부 SDK처럼 React 바깥의 세계와 연결해야 할 때 좁고 명확하게 사용한다.
