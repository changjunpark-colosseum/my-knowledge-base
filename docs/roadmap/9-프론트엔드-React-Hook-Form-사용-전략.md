# 9. 프론트엔드 React Hook Form 사용 전략

## 목적

폼 상태를 컴포넌트 local state에 흩뿌리지 않고 React Hook Form으로 일관되게 관리하기 위한 기준을 세운다.

폼은 단순 입력값 모음이 아니라 default value, validation, submit, reset, dirty state, field error, payload 변환이 함께 움직이는 단위다.

## 핵심 문장

필드가 여러 개이고 submit/reset/validation이 필요한 폼은 React Hook Form을 기본값으로 사용한다.

단순 input 하나의 임시 상태는 `useState`로 충분할 수 있다. 하지만 검색 폼, 등록 폼, 수정 폼처럼 입력값 묶음이 하나의 유스케이스를 구성한다면 React Hook Form으로 관리한다.

## 적용 기준

| 상황                                | 권장                         |
| ----------------------------------- | ---------------------------- |
| input 하나의 단순 UI 상태           | `useState` 가능              |
| 검색 조건 여러 개                   | React Hook Form              |
| 등록/수정 폼                        | React Hook Form              |
| validation이 필요한 폼              | React Hook Form              |
| submit payload 변환이 필요한 폼     | React Hook Form + serializer |
| reset/defaultValues가 중요한 폼     | React Hook Form              |
| 서버 응답을 form 초기값으로 넣는 폼 | React Hook Form + `reset`    |

## 기본 원칙

1. 필드별 `useState`를 늘리지 않는다.
2. Form 컴포넌트는 입력과 submit 이벤트에 집중한다.
3. 검색 실행, API 호출, URL 변경은 Form 바깥에서 처리한다.
4. 서버 DTO를 form value로 직접 쓰지 않는다.
5. form value와 request payload는 serializer로 분리한다.
6. 서버 데이터로 form을 초기화할 때는 `defaultValues`와 `reset` 타이밍을 명확히 한다.
7. validation은 UI 표현과 payload 생성 사이의 계약으로 관리한다.

## 1. 필드가 여러 개라면 useState를 늘리지 않는다

### AS-IS

```tsx
const OrderSearchForm = ({ onSearch }: { onSearch: (params: OrderSearchParams) => void }) => {
  const [keyword, setKeyword] = useState("");
  const [status, setStatus] = useState("");
  const [dateFrom, setDateFrom] = useState("");
  const [dateTo, setDateTo] = useState("");

  const handleSubmit = () => {
    onSearch({
      keyword,
      status,
      dateFrom,
      dateTo,
    });
  };

  const handleReset = () => {
    setKeyword("");
    setStatus("");
    setDateFrom("");
    setDateTo("");
  };

  return (
    <>
      <TextField value={keyword} onChange={(event) => setKeyword(event.target.value)} />
      <Select value={status} onChange={(event) => setStatus(String(event.target.value))} />
      <DatePicker value={dateFrom} onChange={setDateFrom} />
      <DatePicker value={dateTo} onChange={setDateTo} />

      <Button onClick={handleSubmit}>검색</Button>
      <Button onClick={handleReset}>초기화</Button>
    </>
  );
};
```

문제:

- 필드가 늘어날수록 state와 setter가 계속 늘어난다.
- submit payload 생성이 컴포넌트에 섞인다.
- reset 로직이 필드 수만큼 늘어난다.
- validation을 추가하기 어렵다.

### TO-BE

```tsx
type OrderSearchFormValues = {
  keyword: string;
  status: string;
  dateFrom: string;
  dateTo: string;
};

const orderSearchDefaultValues: OrderSearchFormValues = {
  keyword: "",
  status: "",
  dateFrom: "",
  dateTo: "",
};

const OrderSearchForm = ({
  defaultValues = orderSearchDefaultValues,
  onSubmit,
}: {
  defaultValues?: OrderSearchFormValues;
  onSubmit: (values: OrderSearchFormValues) => void;
}) => {
  const form = useForm<OrderSearchFormValues>({
    defaultValues,
  });

  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      <TextField {...form.register("keyword")} />
      <Controller
        control={form.control}
        name="status"
        render={({ field }) => <Select {...field} />}
      />
      <Controller
        control={form.control}
        name="dateFrom"
        render={({ field }) => <DatePicker value={field.value} onChange={field.onChange} />}
      />
      <Controller
        control={form.control}
        name="dateTo"
        render={({ field }) => <DatePicker value={field.value} onChange={field.onChange} />}
      />

      <Button type="submit">검색</Button>
      <Button type="button" onClick={() => form.reset(orderSearchDefaultValues)}>
        초기화
      </Button>
    </form>
  );
};
```

