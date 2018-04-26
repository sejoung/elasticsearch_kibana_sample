
```

==================================================================================================================================================================
■ Saved Search
==================================================================================================================================================================

[1] 성공한 주문 내역만 검색하는 Saved Search 생성

- Name : SEARCH-Successful_Orders
- Search Query : is_successful_order:true


[2] 성공한 주문 내역을 보여주는 Metric 생성

- From a Saved Search : SEARCH-Successful_Orders

- Name : METRIC-Successful_Order_Count
- Metrics
  - Metric
    - Aggregation : Count
    - Custom Label : 성공한 주문 건수


[3] Dashboard 생성

- Name : Dashboard_1
- Visualizations 추가
  ㄴ METRIC-Successful_Order_Count
- Saved Search 추가
  ㄴ SEARCH-Successful_Orders


==================================================================================================================================================================
■ Scripted Field
==================================================================================================================================================================

[4] 아이템들의 평균 가격을 나타내는 Scripted Field 생성

- Name : AVG_SUM_PER_ORDER
- Language : expression
- Type : number
- Format : Number
- Numeral.js format pattern : 0,0
- Script : doc['payment_sum'].value / doc['items_count'].value


[5] 아이템 평균 최고가를 보여주는 Vertical Bar 생성

- From a Saved Search : SEARCH-Successful_Orders

- Name : VERTICAL_BAR-Max_Avg_Items_Price
- Metrics
  - Y-Axis
    - Aggregation : Max
    - Custom Label : 아이템 평균 가격
- Buckets
  - X-Axis
  - Aggregation : Date Histogram
  - Field : order_datetime
  - Interval : Daily
  - Custom Label : 주문 시간


[6] user_firstname과 user_lastname을 결합하는 Scripted Field 생성

- Name : USER_FULL_NAME
- Language : painless
- Type : string
- Script : doc['user_firstname.keyword'].value + ' ' + doc['user_lastname.keyword'].value


[7] 월별 구매 아이템 평균 금액 순으로 유저 이름 표시하는 Vertical Bar 생성

- From a Saved Search : SEARCH-Successful_Orders

- Name : VERTICAL_BAR-Max_Avg_Items_Price_Monthly
- Metrics
  - Y-Axis
    - Aggregation : MAX
    - Custom Label : 아이템 평균 가격
- Buckets
  - X-Axis
    - Aggregation : Date Histogram
    - Field : order_datetime
    - Interval : Monthly
    - Custom Label : 주문 시간
  - Split Series
    - Sub Aggregation : Terms
    - Field : USER_FULL_NAME
    - Order By : metric:아이템 평균 가격
    - Order : Descending
    - Size : 5
    - Custom Lable : 유저 이름


[8] 쿠폰 사용 비율 Scripted Field 생성

- Name : RATE_COUPON_PER_ORDER
- Language : painless
- Type : number
- Format : Number
- Numeral.js format pattern : 0
- Script :
  long items_count = doc['items_count'].value;
  float rate = 0;
  if (items_count > 0) {
    float coupon_count = 0;
    for (int i = 0; i < items_count; i++) {
      if (params._source.items[i].is_applied_coupon == true) {
        coupon_count += 1;
      }
    }
    rate = coupon_count / items_count * 100;
  }
  return rate;


[9] 일일 평균 쿠폰 사용 비율 보여주는 Vertical Bar 생성

- From a Saved Search : SEARCH-Successful_Orders

- Name : VERTICAL_BAR-Rate_Coupon_Per_Order
- Metrics
  - Y-Axis
    - Aggregation : Average
    - Field : RATE_COUPON_PER_ORDER
    - Custom Label : 일일 쿠폰 평균 사용 비율
- Buckets
  - X-Axis
  - Aggregation : Date Histogram
  - Field : order_datetime
  - Interval : Daily
  - Custom Label : 주문 일자


[10] Dashboard 생성

- Name : Dashboard_2
- Visualizations 추가
  ㄴ VERTICAL_BAR-Max_Avg_Items_Price
  ㄴ VERTICAL_BAR-Max_Avg_Items_Price_Monthly
  ㄴ VERTICAL_BAR-Rate_Coupon_Per_Order


==================================================================================================================================================================
■ Timelion
==================================================================================================================================================================

[11] 결제 방법별 주문 건수 생성 (with Threshold)

- Name : TIMESERIES-Order_Count_Payment_Method
- Script :
.es(index=s3-order-list*, timefield=order_datetime).label('Total'),
.es(index=s3-order-list*, timefield=order_datetime, q=payment_method:CREDIT_CARD).label('Credit Card').bars(),
.es(index=s3-order-list*, timefield=order_datetime, q=payment_method:CASH).label('Cash').bars(),
.static(40, label='40').lines(fill=3)


[12] 주문 건수 전주 비교 그래프 생성

- Name : TIMESERIES-Order_Count_Compare_1w
- Script :
.es(index=s3-order-list*, timefield=order_datetime, offset=-1w).label('전주 주문건수').title('금주, 전주 주문건수 비교 그래프'),
.es(index=s3-order-list*, timefield=order_datetime).label('주문건수')


[13] 결제 카드 상위 5개 표현하기

- Name : TIMESERIES-Top5_Card_Type
- Script :
.es(index=s3-order-list*, timefield=order_datetime, split=payment_card_type.keyword:5).bars().label(regex='.* payment_card_type.keyword:(.*) > .*', label='$1')


[14] 주문 호조, 저조한 상태 표현하기

- Name : TIMESERIES-Order_Count_High_Low
- Script :
.es(index=s3-order-list*, timefield=order_datetime).label('주문 건수'),
.es(index=s3-order-list*, timefield=order_datetime).if(operator=lte, 25, .es(index=s3-order-list*, timefield=order_datetime), null).label('주문 저조'),
.es(index=s3-order-list*, timefield=order_datetime).if(operator=gte, 35, .es(index=s3-order-list*, timefield=order_datetime), null).label('주문 호조')


[15] 지금까지 생성한 Timeseries를 보여주는 Dashboard 생성

- Name : Dashboard_3
- Visualizations 추가
  ㄴ TIMESERIES-Order_Count_Payment_Method
  ㄴ TIMESERIES-Order_Count_Compare_1w
  ㄴ TIMESERIES-Top5_Card_Type
  ㄴ TIMESERIES-Order_Count_High_Low

 ```