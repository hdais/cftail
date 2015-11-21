# cftail -  特定キーワードにマッチする行を色つき表示しながらtail-f

## 例

router1.logを常時監視しつつ行が追加されたら表示する。キーワード`LINK-3-UPDOWN` が含まれる行は赤く(`red`)表示する。

    cftail -F 'red red LINK-3-UPDOWN' router1.log

===

`-F`は複数指定可能である。`LINEPROTO-5-UPDOWN`では黄色、`LINK-3-UPDOWN`では赤で表示する。

    cftail -F 'yellow yellow LINEPROTO-5-UPDOWN' -F 'red red LINK-3-UPDOWN' router1.log
    
ルールは先に記載したほうが優先される。

===
`LINK-3-UPDOWN` が含まれる行は赤く(red)表示する。ただし、そのようなキーワードでも`GigabitEthernet2/0/1`が含まれる場合は緑 (`green`)で表示する。
ルールの1番目のアクション(`red`)はキーワードにマッチした場合の色、2番目のアクション (`green`)は、例外ルール `-E`にマッチした場合の色である。

    cftail -F 'red green LINK-3-UPDOWN' -E 'GigabitEthernet2/0/1' router1.log
    
===
キーワードは正規表現が指定できる。

    cftail -F 'red green LINEPROTO-5-UPDOWN|LINK-3-UPDOWN' -E 'GigabitEthernet2/0/[12]' router1.log

===
複数のファイルを同時にtail-fすることもできる。

    cftail -F 'red green LINK-3-UPDOWN' router1.log router2.log router3.log
    
===
ルールをファイルで与えることもできる。

    cftail -f rulefile.rule -e exceptionrule.erule router1.log

ルールファイルは以下のように各行にルールを記載する。

    # rulefile.rule
    red green LINEPROTO-5-UPDOWN
    red green LINEPROTO-5-UPDOWN
    red red HARDWARE


    # exceptionrule.erule
    GigabitEthernet2/0/1
    GigabitEthernet2/0/2
    Port-channel1
    
===

## 説明

    cftail [-r] [-F <rule>] [-f <rule_file>]
           [-E <exception_fule>] [-e <exception_rule_file>]
           file [ file ... ]

`file` を常時監視し、追加行が発生したらその行を標準出力に出力する。通常は端末のデフォルト色で出力するが、後述のようにルールを指定することで特定のキーワード (正規表現で指定)が含まれる行を別の色で表示させることができる。

## オプション

`-F <rule>`

キーワードと、それにマッチした時の表示方法のルールを指定する。`<rule>`の書式は
`<action> <action_when_exception> <regular_expression>` で、それぞれ表示色、"例外キーワード"を含む時の表示色、キーワードの正規表現である。例: `-F 'red green LINEPROTO-5-UPDOWN'`

`-E <exception_rule>`

"例外キーワード" を指定する。`<exception_rule>`は例外キーワードの正規表現である。例: `-E GigabitEthernet 1/2/2`

`-f <rule_file>`

キーワードとアクションをファイルから読み込む。ファイルは1行1ルールで、各行の書式は `-F` オプションの `<rule>`と同じ。

`-e <exception_rule_file>`

「例外キーワード」をファイルから読み込む。ファイルは1行1ルールで、各行の書式は `-E`オプションの `<exception_rule>と同じ。