Form 컴포넌트는 입력값 수집과 submit 이벤트 전달에 집중한다.

## 2. 검색 실행과 URL 변경은 Form 바깥에서 처리한다

Form은 입력값을 수집하고 submit 이벤트를 올린다. 검색 조건을 URL에 반영하거나 query를 실행하는 것은 page/use-case hook의 책임이다.

### AS-IS

```tsx
const OrderSearchForm = () => {
  const router = useRouter();
  const queryClient = useQueryClient();
  const form = useForm<OrderSearchFormValues>();

  const onSubmit = async (values: OrderSearchFormValues) => {
    router.push({
      pathname: router.pathname,
      query: values,
    });

    await queryClient.invalidateQueries(["orders"]);
  };

  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      <TextField {...form.register("keyword")} />
      <Button type="submit">검색</Button>
    </form>
  );
};
```

문제:

- Form 컴포넌트가 router와 queryClient를 안다.
- 입력 UI와 검색 실행 정책이 결합된다.
- 같은 form을 다른 화면에서 재사용하기 어렵다.

### TO-BE

```tsx
const OrderPage = () => {
  const { params, setSearchParams } = useOrderSearchParams();
  const { data, isLoading } = useOrderList(params);

  return (
    <>
      <OrderSearchForm
        defaultValues={params}
        onSubmit={(values) => {
          setSearchParams({
            ...values,
            page: 1,
          });
        }}
      />
      <OrderTable rows={data?.orders ?? []} loading={isLoading} />
    </>
  );
};
```

```tsx
const OrderSearchForm = ({
  defaultValues,
  onSubmit,
}: {
  defaultValues: OrderSearchFormValues;
  onSubmit: (values: OrderSearchFormValues) => void;
}) => {
  const form = useForm<OrderSearchFormValues>({
    defaultValues,
  });

  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      <TextField {...form.register("keyword")} />
      <Button type="submit">검색</Button>
    </form>
  );
};
```

검색 실행과 URL 변경은 page/use-case layer가 담당한다.

## 3. 서버 DTO를 form value로 직접 쓰지 않는다

서버 응답 DTO와 form value는 목적이 다르다.

서버 DTO는 외부 데이터 계약이고, form value는 사용자가 입력/수정하는 화면 모델이다.

### AS-IS

```tsx
const ShippingAddressForm = ({ orderResponse }: { orderResponse: OrderResponse }) => {
  const form = useForm<OrderResponse>({
    defaultValues: orderResponse,
  });

  return (
    <form>
      <TextField {...form.register("recv_nm")} />
      <TextField {...form.register("recv_addr_01")} />
      <TextField {...form.register("recv_zip_no")} />
    </form>
  );
};
```

문제:

- form field name이 서버 필드명에 종속된다.
- 서버 DTO 변경이 form UI에 직접 영향을 준다.
- payload 생성과 화면 입력 모델이 분리되지 않는다.

### TO-BE

```ts
type ShippingAddressFormValues = {
  receiverName: string;
  address1: string;
  zipCode: string;
};

const mapOrderResponseToShippingAddressFormValues = (
  response: OrderResponse,
): ShippingAddressFormValues => {
  return {
    receiverName: response.recv_nm ?? "",
    address1: response.recv_addr_01 ?? "",
    zipCode: response.recv_zip_no ?? "",
  };
};
```

```tsx
const ShippingAddressForm = ({
  defaultValues,
  onSubmit,
}: {
  defaultValues: ShippingAddressFormValues;
  onSubmit: (values: ShippingAddressFormValues) => void;
}) => {
  const form = useForm<ShippingAddressFormValues>({
    defaultValues,
  });

  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      <TextField {...form.register("receiverName")} />
      <TextField {...form.register("address1")} />
      <TextField {...form.register("zipCode")} />
      <Button type="submit">저장</Button>
    </form>
  );
};
```

