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

PaymentRequest は、渡される決済方法(payment method)の詳細データを含む methodData リスト、決済の詳細(details)、および決済オプションを使って構成されます。PaymentRequest コンストラクタに渡された methodData は呼び出し元の優先度順に並んでいるべきです。

NOTE: methodData シーケンスは、ウェブサイトが許容する決済方法の識別子(payment method identifiers)を持つ PaymentMethodData ディクショナリ、ならびに決済方法の詳細データを保持します。

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

`PaymentRequest(methodData, details, options)` コンストラクタは次のように振る舞う必要があります。

1. current settings object の responsible document が allowpaymentrequest 属性に示された機能の使用を許可していないとき、 SecurityError DOMException を投げます。
2. serializedMethodData を空のリストにします。
3. リクエストIDを発行します。
    1. もし details.id が存在しなければ details に id メンバを追加し、この決済請求を一意に特定できるような文字列をセットします。この文字列は UUID (A Universally Unique IDentifier URN Namespace) であることが推奨されています。
4. payment methods を処理します。
    1. methodData シーケンスの長さが0なら TypeError を投げ、最低一つの payment method が必要であることをデベロッパーに伝えます。
    2. methodData のそれぞれの paymentMethod について
        1. paymentMethod.supportedMethods シーケンスの長さが0なら TypeError を投げ、各 payment method は最低一つの payment method 識別子が必要であることをデベロッパーに伝えます。
        2. paymentMethod.data メンバが存在するときは serializedData を paymentMethod.data をJSONシリアライズした結果（文字列）にし、そうでない場合は null にします。例外があれば再スローしてください。
        3. serializedMethodData にタプル (paymentMethod.supportedMethods, serializedData) を追加します。
5. total を処理します。
    1. details.total.amount.value が有効な10進数の金銭的な数値でないとき TypeError を投げ、無効な値であることをデベロッパーに伝えます。
    2. details.total.amount.value の始めの文字が U+002D HYPHEN-MINUS（-のこと）なら TypeError を投げ、total が負数にならないことをデベロッパーに伝えます。
6. details.displayItems が存在するとき、 details.displayItems のそれぞれについて
    1. item.amount.value が有効な10進数の金銭的な数値でないとき TypeError を投げ、無効な値であることをデベロッパーに伝えます。
7. selectedShippingOption を null にします。
8. 配送オプションを処理します。
    1. options を空のシーケンス \<PaymentShippingOption\> にします。
    2. details.shippingOptions が存在するとき
        1. seenIDs を空のリストにします。
        2. options を details.shippingOptions とします。
        3. options のそれぞれの option について
            1. option.amount.value が有効な10進数の金銭的な数値でないとき TypeError を投げ、無効な値であることをデベロッパーに伝えます。
            2. seenIDs が option.id をもっているとき、options を空のシーケンスにして break します。
            3. option.id を seenIDs に追加します。
        4. options のそれぞれの option について（前の手順で空のシーケンスにリセットされているもの）
            1. option.selected が true なら、selectedShippingOption を option.id にします。
    3. options を details.shippingOptions とします。
9. serializedModifierData を空のリストにします。
10. payment details modifiers を処理します。
    1. modifiers を空のシーケンス \<PaymentDetailsModifier\> にします。
    2. details.modifiers が存在するとき
        1. modifiers をdetails.modifiers とします。
        2. modifiers のそれぞれの modifier について
            1. modifier に total メンバが存在するとき
                1. value を modifier.total.amount.value にします。
                2. value が有効な10進数の金銭的な数値でないとき TypeError を投げ、無効な値であることをデベロッパーに伝えます。
                3. value の始めの文字が U+002D HYPHEN-MINUS（-のこと）なら TypeError を投げ、total が負数にならないことをデベロッパーに伝えます。
            2. modifier.additionalDisplayItems が存在するとき、 modifier.additionalDisplayItems のそれぞれについて
                1. value を item.amount.value にします。
                2. valueが有効な10進数の金銭的な数値でないとき TypeError を投げ、無効な値であることをデベロッパーに伝えます。
        3. modifier.data が存在するとき、serializedData は modifier.data をJSONシリアライズした結果（文字列）にし、存在しなければ null にします。例外があれば再スローしてください。
        4. serializedModifierData に serializedData を追加します。
        5. modifier に data が存在すれば削除します。
    3. details.modifiers を　modifiers にします。
