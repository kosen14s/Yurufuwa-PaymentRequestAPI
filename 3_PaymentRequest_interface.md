# 3. PaymentRequest interface

```
[Constructor(sequence<PaymentMethodData> methodData, PaymentDetailsInit details, optional PaymentOptions options),
 SecureContext]
interface PaymentRequest : EventTarget {
    Promise<PaymentResponse> show();
    Promise<void>            abort();
    Promise<boolean>         canMakePayment();

    readonly attribute DOMString            id;
    readonly attribute PaymentAddress?      shippingAddress;
    readonly attribute DOMString?           shippingOption;
    readonly attribute PaymentShippingType? shippingType;

             attribute EventHandler         onshippingaddresschange;

             attribute EventHandler         onshippingoptionchange;
};
```

ウェブページは決済請求のために PaymentRequest を作ります。
これは一般的に決済処理を始めるユーザーに関連付けられます（例えばゲームで「パワーアップ」を選択したり、立体駐車場を利用して精算機に向かったり、「買う」「支払う」といったボタンを押したりする）。
この PaymentRequest はユーザーが決済請求の承認や拒否前に入力を提供しているとき、ウェブページがユーザーエージェントと情報交換することを可能にします。

`shippingAddress` `shippingOption` `shippingType` 属性は requestShipping フラグが立っているときに設定されます。

下の例では、どのようにして PaymentRequest を組立て、ユーザーインタラクションを始めるかを示しています。

```
EXAMPLE 1: Constructing a PaymentRequest object
function validateResponse(response){
  // check that the response is ok... throw if bad, for example.
}

async function doPaymentRequest() {
  const payment = new PaymentRequest(methodData, details, options);
  payment.addEventListener("shippingaddresschange", event => {
      // Process shipping address change
  });
  let paymentResponse;
  try {
    paymentResponse = await payment.show();
    // paymentResponse.methodName contains the selected payment method.
    // paymentResponse.details contains a payment method specific
    // response.
    validateResponse(paymentResponse);
    paymentResponse.complete("success");
  } catch (err) {
    console.error("Uh oh, bad payment response!", err.message);
    paymentResponse.complete("fail");
  }
}
doPaymentRequest();
```
