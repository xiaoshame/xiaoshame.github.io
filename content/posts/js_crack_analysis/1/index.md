---
title : "网页破解系列1" 
date : "2024-05-30T16:49:11+08:00" 
lastmod : "2024-05-30T16:49:11+08:00" 
tags : ["js","破解"] 
categories : ["技术"]
draft : false
featuredImage : /images/posts/js_crack_analysis/1/featuredImage.jpg
summary : 'JS破解思路记录'
---

最近分析一个网站，里面使用的加密方式有点意思，记录下分析流程

## 详解

### 加密代码

F12查看源代码，核心加密代码如下：

```js
let d = window.eval(
  (function (p, a, c, k, e, d) {
    e = function (c) {
      return (
        (c < a ? "" : e(parseInt(c / a))) +
        ((c = c % a) > 35 ? String.fromCharCode(c + 29) : c.toString(36))
      );
    };
    if (!"".replace(/^/, String)) {
      while (c--) d[e(c)] = k[c] || e(c);
      k = [
        function (e) {
          return d[e];
        },
      ];
      e = function () {
        return "\\w+";
      };
      c = 1;
    }
    while (c--)
      if (k[c]) p = p.replace(new RegExp("\\b" + e(c) + "\\b", "g"), k[c]);
    return p;
  })(
    'V.Q({"z":7,"y":"x","w":"7.2","v":u,"t":"6","s":["3%4%r.2.5","3%4%q.2.5","3%4%p.2.5","3%4%o.2.5","3%4%n.2.5","3%4%l.2.5","3%4%k.2.5","3%4%j.2.5","3%4%8.2.5","3%4%9.2.5","3%4%a.2.5","3%4%b.2.5","3%4%f.2.5","3%4%g.2.5","3%4%h.2.5","3%4%i.2.5","3%4%A.2.5","3%4%B.2.5","3%4%M.2.5","3%4%D.2.5","3%4%C.2.5","3%4%R.2.5","3%4%S.2.5","3%4%P.2.5","3%4%U.2.5"],"10":W,"X":Y,"Z":"/T/d/N/O/6/","L":1,"K":"","J":I,"H":0,"G":{"e":F,"m":"E"}}).c();',
    62,
    63,
    "D7BWAcHNgJQewEYEsB2wBMAGbngHcBTBcYQGm9tB3ZWAGZ0A2TdDbAdmcwA52BOZgRlzgATgQCSKJABcQ/PvyZY+1fgBZ2ddgFZ2a9sqyN2cg5gCG7XADMkAGwIBnYAGMUpgLYFgKlXxXdlTkgAJsDESE7AgOr+gH3aAGpqCK4eocH82ooairyKXJh8LACOANYA0jAFeAAWAMJ0ADIlBaD2pgRIRQCCwPn53NwcfHRq9jbAwgQAbqIh3r7caigEAB6S06E2cE5FAPpOEfaSppIAro6KbEFOu3Bu4Y5CiKh8ffqMTEhukAAiR+ZYWMx0HJwPYFIxlABlACyAAlgJZTDZ7J47Gh0NpwEdKvDUEh7JUCEEgA==="[
      "\x73\x70\x6c\x69\x63"
    ]("\x7c"),
    0,
    {}
  )
);
```

### 分析思路

```js
"D7BWAcHNgJQewEYEsB2wBMAGbngHcBTBcYQGm9tB3ZWAGZ0A2TdDbAdmcwA52BOZgRlzgATgQCSKJABcQ/PvyZY+1fgBZ2ddgFZ2a9sqyN2cg5gCG7XADMkAGwIBnYAGMUpgLYFgKlXxXdlTkgAJsDESE7AgOr+gH3aAGpqCK4eocH82ooairyKXJh8LACOANYA0jAFeAAWAMJ0ADIlBaD2pgRIRQCCwPn53NwcfHRq9jbAwgQAbqIh3r7caigEAB6S06E2cE5FAPpOEfaSppIAro6KbEFOu3Bu4Y5CiKh8ffqMTEhukAAiR+ZYWMx0HJwPYFIxlABlACyAAlgJZTDZ7J47Gh0NpwEdKvDUEh7JUCEEgA==="["\x73\x70\x6c\x69\x63"]("\x7c")
```

1. 核心加密代码中p就是解密后的数据，整体代码不复杂，核心是输入参数，即上面代码。
2. 咨询comate提示是对一段base64编码数据使用splic('|')函数。使用base64进行解码失败。
3. 继续在网页中分析，将上面代码单独执行，单步进入F11执行得到如下代码，意外的惊喜。

