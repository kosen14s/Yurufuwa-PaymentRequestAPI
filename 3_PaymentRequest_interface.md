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
これは一般的に決済処理を始めるユーザーに関連付けられます（例えばゲームで「パワーアップ」を選択したり、立体駐車場で車を精算機につけたり、「買う」「支払う」といったボタンを押したりする）。
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

## 3.1 Constructor

PaymentRequest は、渡された methodData リスト（決済方法固有のデータを含む）、決済の詳細、決済オプションによって構成されます。
PaymentRequest コンストラクタに渡された methodData は、呼び出し元の優先度（プリファレンス）順に並んでいる必要があります。

NOTE:
methodData シーケンスは PaymentMethodData dictionaries をもち、それはウェブサイトが許可する決済方法のための識別子 `payment method identifiers` 及び関連する決済方法固有のデータをもちます。

```
EXAMPLE 2: The 'methodData' argument
const methodData = [{
  supportedMethods: ["basic-card"],
  data: {
    supportedNetworks: ['aFamousBrand', 'aDebitNetwork'],
    supportedTypes: ['debit']
  }
}, {
  supportedMethods: ["bobpay.com"],
  data: {
    merchantIdentifier: "XXXX",
    bobPaySpecificField: true
  }
}];
const request = new PaymentRequest(methodData, details, options);
```

details オブジェクトには例えば注文の詳細など、ユーザーの入力が求められている取引についての情報が含まれています。

```
EXAMPLE 3: The 'details' argument
const details = {
  id: "super-store-order-123-12312",
  displayItems: [
    {
      label: "Sub-total",
      amount: { currency: "USD", value : "55.00" }, // US$55.00
    },
    {
      label: "Sales Tax",
      amount: { currency: "USD", value : "5.00" }, // US$5.00
    }
  ],
  total: {
    label: "Total due",
    amount: { currency: "USD", value : "60.00" }, // US$60.00
  }
}
const request = new PaymentRequest(methodData, details, options);
```

options オブジェクトには、ウェブページが決済請求システムからどのオプションを使いたいかについての情報が含まれています。

```
EXAMPLE 4: The 'options' argument
const options = {
  requestShipping: true
}
const request = new PaymentRequest(methodData, details, options);
```

`PaymentRequest(methodData, details, options)` コンストラクタは次のように振る舞います。