form은 화면 입력 모델을 사용하고, 서버 DTO 변환은 mapper가 담당한다.

## 4. form value와 request payload를 분리한다

form value는 사용자가 입력하기 편한 형태이고, request payload는 서버가 요구하는 형태다.

### AS-IS

```tsx
const onSubmit = async (values: ShippingAddressFormValues) => {
  await updateShippingAddress({
    recv_nm: values.receiverName,
    recv_addr_01: values.address1,
    recv_zip_no: values.zipCode,
  });
};
```

문제:

- submit handler에 서버 필드명이 섞인다.
- payload 변환 규칙이 컴포넌트에 들어온다.

### TO-BE

```ts
const createShippingAddressRequest = (
  values: ShippingAddressFormValues,
): UpdateShippingAddressRequest => {
  return {
    recv_nm: values.receiverName.trim(),
    recv_addr_01: values.address1.trim(),
    recv_zip_no: values.zipCode,
  };
};
```

```tsx
const useUpdateShippingAddress = () => {
  return useMutation({
    mutationFn: (values: ShippingAddressFormValues) => {
      return updateShippingAddress(createShippingAddressRequest(values));
    },
  });
};
```

payload 생성은 serializer/helper로 분리한다.

## 5. 서버 데이터로 form을 초기화할 때 reset 타이밍을 명확히 한다

`defaultValues`는 최초 렌더링 시점에 적용된다. 서버 데이터가 나중에 도착하면 `reset`으로 명시적으로 반영한다.

### AS-IS

```tsx
const OrderEditForm = ({ order }: { order?: Order }) => {
  const form = useForm<OrderEditFormValues>({
    defaultValues: {
      receiverName: order?.receiverName ?? "",
    },
  });

  return <TextField {...form.register("receiverName")} />;
};
```

서버 데이터가 나중에 도착해도 `defaultValues`가 자동으로 다시 적용되지 않는다.

### TO-BE

```tsx
const OrderEditForm = ({ order }: { order?: Order }) => {
  const form = useForm<OrderEditFormValues>({
    defaultValues: emptyOrderEditFormValues,
  });

  useEffect(() => {
    if (!order) {
      return;
    }

    form.reset(mapOrderToOrderEditFormValues(order));
  }, [form, order]);

  return <TextField {...form.register("receiverName")} />;
};
```

서버 데이터가 바뀌는 시점에 form 초기화가 필요한 경우에만 `reset`을 사용한다.

주의: 이 `useEffect`는 외부 데이터인 서버 응답과 form 내부 상태를 동기화하는 목적이므로 허용된다. 단, 단순 derived state를 만들기 위한 effect와는 구분한다.

## 6. validation은 form schema로 관리한다

입력값 검증은 컴포넌트 곳곳의 조건문이 아니라 form validation 규칙으로 관리한다.

### AS-IS

```tsx
const onSubmit = (values: OrderFormValues) => {
  if (!values.receiverName) {
    alert({ title: "ERROR", content: "수령인명을 입력해주세요." });
    return;
  }

  if (!values.zipCode) {
    alert({ title: "ERROR", content: "우편번호를 입력해주세요." });
    return;
  }

  submitOrder(values);
};
```

### TO-BE

```ts
const orderFormSchema = z.object({
  receiverName: z.string().min(1, "수령인명을 입력해주세요."),
  zipCode: z.string().min(1, "우편번호를 입력해주세요."),
});

type OrderFormValues = z.infer<typeof orderFormSchema>;
```

```tsx
const form = useForm<OrderFormValues>({
  defaultValues,
  resolver: zodResolver(orderFormSchema),
});
```

```tsx
<TextField
  {...form.register("receiverName")}
  error={Boolean(form.formState.errors.receiverName)}
  helperText={form.formState.errors.receiverName?.message}
/>
```

검증 실패는 field error로 보여주고, submit handler는 성공한 값만 처리한다.

## 7. Controller는 외부 controlled component에만 사용한다

기본 input처럼 `register`로 연결 가능한 필드는 `register`를 우선 사용한다.

MUI Select, DatePicker, custom input처럼 controlled component는 `Controller`를 사용한다.

