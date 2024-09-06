## Toss Payments

<img width="1008" alt="image" src="https://github.com/user-attachments/assets/8b9779ae-242a-4209-a5db-dd6a39e9374d">

결제 시스템은 다음의 이유로, 결제 대행사 PG사의 도움을 받아 결제를 처리해야 한다.

- 아마존과 비교하여 PCI DSS (Payment Card Industry Data Security Standard) 같은 엄격한 보안을 달성하는 것이 어렵다.
- PG사가 없으면, 온라인 사업자는 카드사와 직접 계약을 맺어야 하며, 이 과정은 불편함을 주는 요소이다. 그래서 PG사를 이용하는 것을 선호

### 결제 위젯을 이용한 결제

Toss Payments는 아래와 같은 결제 위젯을 제공한다. 이 결제 위젯을 통해 사용자는 카드 정보 등의 결제 정보를 입력할 수 있다.

<img width="600" alt="image" src="https://github.com/user-attachments/assets/0ab6c065-45ba-478a-a093-f81bc23383a0">

## Toss Payments를 이용한 결제 시스템 설계

결제 시스템과 결제 처리 프로세스는 다음과 같다.

<img width="684" alt="image" src="https://github.com/user-attachments/assets/4be3c494-e7dd-40ee-9d3e-9de3e06e5458">


1. Check-out
    1. 장바구니에 물건을 담은 후, ‘구매하기’와 같은 버튼을 클릭하는 과정
2. Display
    1. Payment Service에서 Event 생성한 후 결제 페이지를 표시하는 과정
    2. 중요한 것은 결제를 식별할 수 있는 Unique id (ex. UUID)를 생성하고, 결제 금액을 계산하여 함께 전달하는 것이다. 이를 바탕으로 사용자는 결제 위젯을 통해 결제를 수행한다.
3. Start Payment
    1. 결제 페이지의 결제 위젯에서 “결제하기” 버튼을 클릭하면 PSP 결제 창이 표시된다. (토스페이먼츠, 현대카드에서 등등 “결제하기” 버튼을 확인해봤을 것이다.) 여기에서 카드 정보와 같은 결제 정보를 입력하여 결제를 진행한다.
    2. 입력된 정보는 암호화된 후 PSP로 전달되며 결제가 시작된다.


<img width="313" alt="image" src="https://github.com/user-attachments/assets/0f7ed2a0-08fa-4e95-8baf-1454dceea407">

<img width="291" alt="image" src="https://github.com/user-attachments/assets/8db0249d-ac74-4838-bb0e-6817e8dcb36f">

1. Payment Authentication Result
    1. PSP에서 결제 인증이 완료된 후, 성공 또는 실패에 대한 응답을 알려주는 과정
2. Redirect
ex) `https://store.com/success?paymentKey={PAYMENT_KEY}&orderId={ORDER_ID}&amount=100`
    1. 전달받은 Payment Result에 따라 성공한 경우 성공한 페이지, 실패한 경우 실패 페이지로 리다이렉트되는 과정이다. 리다이렉션될 때 파라미터로 paymentKey, orderId, amount 를 전달받는다.
3. Notify
    1. 구매자가 결제에 성공했다는 사실을 결제 서버에 알리는 과정이다. 이제 서버 측에서 결제 승인을 하면, 결제 거래가 종료되며 돈이 이동하게 된다.
4. Request
    1. 결제 서버가 결제 승인을 위해 PSP에 요청하는 과정이다.
    2. 결제 승인을 요청할 때는 리다이렉션으로 받은 파라미터인 paymentKey, orderId, amount를 사용하여 PSP에서 지원하는 결제 승인 API를 호출하면 된다. 이 과정이 완료되어야 구매자의 계좌에서 돈이 차감된다.
5. Payment Confirm Result
    1. PSP에서 결제 승인 결과를 전달하는 과정이다.
6. Send Response
    1. 사용자에게 결제 승인 결과를 전달하는 과정이다.
7. Send Event
    1. 결제 승인이 완료된 이후 결제 승인 이벤트를 발생시키는 과정
8. Complete Payment Event
    1. 모든 결제에 관련한 후속 작업이 처리되면 결제가 완료된다.

<h2>Payment Service를 위한 데이터 모델</h2>
<p>결제 서비스에 필수적인 도메인 객체인 Payment Event와 Payment Order에 대해 알아보자.</p>
<h3>Payment Event와 Order</h3>
Payment Event는 결제가 필요할 때 만들어지며, 장바구니에서 결제 페이지로 이동하는 CheckOut과정에서 생성된다. <br />
Payment Event의 중요한 데이터들은 다음과 같다.

Name | Type
-- | --
id | BIG INT (PK)
order_id | STRING (UK)
payment_key | STRING
is_payment_done | BOOLEAN


<ul>
<li>id: Payment Event의 id값을 의미</li>
<li>order_id: PSP에서 주문을 유일하게 식별하기 위해 사용</li>
<li>payment_key: PSP에서 결제 승인 처리 후에, 해당 결제를 식별하기 위해 생성된 아이디 값으로, 이 값 또는 order_id를 사용하여 결제 내역을 조회하거나 취소할 수 있다.</li>
<li>is_payment_done: 결제가 성공적으로 완료되었는지 여부를 나타낸다.</li>
</ul>
<p>Payment Order는 실제 결제 대상을 의미하며, 이는 구매자가 구매하는 물품을 지칭한다.</p>

Name | Type
-- | --
id | BIG INT (PK)
payment_event_id | BIG INT (FK)
payment_order_status | ENUM
amount | BIG DECIMAL
ledger_updated | BOOLEAN
wallet_updated | BOOLEAN
buyer_id | BIG INT (FK)
seller_id | BIG INT (FK)

<ul>
<li>id: Payment Order의 id값을 의미한다.</li>
<li>payment_event_id: Payment Event와 관계를 의미한다.</li>
<li>payment_order_status: 결제 승인 상태를 나타낸다. (e.g NOT_STARTED, EXECUTING, FAILURE, SUCCESS, UNKNOWN)</li>
<li>amount: 결제 금액을 나타낸다.</li>
<li>ledger_updated: 결제 승인 이후에 장부에 기록했는지 여부를 나타낸다.</li>
<li>wallet_updated: 결제 승인 이후에 판매자 지갑에 정산 처리를 했는 지 여부를 나타낸다.</li>
<li>buyer_id: 구매자의 id 값을 의미한다</li>
<li>seller_id: 판매자의 id 값을 의미한다.</li>
</ul>
<h2>Double-Entry Ledger System</h2>
<p>Ledger 서비스는 결제 거래 내역을 추적하기 위해 모든 거래 내역을 장부에 기록한다. 이때, 돈을 얻은 쪽과 잃은 쪽 모두를 기록하는 Double-Entry 기법을 사용한다.</p>

Account | Debit | Credit
-- | -- | --
buyer | $1 |  
seller |   | $1


<p>이렇게 하는 이유는 재무적인 오류를 쉽게 발견할 수 있기 때문이다. (장부에 기록된 내역들을 합치면 0이 되어야 한다.)</p>
<p>그리고 장부는 원칙적으로 불변성을 보장하고 있다. 변경이 가능한 순간에는 변경 내역 추적이 어렵다.</p>