11. request を新しい PaymentRequest にします。
12. request.[[options]] に options をセットします。
13. request.[[state]] に "created" をセットします。
14. request.[[updating]] に false をセットします。
15. request.[[details]] に details をセットします。
16. request.[[serializedModifierData]] に serializedModifierData をセットします。
17. request.[[serializedMethodData]] に serializedMethodData をセットします。
18. selectedShippingOption に request の shippingOption 属性の値をセットします。
19. request の shippingAddress を null にします。
20. options.requestShippingがtrueであれば、request の shippingType 属性の値に option.shippingType をセットします。trueでない場合は null をセットします。
21. request を返します。

## 3.2 id attribute

取得すると、 id 属性は PaymentRequest の [[details]].id を返します。

## 3.3 show() method

NOTE:
show() メソッドはウェブページが決済要求のためユーザーインタラクションを望んだときに呼び出されます。
show() メソッドはユーザーが決済要求を受諾した時に解決される Promise を返します。
show() メソッドがリターンされたあと、決済要求を楽にするため、何かしらのユーザーインターフェースが表示されます。

show() メソッドは次のように振る舞わなければなりません。

1. request を abort() メソッドのレシーバである PaymentRequest オブジェクトとします。
2. もし request.[[state]] の値が、 "created" でない場合には "InvalidStateError" DOMException を送出します。
3. request.[[state]] を "interactive" に設定します。
4. acceptPromise をあたらしい Promise オブジェクトとします。
5. acceptPromise を request.[[acceptPromise]] に設定します。
6. 必要に応じて以下のことを行います。
    1. acceptPromise を "AbortError" DOMException とともにリジェクトします。
    2. request.[[state]] を "closed" に設定します。
    3. このアルゴリズムを取り消します。

    NOTE:
    これにより、ユーザーエージェントはよしなにユーザーがすぐに決済要求を取り消したように振る舞うことができます。
    例えばブラウザのプライベートモードなどで、ユーザーエージェントはこの手順をとることがあります。
7. acceptPromise をリターンし、以下のステップを並列に（parallelに）処理します。
8. request.[[serializedMethodData]] 内それぞれの paymentMethod メンバについて
    1. paymentMethod タプルの最初の要素によって与えられた決済方法識別子をサポートする決済アプリケーションを決めます。
       決済アプリケーションそれぞれについて、決済アプリケーションが提供する決済方法特有の機能がタプルの2番目の要素と一致する場合、決済アプリケーションはこれにマッチします。
9. サポートされた決済方法が見つからない場合は、 acceptPromise を "NotSupportedError" DOMException とともにリジェクトして、このアルゴリズムを取り消します。
10. それ以外の場合は、これまでの手順によって実行可能とされた決済アプリケーションと決済方法を使って、ユーザーが決済要求プロセスとやりとりできるユーザーインターフェースを表示します。
    ユーザーエージェントは supportedMethods で与えられた順番で決済方法を表示できますが、決済方法と決済アプリケーションを表示するときにはユーザーの優先順位を優先する必要があります。

    決済プロセスを通じてユーザーをガイドするため、決済アプリケーションは request から適切なデータを送信される必要があります。
    これは様々な属性と request 内部の値を含みます。

    acceptPromise はユーザーが決済要求アルゴリズムを受け入れるかユーザーインターフェースとのインタラクションを通して取り消すことによって、のちに解決またはリジェクトされます。

## 3.4 abort() method