```js
var LZString = (function() {
    var f = String.fromCharCode;
    var keyStrBase64 = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=";
    var baseReverseDic = {};
    function getBaseValue(alphabet, character) {
        if (!baseReverseDic[alphabet]) {
            baseReverseDic[alphabet] = {};
            for (var i = 0; i < alphabet.length; i++) {
                baseReverseDic[alphabet][alphabet.charAt(i)] = i
            }
        }
        return baseReverseDic[alphabet][character]
    }
    var LZString = {
        decompressFromBase64: function(input) {
            if (input == null)
                return "";
            if (input == "")
                return null;
            return LZString._0(input.length, 32, function(index) {
                return getBaseValue(keyStrBase64, input.charAt(index))
            })
        },
        _0: function(length, resetValue, getNextValue) {
            var dictionary = [], next, enlargeIn = 4, dictSize = 4, numBits = 3, entry = "", result = [], i, w, bits, resb, maxpower, power, c, data = {
                val: getNextValue(0),
                position: resetValue,
                index: 1
            };
            for (i = 0; i < 3; i += 1) {
                dictionary[i] = i
            }
            bits = 0;
            maxpower = Math.pow(2, 2);
            power = 1;
            while (power != maxpower) {
                resb = data.val & data.position;
                data.position >>= 1;
                if (data.position == 0) {
                    data.position = resetValue;
                    data.val = getNextValue(data.index++)
                }
                bits |= (resb > 0 ? 1 : 0) * power;
                power <<= 1
            }
            switch (next = bits) {
            case 0:
                bits = 0;
                maxpower = Math.pow(2, 8);
                power = 1;
                while (power != maxpower) {
                    resb = data.val & data.position;
                    data.position >>= 1;
                    if (data.position == 0) {
                        data.position = resetValue;
                        data.val = getNextValue(data.index++)
                    }
                    bits |= (resb > 0 ? 1 : 0) * power;
                    power <<= 1
                }
                c = f(bits);
                break;
            case 1:
                bits = 0;
                maxpower = Math.pow(2, 16);
                power = 1;
                while (power != maxpower) {
                    resb = data.val & data.position;
                    data.position >>= 1;
                    if (data.position == 0) {
                        data.position = resetValue;
                        data.val = getNextValue(data.index++)
                    }
                    bits |= (resb > 0 ? 1 : 0) * power;
                    power <<= 1
                }
                c = f(bits);
                break;
            case 2:
                return ""
            }
            dictionary[3] = c;
            w = c;
            result.push(c);
            while (true) {
                if (data.index > length) {
                    return ""
                }
                bits = 0;
                maxpower = Math.pow(2, numBits);
                power = 1;
                while (power != maxpower) {
                    resb = data.val & data.position;
                    data.position >>= 1;
                    if (data.position == 0) {
                        data.position = resetValue;
                        data.val = getNextValue(data.index++)
                    }
                    bits |= (resb > 0 ? 1 : 0) * power;
                    power <<= 1
                }
                switch (c = bits) {
                case 0:
                    bits = 0;
                    maxpower = Math.pow(2, 8);
                    power = 1;
                    while (power != maxpower) {
                        resb = data.val & data.position;
                        data.position >>= 1;
                        if (data.position == 0) {
                            data.position = resetValue;
                            data.val = getNextValue(data.index++)
                        }
                        bits |= (resb > 0 ? 1 : 0) * power;
                        power <<= 1
                    }
                    dictionary[dictSize++] = f(bits);
                    c = dictSize - 1;
                    enlargeIn--;
                    break;
                case 1:
                    bits = 0;
                    maxpower = Math.pow(2, 16);
                    power = 1;
                    while (power != maxpower) {
                        resb = data.val & data.position;
                        data.position >>= 1;
                        if (data.position == 0) {
                            data.position = resetValue;
                            data.val = getNextValue(data.index++)
                        }
                        bits |= (resb > 0 ? 1 : 0) * power;
                        power <<= 1
                    }
                    dictionary[dictSize++] = f(bits);
                    c = dictSize - 1;
                    enlargeIn--;
                    break;
                case 2:
                    return result.join('')
                }
                if (enlargeIn == 0) {
                    enlargeIn = Math.pow(2, numBits);
                    numBits++
                }
                if (dictionary[c]) {
                    entry = dictionary[c]
                } else {
                    if (c === dictSize) {
                        entry = w + w.charAt(0)
                    } else {
                        return null
                    }
                }
                result.push(entry);
                dictionary[dictSize++] = w + entry.charAt(0);
                enlargeIn--;
                w = entry;
                if (enlargeIn == 0) {
                    enlargeIn = Math.pow(2, numBits);
                    numBits++
                }
            }
        }
    };
    return LZString
}
)();
String.prototype.splic = function(f) {
    return LZString.decompressFromBase64(this).split(f)
}
;
```

### 分析总结

1. 阅读上面代码可知，string.prototype扩展了一个splic方法
2. ``function(index) {return getBaseValue(keyStrBase64, input.charAt(index))}``作为一个参数被传递到_0 使用
3. 核心代码隐藏在vm中执行
