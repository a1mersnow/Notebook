来谈一谈JS中的编码有关的问题。

JS 中字符串的表示方式
```js
'z'
'\z'
'\172' //八进制
'\x7A' //十六进制
'\u007A'
'\u{7A}'
```

### escape
ascii -> ascii (不变)
ascii 扩展 -> '\x'变成'%'
unicode(utf-16) -> '%u457a'

### unescape
% -> \x or \u00  
%u457a -> \u457a

### encodeURIComponent
相当于`escape(unicodeToUTF8(str))`

### String.fromCharCode
0x11 -> \x11  
0xa1 -> \xa1  
0xa175 -> \ua175
