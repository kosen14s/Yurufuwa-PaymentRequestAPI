# 2. Definitions

 ある文字列が、次のような要素から構成されている場合、それは有効な10進数の、金銭的な数値である。

 1. これはなくてもいいが、 U+002D HYPHEN-MINUS（-のこと）が存在する場合、その値は負値である
 2. 一つかそれ以上の、U+0030 DIGIT ZERO(0のこと）からU+0039 DIGIT NINE（9のこと）までの文字
 3. これもなくてもいいが、ひとつの U+002E FULL STOP（.のこと）とそれに続く、ひとつかそれ以上の U+0030 DIGIT ZERO(0のこと）からU+0039 DIGIT NINE（9のこと）までの文字

NOTE: つまり次のような正規表現であらわされる。 `^-?[0-9]+(\.[0-9]+)?$`


