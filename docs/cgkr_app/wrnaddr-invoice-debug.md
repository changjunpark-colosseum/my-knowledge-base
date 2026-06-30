# 송장 출력 `20121 NOT_VALID_ADDRESS` 이슈 확인 요청

## 요약

송장 출력 API 호출 시 `20121 주소가 유효하지 않습니다.` 응답이 발생했습니다.

현재 코드 흐름상 이 에러는 프론트엔드에서 주소를 검증해서 만든 값이 아니라, 백엔드가 `outbound_tracking_dtl.invoice_stats_cd = 'WRNADDR'` 상태를 읽고 `NOT_VALID_ADDRESS`로 변환하면서 발생한 것으로 보입니다.

## 발생 API

- API: `POST /api/basic/v1/pack/invoice`
- 화면 추정: `/outbound/process/invoice`
- 응답 코드: `20121`
- 응답 메시지: `주소가 유효하지 않습니다.`
- 응답 시간: `2026-05-11 00:57:16`
- `tranNo`: `2fabf999-777b-4dfc-a786-6570b3867d02`

## 요청 payload

```json
{
  "outboundId": 350859,
  "outboundInvoiceDtlId": 344022,
  "outboundTrackingDtlId": 352408
}
```

## 코드상 직접 원인

`/api/basic/v1/pack/invoice`는 `outboundTrackingDtlId`로 송장 정보를 조회한 뒤, 기존 송장 상태가 오류 상태면 바로 예외를 던집니다.

파일:

```text
wms-legacy/src/main/java/global/colosseum/colo/outbound/pack/controller/PackController.java
```

관련 로직:

```java
PrintInvoiceOutDTO printInfo = packCommService.findInvoiceInfoWithAttachCd(outboundTrackingDtlId, languageTypeCd);

if (StringUtils.hasText(printInfo.getInvoiceStatsCd())) {
    switch (EnumUtil.toEnum(InvoiceStatsCd.class, printInfo.getInvoiceStatsCd())) {
        case WRNADDR -> throw new BaseException(BaseMsg.NOT_VALID_ADDRESS);
        case NOITMWT -> throw new BaseException(BaseMsg.EMPTY_WEIGHT);
        case FAILREQ,COLOERR -> throw new BaseException(BaseMsg.FAIL_INVOICE);
    }
}
```

따라서 이번 응답은 `/pack/invoice`에서 주소 검증을 새로 수행해서 실패한 것이 아니라, 이미 저장된 `outbound_tracking_dtl.invoice_stats_cd`가 `WRNADDR`였기 때문에 발생한 것으로 판단됩니다.

## 조회 쿼리 근거

`findInvoiceInfoWithAttachCd`는 `outboundTrackingDtlId` 기준으로 `outbound_tracking_dtl.invoice_stats_cd`를 조회합니다.

파일:

```text
src/main/resources/mapper/PackMapper.xml
```

관련 컬럼:

```sql
otd.invoice_stats_cd AS invoice_stats_cd
```

관련 조건:

```sql
WHERE otd.outbound_tracking_dtl_id = #{outboundTrackingDtlId}
```

## `WRNADDR`가 처음 만들어지는 시점

`WRNADDR`는 송장 출력 API가 아니라, 그 이전 송장 발번/택배사 연동 단계에서 `outbound_tracking_dtl`에 저장됩니다.

대표 진입 API:

- `POST /api/basic/v1/outbound/generate-invoice`
- `POST /api/basic/v1/outbound/generate-and-reprint-invoice-by-work`
- `POST /api/basic/v1/outbound/add-invoice`

파일:

```text
wms-legacy/src/main/java/global/colosseum/colo/outbound/common/controller/OutboundCenterController.java
```

택배사별 처리:

- CJ: `invoiceService.generateInvoiceCj(...)`
- 한진: `invoiceService.generateInvoiceHj(...)`

## `WRNADDR` 세팅 조건

현재 코드에서 `InvoiceStatsCd.WRNADDR`를 직접 세팅하는 곳은 `InvoiceService` 안의 3곳입니다.

파일:

```text
wms-legacy/src/main/java/global/colosseum/colo/outbound/invoice/service/InvoiceService.java
```

### CJ 신규 송장 발번

조건:

```java
if (isNotBlank(hist.getInvcNo())) {
    // 성공: CREINVC
} else {
    trackingDtlDTO.setInvoiceStatsCd(InvoiceStatsCd.WRNADDR.toCd());
}
```

의미:

