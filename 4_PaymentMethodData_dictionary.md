# 4. PaymentMethodData dictionary
```
WebIDL
dictionary PaymentMethodData {
    required sequence<DOMString> supportedMethods;
             object              data;
};
```
PaymentMethodData ディクショナリは、サポートされる決済方法(payment method)および関連する決済方法に関する詳細データの集合を指し示すために使われます。
次のメンバは PaymentMethodData ディクショナリの一部です：

**supportedMethods** メンバ
> supportedMethods は、売り手のウェブサイトが許容する決済方法に対する決済方法識別子(payment method identifiers)を含んだ文字列のリストで、これは必須です。

**data** メンバ
> data はサポートされる決済方法によって必要になる可能性のある任意の情報を提供するためのオブジェクトです。もしこれを使用する場合、JSONでシリアライズした結果を格納してください。