NOTE:
 abort() メソッドはウェブページがユーザエージェントに対して、決済の要求を中止して、表示されているユーザインタフェースを取り消すことを望んだときに呼び出されます。 abort() メソッドは show() メソッドが呼ばれたあとで、かつ、そのインスタンスの [[acceptPromise]] が解決される前にのみ呼び出すことができます。たとえば、あるウェブページは、商品を限られた時間でのみ販売するためにこのメソッドを使うかもしれません。もし、ユーザが時間の制限内で決済要求を受諾しなければ、その取引は中止されます。

 ユーザエージェントはいつでもリクエストを中止できるわけではないかもしれません。たとえば、ユーザエージェントが、ほかのアプリケーションへのリクエストの責任を負っているような場合には、 abort() メソッドはリターンされたPromiseをリジェクトします。


 abort() メソッドは次のように振る舞わなければなりません。

1. request を abort() メソッドのレシーバである PaymentRequest オブジェクトとします。
2. もし request.[[state]] の値が、 "interactive" でない場合には "InvalidStateError" DOMException を送出します。
3. promise をあたらしい Promise オブジェクトとします
4. promise をリターンし、以下のステップを並列に（parallelに）処理します。
5. 現在のユーザとの対話処理の中止を試み、残っているユーザインタフェースをすべて畳みます。
6. 以下のステップを実行するような、ユーザインタラクションタスクソース上のタスクを、キューに追加します。
    1. もし、現在のユーザとの対話処理が中止不可能ならば、promise を "InvalidStateError" DOMException の発行とともにリジェクトし、以下のステップも中止します。
    2. 内部的な値である request.[[state]] を "closed" に設定します。
    3. request.[[accceptPromise]] promiseを "AbortError" DOMException とともにリジェクトします
    4. promise を undefined によって解決します

## 3.5 canMakePayment() method

NOTE: canMakePayment() メソッドは開発者がPaymentRequestオブジェクトを、決済を行うために利用できるかどうか、 show() メソッドを呼び出す前に確かめるために用いることができます。 canMakePayment() メソッドは Promise オブジェクトを返します。この Promise オブジェクトは、PaymentRequest のコンストラクタに対して供給される何かしらの決済方法をユーザエージェントがサポートしている場合には true 、サポートされていない場合には false が与えれられ Fullfilled となります。 もしも、このメソッドがあまりにも多くの回数呼ばれた場合には、ユーザエージェントは自身の裁量で Promise を "QuotaExceededError" DOMException とともに reject するかもしれません。 

canMakePayment() メソッドは次のように振る舞わなければなりません。

1. request を canMakePayment() メソッドのレシーバである PaymentRequest オブジェクトとします
2. request.[[state]] が "created" でなければ "InvalidSateteError" DOMException によってリジェクトされた Promise オブジェクトを返します。
3. ユーザエージェントは自身の裁量によって、 "QuotaExceededError" DOMException によってリジェクトされた Promise オブジェクトを返しても構いません。
NOTE: これはユーザエージェントに対して、ヒューリスティックに、フィンガープリントを取得する目的での canMakePayment() メソッドの乱用を検出し、それを防止することを許可します。 この canMakePayment() メソッドの乱用にはたとえば、 様々なサポートされている決済方法を使用して PaymentRequest オブジェクトを作成し、それらが使用できるかどうか canMakePayment() メソッドで確かめるという手法が考えられます。 ユーザエージェントはトップレベルのブラウジングコンテキストや、それらの呼び出しが行われた時間に基づいて、成功した呼び出しの数に制限をかけることができます。

4. promise を 新しい Promise オブジェクトとします。
5. promise を返します。そして、以下の残ったステップを並列に（parallel に）実行します。
6. request.[[serializedMethodData]] の各 methodData に対して、以下を行います。
    1. もし、 methodData.supportedMethods がユーザエージェントや、その他の決済アプリケーションのサポートする決済方法（これには決済方法の特定の機能も含みます）の識別子を含んでいる場合、 promise を true で解決し、このアルゴリズムを終了します。
7. promise を false で解決します。
