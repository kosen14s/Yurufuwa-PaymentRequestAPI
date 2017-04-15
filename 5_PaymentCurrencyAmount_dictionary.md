# 5. PaymentCurrencyAmount dictionary
```
WebIDL
dictionary PaymentCurrencyAmount {
    required DOMString currency;
    required DOMString value;
             DOMString currencySystem = "urn:iso:std:iso:4217";
};
```

PaymentCurrencyAmount ディクショナリは金額を渡すために使われます。

**currencySystem**
> 通貨識別子(currency identifier)が属する通貨システムを示すURLです。デフォルト値は、通貨が ISO4217（例えばUSDは米ドル）で定義されていることを表す "urn:iso:std:iso:4217" です。

**currency**
> 通貨識別子を持つ文字列です。通貨(currency)の値はcurrencySystemで指定された通貨システムに準拠する文字列であれば何でも構いません。

**value**
> 金額を持つ有効な10進数の金銭的な数値です。

55.00米ドルがどう表されるかの例を次に示します。
```
EXAMPLE 5
{
  "currency": "USD",
  "value" : "55.00"
}
```
