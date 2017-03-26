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

`PaymentRequest(methodData, details, options)` コンストラクタは次のように振る舞うべきです。

1. current settings object の responsible document が allowpaymentrequest 属性に示された機能の使用を許可していないとき、 SecurityError DOMException を投げます。
2. serializedMethodData を空のリストにします。
3. リクエストIDを発行します。
    1. もし details.id が無ければ details に id を追加し、この決済請求を一意に特定できるような文字列をセットします。この文字列は UUID (A Universally Unique IDentifier URN Namespace) であることが推奨されています。
4. payment methods を処理します。
    1. methodData シーケンスの長さが0なら TypeError を投げ、最低一つの payment method が必要であることをデベロッパーに伝えます。
    2. methodData のそれぞれの paymentMethod について
        1. paymentMethod.supportedMethods シーケンスの長さが0なら TypeError を投げ、各 payment method は最低一つの payment method 識別子が必要であることをデベロッパーに伝えます。
        2. paymentMethod が存在するとき serializedData は paymentMethod.data をJSONシリアライズした結果（文字列）にします。存在しなければ null を返し、何か例外を投げます。
        3. serializedMethodData にタプル (paymentMethod.supportedMethods, serializedData) を追加します。
5. total を処理します。
    1. details.total.amount.value が有効な10進数の金銭的な数値でないとき TypeError を投げ、無効な値であることをデベロッパーに伝えます。
    2. details.total.amount.value の始めの文字が U+002D HYPHEN-MINUS（-のこと）なら TypeError を投げ、total が負数にならないことをデベロッパーに伝えます。
6. details.displayItems が存在するとき、 details.displayItems のそれぞれについて
    1. item.amount.value が有効な10進数の金銭的な数値でないとき TypeError を投げ、無効な値であることをデベロッパーに伝えます。
7. selectedShippingOption を null にします。
8. 配送オプションを処理します。
    1. options を空のシーケンス <PaymentShippingOption> にします。
    2. details.shippingOptions が存在するとき
        1. seenIDs を空のリストにします。
        2. options を details.shippingOptions とします。
        3. options のそれぞれの option について
            1. option.amount.value が有効な10進数の金銭的な数値でないとき TypeError を投げ、無効な値であることをデベロッパーに伝えます。
            2. seenIDs が option.id をもっているとき、options を空のシーケンスにして break します。
            3. option.id を seenIDs に追加します。
        4. options のそれぞれの option について（前の手順で空のシーケンスにリセットされているもの）
            1. option.selected が true なら、option.id に selectedShippingOption をセットします。
    3. options を details.shippingOptions とします。
9. serializedModifierData を空のリストにします。
10. payment details modifiers を処理します。
    1. modifiers を空のシーケンス <PaymentDetailsModifier> にします。
    2. details.modifiers が存在するとき
        1. details.modifiers に modifiers をセットします。
        2. modifiers のそれぞれの modifier について
            1. modifier に total が存在するとき
                1. value を modifier.total.amount.value にします。
                2. value が有効な10進数の金銭的な数値でないとき TypeError を投げ、無効な値であることをデベロッパーに伝えます。
                3. value の始めの文字が U+002D HYPHEN-MINUS（-のこと）なら TypeError を投げ、total が負数にならないことをデベロッパーに伝えます。
            2. additionalDisplayItems が存在するとき、 modifier.additionalDisplayItems のそれぞれについて
                1. value を item.amount.value にします。
                2. valueが有効な10進数の金銭的な数値でないとき TypeError を投げ、無効な値であることをデベロッパーに伝えます。
        3. serializedData は modifier.data をJSONシリアライズした結果（文字列）にします。存在しなければ null を返し、何か例外を投げます。
        4. serializedModifierData に serializedData を追加します。
        5. modifier に data が存在すれば削除します。
    3. modifiers に details.modifiers をセットします。
11. request を新しい PaymentRequest にします。
12. request.[[options]] に options をセットします。
13. request.[[state]] に "created" をセットします。
14. request.[[updating]] に false をセットします。
15. request.[[details]] に details をセットします。
16. request.[[serializedModifierData]] に serializedModifierData をセットします。
17. request.[[serializedMethodData]] に serializedMethodData をセットします。
18. selectedShippingOption に request の shippingOption 属性の値をセットします。
19. request の shippingAddress を null にします。
20. request の shippingType を null にします。
21. request を返します。

## 3.2 id attribute

取得すると、 id 属性は PaymentRequest の [[details]].id を返します。
