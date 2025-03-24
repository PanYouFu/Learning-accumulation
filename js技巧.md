> js会用到的小套路

### 生成随机字符串

#### 使用Math.random().toString(36)

```javascript
Math.random().toString(36).slice(-8)
```

```javascript
Math.random()                        // 生成随机数字, eg: 0.123456
             .toString(36)           // 转化成36进制 : "0.4fzyo82mvyr"
                          .slice(-8);// 截取最后八位 : "yo82mvyr"
```

缺点：

- 只能生成有 0-9、a-z字符组成的字符串
- 由于 `Math.random()`生成的18位小数，可能无法填充36位，最后几个字符串，只能在指定的几个字符中选择。导致随机性降低。
- 某些情况下会返回空值。例如，当随机数为 0, 0.5, 0.25, 0.125...时，返回为空值。

#### 指定字符集

```javascript
function randomStr(len, chars) {
  let res = ''
  for (let i=0; i<len; i++) {
    res += chars[Math.floor(Math.random() * len)]
  }

  return res
}

var rString = randomString(32, '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ');
// 'juqag84uquqqol47s2ofs7io2egbe4uv'
```
