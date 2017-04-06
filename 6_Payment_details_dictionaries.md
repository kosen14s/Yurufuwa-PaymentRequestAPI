# 6. Payment details dictionaries
## 6.1 PaymentDetailsBase dictionary
```
WebIDL
dictionary PaymentDetailsBase {
    sequence<PaymentItem>            displayItems;
    sequence<PaymentShippingOption>  shippingOptions;
    sequence<PaymentDetailsModifier> modifiers;
};
```

次のメンバは PaymentDetailsBase ディクショナリの一部です：

**displayItems**
> この PaymentItem ディクショナリのリストは、ユーザーエージェントが表示する可能性のある決済要求の詳細項目を持ちます。例えば、このメンバは製品の詳細や、税や送料の内訳を含むかもしれません。
> ```
> NOTE:
> このメンバから計算した総計が、後に示す total と一致するかを確認するのは開発者側の責任です。
> ```

**shippingOptions**
> ユーザーが選択できる異なる配送オプションを持つリストです。もしこのリストが空であれば、それは売り手が現在の shippingAddress に配送できないことを意味します。
> もしこのリスト中のあるアイテムの selected メンバが true のとき、それがデフォルトで使用される配送オプションとなります。またそのとき、後に説明される配送オプション変更アルゴリズム（shipping option changed algorithm）を動かすことなく、shippingOption はそのオプションの id に設定されます。そのため、設定時に複数の selected メンバの値を true に**してはいけません**。万が一そのような状態になったとき、ユーザーエージェントは**必ず**リスト内で最後に現れたものを選ぶようにしてください。
> shippingOptions メンバは、PaymentOptions.requestShipping が true の状態の PaymentRequest が作られたときにのみ使用してください。
> ```
> NOTE:
> もしリストが、selected メンバが true のアイテムを持つ場合、コーダーは total メンバが配送オプションの料金を含んでいることを保証する責任があります。これは、ユーザーが最初に別のオプションを選ばない限り shippingoptionchange イベントが呼び出されることがないからです。
> ```

**modifiers**
> この PaymentDetailsModifier ディクショナリのリストは特定の決済方法識別子(payment method identifiers)に対する修飾子を持ちます。例えば、決済方法に合わせて合計金額を調整することができます。