- CJ 주소정제/송장번호 채번/택배 접수 이후에도 `invcNo`가 없으면 `WRNADDR`로 저장됩니다.
- 상세 실패 사유는 `hist.getItfErrCd()`, `hist.getItfErrMsg()`가 있으면 `system_stats_cd`, `fail_message_description`에 저장됩니다.
- 없으면 CJ unknown status로 저장됩니다.

### 한진 신규 송장 발번

조건:

```java
List<HanjinParcelInvoicePrintHistDTO> failList = printHists.stream()
        .filter(i -> !"OK".equals(i.getAddressCheckResultCd()) || !"OK".equals(i.getParcelOrderResultCd()))
        .toList();
```

의미:

- `addressCheckResultCd != "OK"`이면 주소검증 실패입니다.
- `parcelOrderResultCd != "OK"`이면 택배주문 접수 실패입니다.
- 둘 중 하나라도 실패하면 `WRNADDR`로 저장됩니다.

## 백엔드 확인 요청

아래 tracking row가 언제, 어떤 택배사 연동 결과로 `WRNADDR`가 되었는지 확인 부탁드립니다.

```sql
SELECT
  outbound_tracking_dtl_id,
  invoice_stats_cd,
  system_stats_cd,
  fail_message_description,
  tracking_no,
  carrier_type_cd,
  cre_dt,
  upd_dt,
  cre_id,
  upd_id
FROM outbound_tracking_dtl
WHERE outbound_tracking_dtl_id = 352408;
```

확인 포인트:

- `invoice_stats_cd`가 실제로 `WRNADDR`인지
- `WRNADDR`로 변경된 시점이 언제인지
- `system_stats_cd`, `fail_message_description`에 저장된 실제 실패 사유가 무엇인지
- `carrier_type_cd`가 `CJ`, `HJ`, 또는 `NULL` 중 무엇인지
- `tracking_no`가 비어 있는지

## CJ 이력 확인

CJ 발번 시도 건이면 아래 이력을 확인 부탁드립니다.

```sql
SELECT *
FROM cj_parcel_invoice_print_hist
WHERE outbound_tracking_dtl_id = 352408
  AND delete_yn = 'N'
ORDER BY cre_dt DESC;
```

확인 포인트:

- `invc_no` 또는 송장번호 관련 값이 비어 있는지
- `itf_err_cd`, `itf_err_msg`에 CJ 실패 사유가 남아 있는지
- `cre_dt` 기준으로 실제 CJ 연동 시점이 언제인지

## 한진 이력 확인

한진 발번 시도 건이면 아래 이력을 확인 부탁드립니다.

```sql
SELECT *
FROM hanjin_parcel_invoice_print_hist
WHERE outbound_tracking_dtl_id = 352408
  AND delete_yn = 'N'
ORDER BY cre_dt DESC;
```

확인 포인트:

- `address_check_result_cd`, `address_check_result_msg`
- `parcel_order_result_cd`, `parcel_order_result_msg`
- 주소검증 실패인지 택배주문 접수 실패인지

## 판단

현재까지 확인한 바로는 프론트엔드에서 주소 유효성 검증을 실패시킨 이슈라기보다는, 백엔드에 이미 `WRNADDR` 상태로 저장된 송장 tracking row를 출력하려고 해서 발생한 이슈입니다.

다만 프론트엔드가 잘못된 `outboundTrackingDtlId`를 보냈다면 프론트엔드 이슈가 섞일 수 있으므로, `outboundTrackingDtlId = 352408`이 실제 사용자가 선택한 송장 row와 일치하는지도 함께 확인하면 좋습니다.

## 백엔드에 묻고 싶은 결론 질문

1. `outboundTrackingDtlId = 352408`가 언제 `WRNADDR`로 변경되었나요?
2. 해당 건은 CJ 발번 실패인가요, 한진 발번 실패인가요?
3. `system_stats_cd`, `fail_message_description` 또는 택배사 이력 테이블에 남은 실제 실패 사유는 무엇인가요?
4. 해결은 주소/우편번호 수정 후 재발번이면 되나요?
5. 아니면 기존 `outbound_tracking_dtl` 초기화 또는 송장 row 재생성이 필요한 케이스인가요?

## 공유 시 주의

운영 curl 전체를 공유할 때는 쿠키, 세션, Sentry header 등 민감값을 제거해야 합니다.

특히 아래 값은 공유하지 않습니다.

- `CSESSIONID`
- `_ga`, `_clck` 등 브라우저 쿠키
- `sentry-trace`, `baggage`
- 사용자 세션 전체 JSON