### Do

```tsx
<TextField {...form.register("keyword")} />
```

```tsx
<Controller
  control={form.control}
  name="dateFrom"
  render={({ field }) => <DatePicker value={field.value} onChange={field.onChange} />}
/>
```

### Do Not

```tsx
<Controller
  control={form.control}
  name="keyword"
  render={({ field }) => <TextField {...field} />}
/>
```

모든 필드를 습관적으로 `Controller`로 감싸지 않는다.

## 8. FormProvider는 깊은 form tree에서만 사용한다

폼 필드가 여러 하위 컴포넌트로 나뉘고 props drilling이 커질 때 `FormProvider`를 사용한다.

### Do

```tsx
const OrderCreateForm = () => {
  const methods = useForm<OrderCreateFormValues>({
    defaultValues,
  });

  return (
    <FormProvider {...methods}>
      <OrderReceiverFields />
      <OrderItemFields />
      <OrderMemoFields />
    </FormProvider>
  );
};
```

```tsx
const OrderReceiverFields = () => {
  const { register } = useFormContext<OrderCreateFormValues>();

  return <TextField {...register("receiverName")} />;
};
```

### Do Not

```tsx
const KeywordInput = () => {
  const { register } = useFormContext<OrderSearchFormValues>();

  return <TextField {...register("keyword")} />;
};
```

작은 검색 폼에서 field 하나를 분리하기 위해 무조건 FormProvider를 쓰지 않는다.

## 9. watch 남용을 피한다

`watch`는 form 값 변화를 구독한다. 많이 쓰면 렌더링과 의존 관계를 추적하기 어려워질 수 있다.

### Do

```tsx
const shippingType = form.watch("shippingType");

return (
  <>
    <ShippingTypeSelect control={form.control} />
    {shippingType === "PARCEL" && <ParcelFields control={form.control} />}
  </>
);
```

### Do Not

```tsx
const values = form.watch();

useEffect(() => {
  setQueryData(values);
}, [values]);
```

전체 form 값을 watch해서 외부 상태와 계속 동기화하지 않는다. 검색은 submit 시점에 반영한다.

## 금지 패턴 요약

### 필드마다 useState 생성

```tsx
const [keyword, setKeyword] = useState("");
const [status, setStatus] = useState("");
const [dateFrom, setDateFrom] = useState("");
```

### Form 안에서 router/queryClient 직접 사용

```tsx
router.push({ query: values });
queryClient.invalidateQueries(["orders"]);
```

### 서버 DTO를 form type으로 사용

```tsx
useForm<OrderResponse>({ defaultValues: response });
```

### submit handler에서 payload 직접 조립

```tsx
await updateAddress({ recv_nm: values.receiverName });
```

### 모든 필드를 Controller로 감싸기

```tsx
<Controller render={({ field }) => <TextField {...field} />} />
```

### 전체 form watch로 외부 상태 동기화

```tsx
const values = form.watch();
useEffect(() => setParams(values), [values]);
```

## 판단 기준

```md
1. 필드가 1개뿐인 단순 UI 상태인가?
   -> useState 가능.

2. 필드가 여러 개이고 submit/reset/validation이 필요한가?
   -> React Hook Form을 사용한다.

3. 검색 조건을 URL에 반영해야 하는가?
   -> form submit 결과를 page/use-case layer에서 URL state로 반영한다.

4. 서버 응답으로 초기값을 채워야 하는가?
   -> DTO -> form values mapper를 만들고 reset 타이밍을 명확히 한다.

5. 서버 요청 payload와 form 값이 다른가?
   -> serializer를 만든다.

6. 하위 필드 컴포넌트가 많아 props drilling이 심한가?
   -> FormProvider를 고려한다.
```

## 결론

React Hook Form은 input 값을 편하게 읽기 위한 라이브러리가 아니라 폼 유스케이스의 상태와 계약을 관리하는 도구다.

검색/등록/수정 폼처럼 필드 묶음이 하나의 업무 흐름을 구성한다면 React Hook Form을 사용한다. Form 컴포넌트는 입력과 submit 이벤트에 집중하고, URL 변경, API 호출, payload 변환, 서버 DTO 변환은 바깥 계층으로 분리한다.
