---
title : "网页破解系列2" 
date : "2024-06-04T19:10:48+08:00" 
lastmod : "2024-06-04T19:10:48+08:00" 
tags : ["js","破解"] 
categories : ["技术"]
draft : false
featuredImage : /images/posts/js_crack_analysis/2/featuredImage.jpg
summary : 'js中常见混淆案例解析'
---

最近分析网页阅读js代码的时候，代码混淆特别喜欢使用IIFE模式(立即调用的函数表达式)和函数嵌套 ,因为没有系统学习过js，导致理解上有些困难，下面记录下IIFE模式转换成常规js代码的过程。

### 案例1

```javascript
(function (_0x139810, _0x225121, _0x528ff3, _0x556706, _0x17b995, _0x2888eb, _0x613779) { 
	
	return _0x139810 = _0x139810 >> 0x8, _0x2888eb = 'hs', _0x613779 = 'hs', function (_0x5c8a1f, _0x204eba, _0x273121, _0x46f859, _0x572acf) { 
		const _0x1283bd = _0x52aa; 
		_0x46f859 = 'tfi';
		_0x2888eb = _0x46f859 + _0x2888eb;
		_0x572acf = 'up';
		_0x613779 += _0x572acf;
		_0x2888eb = _0x273121(_0x2888eb);
		_0x613779 = _0x273121(_0x613779)
		_0x273121 = 0x0;
		const _0x135ded = _0x5c8a1f();
		while (!![] && --_0x556706 + _0x204eba) {
			try { 
				_0x46f859 = -parseInt(_0x1283bd(0x1da, 'R3)f')) / 0x1 * (-parseInt(_0x1283bd(0x1ed, '*c&W')) / 0x2) + parseInt(_0x1283bd(0x1ee, 'pQTS')) / 0x3 + -parseInt(_0x1283bd(0x1f3, 'lNB3')) / 0x4 + -parseInt(_0x1283bd(0x1f8, 'IzuF')) / 0x5 + parseInt(_0x1283bd(0x1df, 'lsaR')) / 0x6 * (parseInt(_0x1283bd(0x1e2, '(ga@')) / 0x7) + -parseInt(_0x1283bd(0x1db, 'o%ET')) / 0x8 + parseInt(_0x1283bd(0x1e8, 'awuH')) / 0x9; 
			} 
			catch (_0x32a3ec) { 
				_0x46f859 = _0x273121;
			}
			finally { 
				_0x572acf = _0x135ded[_0x2888eb](); 
				if (_0x139810 <= _0x556706) _0x273121 ? _0x17b995 ? _0x46f859 = _0x572acf : _0x17b995 = _0x572acf : _0x273121 = _0x572acf; 
				else {
					 if (_0x273121 == _0x17b995['replace'](/[DMWQITftRwLHqlNxJKPr=]/g, '')) {
						 if (_0x46f859 === _0x204eba) { 
							_0x135ded['un' + _0x2888eb](_0x572acf); 
							break;
						} _0x135ded[_0x613779](_0x572acf);
					}
				}
			}
		}
	}(_0x528ff3, _0x225121, decode1);
}(0xc800, 0x3651f, _0x17af, 0xca), _0x17af) && (_0xodU = 0xca);
```

- 第一步从最内部函数开始，将function (_0x5c8a1f, _0x204eba, _0x273121, _0x46f859, _0x572acf)剥离出来
    - _0x46f859, _0x572acf参数内部赋值，可以理解为变量声明
    - _0x2888eb,_0x613779,_0x556706为上层函数声明，作为函数参数参入
    - 调整后整体函数如下

```javascript
function decode2(_0x5c8a1f, _0x204eba, _0x273121,_0x139810,_0x2888eb,_0x613779,_0x556706) { 
	const _0x1283bd = _0x52aa;
	var _0x17b995 ;
	_0x46f859 = 'tfi';
	_0x2888eb = _0x46f859 + _0x2888eb;
	_0x572acf = 'up';
	_0x613779 += _0x572acf;
	_0x2888eb = _0x273121(_0x2888eb);
	_0x613779 = _0x273121(_0x613779)
	_0x273121 = 0x0;
	const _0x135ded = _0x5c8a1f;
	while (!![] && --_0x556706 + _0x204eba) {
		try { 
			_0x46f859 = -parseInt(_0x1283bd(0x1da, 'R3)f')) / 0x1 * (-parseInt(_0x1283bd(0x1ed, '*c&W')) / 0x2) + parseInt(_0x1283bd(0x1ee, 'pQTS')) / 0x3 + -parseInt(_0x1283bd(0x1f3, 'lNB3')) / 0x4 + -parseInt(_0x1283bd(0x1f8, 'IzuF')) / 0x5 + parseInt(_0x1283bd(0x1df, 'lsaR')) / 0x6 * (parseInt(_0x1283bd(0x1e2, '(ga@')) / 0x7) + -parseInt(_0x1283bd(0x1db, 'o%ET')) / 0x8 + parseInt(_0x1283bd(0x1e8, 'awuH')) / 0x9; 
		} 
		catch (_0x32a3ec) { 
			_0x46f859 = _0x273121;
		}
		finally { 
			_0x572acf = _0x135ded[_0x2888eb](); 
			if (_0x139810 <= _0x556706) _0x273121 ? _0x17b995 ? _0x46f859 = _0x572acf : _0x17b995 = _0x572acf : _0x273121 = _0x572acf; 
			else {
				 if (_0x273121 == _0x17b995['replace'](/[DMWQITftRwLHqlNxJKPr=]/g, '')) {
					 if (_0x46f859 === _0x204eba) { 
						_0x135ded['un' + _0x2888eb](_0x572acf); 
						break;
					} _0x135ded[_0x613779](_0x572acf);
				}
			}
		}
	}
}

(function (_0x139810, _0x225121, _0x528ff3, _0x556706, _0x17b995, _0x2888eb, _0x613779) { 
	
	_0x139810 = _0x139810 >> 0x8, _0x2888eb = 'hs', _0x613779 = 'hs';
	return decode2(_0x528ff3, _0x225121, decode1,_0x139810,_0x2888eb,_0x613779,_0x556706);
}(0xc800, 0x3651f, _0x17af, 0xca), _0x17af) && (_0xodU = 0xca);

```
- 第二步将function (_0x139810, _0x225121, _0x528ff3, _0x556706, _0x17b995, _0x2888eb, _0x613779)剥离出来
    - _0x2888eb,_0x613779 声明为内部变量,_0x17b995 参数无用

```javascript
function decode3(_0x139810, _0x225121, _0x528ff3, _0x556706){
	_0x139810 = _0x139810 >> 0x8, _0x2888eb = 'hs', _0x613779 = 'hs';
	return decode2(_0x528ff3, _0x225121, decode1,_0x139810,_0x2888eb,_0x613779,_0x556706);
}

(decode3(0xc800, 0x3651f, _0x17af, 0xca), _0x17af) && (_0xodU = 0xca);
```

- 第三步将``, _0x17af) && (_0xodU = 0xca)``等代码为无用代码
    - 在函数中直接调用 ``decode3(0xc800, 0x3651f, _0x17af, 0xca)``即可

### 案例2

```javascript
var _0xodU = 'jsjiami.com.v7';
function _0x17af() {
	const _0x45cdd6 = (function () {
		return [_0xodU, 'qwjsTHjRiRWMatmLixJ.cwtKoQrfm.DtrvIN7lMP==', 'WPK4W5jqvW', 'WP3dRu/dGhW', 'WPFdNSkZWQ4oWOVcSeVcOfXQW40', 'hCkECbVdQSozea', 'W5nddCkvaSkSaSkOW65Pfx0', 'WR7dV8kTlKC', 'WPieWRWcwG', 'W5aQq8kLWRBcNG', 'W6jlimoupZS', 'W41eWRiCn8k7aCkezhjssG', 'W6ldGmkEdCkljCkvW4JdQ2z5WRC'].concat((function () { 
			return ['WOJcQd7dHmo3rf7cOmk0WOKDW5m', 'cSoiW6esWQP4WOCJtW', 'fLtcPHFcItz5W7tcI8o/sNe', 'lXr1dmkKlCoLWOWfW6NcGsS', 'WPldR8oWjgtdQ8kBwHtcTHqR', 'ASk2WQG5WRG', 'W4HXomoSyCoa', 'W4JcP8ooW4ldHwddMshdOH/dM8kU', 'AMVdKZv/W4yGW5v3WR86W4yu', 'WOnzw8oZhCk/hKudW4DfWP8', 'jKtcNmkUf2nmW47cHMZcJJK', 'FxlcNSkAW6BcGSkDDmorBNb2', 'WObbn8kZyYZdJCkVECkH'].concat((function () { 
				return ['rCozkrrbtmkEhSkXW5vvdq', 'W5exw8k1WRK', 'W5ZdQwNdSCoxyxNcK8kC', 'WQe8Fuia', 'dqidWQ0Rd8ox', 'W4LeWRSsrmornSkezLG', 'WRJdVmkRW63dUG', 'W4S8zCo/uG', 'ysddKMJcRmkRW5NcJt4BlhJdJW', 'tmoDBhSp'];
			}()));
		}()));
	}());
	_0x17af = function () { 
		return _0x45cdd6; 
	};
	return _0x17af();
}
```

- 这个案例通过concat函数将一个数组混淆成了一个函数，还原后如下
- 在调用_0x17af()函数的地方，修改为_0x17af即可

```javascript
var _0x17af = ['jsjiami.com.v7', 'qwjsTHjRiRWMatmLixJ.cwtKoQrfm.DtrvIN7lMP==', 'WPK4W5jqvW', 'WP3dRu/dGhW', 'WPFdNSkZWQ4oWOVcSeVcOfXQW40', 'hCkECbVdQSozea', 'W5nddCkvaSkSaSkOW65Pfx0', 'WR7dV8kTlKC', 'WPieWRWcwG', 'W5aQq8kLWRBcNG', 'W6jlimoupZS', 'W41eWRiCn8k7aCkezhjssG', 'W6ldGmkEdCkljCkvW4JdQ2z5WRC','WOJcQd7dHmo3rf7cOmk0WOKDW5m', 'cSoiW6esWQP4WOCJtW', 'fLtcPHFcItz5W7tcI8o/sNe', 'lXr1dmkKlCoLWOWfW6NcGsS', 'WPldR8oWjgtdQ8kBwHtcTHqR', 'ASk2WQG5WRG', 'W4HXomoSyCoa', 'W4JcP8ooW4ldHwddMshdOH/dM8kU', 'AMVdKZv/W4yGW5v3WR86W4yu', 'WOnzw8oZhCk/hKudW4DfWP8', 'jKtcNmkUf2nmW47cHMZcJJK', 'FxlcNSkAW6BcGSkDDmorBNb2', 'WObbn8kZyYZdJCkVECkH','rCozkrrbtmkEhSkXW5vvdq', 'W5exw8k1WRK', 'W5ZdQwNdSCoxyxNcK8kC', 'WQe8Fuia', 'dqidWQ0Rd8ox', 'W4LeWRSsrmornSkezLG', 'WRJdVmkRW63dUG', 'W4S8zCo/uG', 'ysddKMJcRmkRW5NcJt4BlhJdJW', 'tmoDBhSp'];
```

### 案例3

```javascript
function _0x52aa(_0x3c83db, _0x469b49) {
	const _0x17af80 = _0x17af();
	return _0x52aa = function (_0x52aa41, _0x3728c1) {
		_0x52aa41 = _0x52aa41 - 0x1da;
		let _0x34a1cf = _0x17af80[_0x52aa41];
		if (_0x52aa['WwUxJQ'] === undefined) {
			var _0x1dd466 = function (_0x4b772e) {
				const _0x32b02a = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789+/=';
				let _0x4a56a0 = '', _0x56bd5b = '';
				for (let _0x12059c = 0x0, _0x463da3, _0x592aa3, _0x127494 = 0x0; _0x592aa3 = _0x4b772e['charAt'](_0x127494++); ~_0x592aa3 && (_0x463da3 = _0x12059c % 0x4 ? _0x463da3 * 0x40 + _0x592aa3 : _0x592aa3, _0x12059c++ % 0x4) ? _0x4a56a0 += String['fromCharCode'](0xff & _0x463da3 >> (-0x2 * _0x12059c & 0x6)) : 0x0) {
					_0x592aa3 = _0x32b02a['indexOf'](_0x592aa3);
				}
				for (let _0x4b1bb9 = 0x0, _0x261b0c = _0x4a56a0['length']; _0x4b1bb9 < _0x261b0c; _0x4b1bb9++) {
					_0x56bd5b += '%' + ('00' + _0x4a56a0['charCodeAt'](_0x4b1bb9)['toString'](0x10))['slice'](-0x2);
				}
				return decodeURIComponent(_0x56bd5b);
			};
			const _0x1809cd = function (_0x33deb3, _0x3c0f88) {
				let _0x8e9e97 = [], _0x3b4eb2 = 0x0, _0x2ebcde, _0x514c3b = '';
				_0x33deb3 = _0x1dd466(_0x33deb3);
				let _0x35889e;
				for (_0x35889e = 0x0; _0x35889e < 0x100; _0x35889e++) {
					_0x8e9e97[_0x35889e] = _0x35889e; 
				}
				for (_0x35889e = 0x0; _0x35889e < 0x100; _0x35889e++) {
					_0x3b4eb2 = (_0x3b4eb2 + _0x8e9e97[_0x35889e] + _0x3c0f88['charCodeAt'](_0x35889e % _0x3c0f88['length'])) % 0x100, _0x2ebcde = _0x8e9e97[_0x35889e], _0x8e9e97[_0x35889e] = _0x8e9e97[_0x3b4eb2], _0x8e9e97[_0x3b4eb2] = _0x2ebcde;
				}
				_0x35889e = 0x0, _0x3b4eb2 = 0x0;
				for (let _0x8d4ec6 = 0x0; _0x8d4ec6 < _0x33deb3['length']; _0x8d4ec6++) { 
					_0x35889e = (_0x35889e + 0x1) % 0x100, _0x3b4eb2 = (_0x3b4eb2 + _0x8e9e97[_0x35889e]) % 0x100, _0x2ebcde = _0x8e9e97[_0x35889e], _0x8e9e97[_0x35889e] = _0x8e9e97[_0x3b4eb2], _0x8e9e97[_0x3b4eb2] = _0x2ebcde, _0x514c3b += String['fromCharCode'](_0x33deb3['charCodeAt'](_0x8d4ec6) ^ _0x8e9e97[(_0x8e9e97[_0x35889e] + _0x8e9e97[_0x3b4eb2]) % 0x100]);
				}
				return _0x514c3b;
			};
			_0x52aa['JzzYJi'] = _0x1809cd, _0x3c83db = arguments, _0x52aa['WwUxJQ'] = !![];
		}
		const _0xd630f0 = _0x17af80[0x0], _0x45e113 = _0x52aa41 + _0xd630f0, _0x4c2862 = _0x3c83db[_0x45e113]; 
		return !_0x4c2862 ? (_0x52aa['usUPJs'] === undefined && (_0x52aa['usUPJs'] = !![]), _0x34a1cf = _0x52aa['JzzYJi'](_0x34a1cf, _0x3728c1), _0x3c83db[_0x45e113] = _0x34a1cf) : _0x34a1cf = _0x4c2862, _0x34a1cf;
	}, _0x52aa(_0x3c83db, _0x469b49);
}
```
- _0x1809cd,_0x1dd466,_0x52aa 进行了函数声明单独拎出来
- 'undefined' 使用null替换,'arguments' 使用0替换
- _0x52aa函数内声明了_0x52aa数组，可以将_0x52aa数组变量名替换为data方便阅读
- 调整后代码如下

```javascript
function decode4(_0x4b772e){
	const _0x32b02a = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789+/=';
	let _0x4a56a0 = '', _0x56bd5b = '';
	for (let _0x12059c = 0x0, _0x463da3, _0x592aa3, _0x127494 = 0x0; _0x592aa3 = _0x4b772e['charAt'](_0x127494++); ~_0x592aa3 && (_0x463da3 = _0x12059c % 0x4 ? _0x463da3 * 0x40 + _0x592aa3 : _0x592aa3, _0x12059c++ % 0x4) ? _0x4a56a0 += String['fromCharCode'](0xff & _0x463da3 >> (-0x2 * _0x12059c & 0x6)) : 0x0) {
		_0x592aa3 = _0x32b02a['indexOf'](_0x592aa3);
	}
	for (let _0x4b1bb9 = 0x0, _0x261b0c = _0x4a56a0['length']; _0x4b1bb9 < _0x261b0c; _0x4b1bb9++) {
		_0x56bd5b += '%' + ('00' + _0x4a56a0['charCodeAt'](_0x4b1bb9)['toString'](0x10))['slice'](-0x2);
	}
	return decodeURIComponent(_0x56bd5b);
};

function decode5(_0x33deb3, _0x3c0f88) {
	let _0x8e9e97 = [], _0x3b4eb2 = 0x0, _0x2ebcde, _0x514c3b = '';
	_0x33deb3 = decode4(_0x33deb3);
	let _0x35889e;
	for (_0x35889e = 0x0; _0x35889e < 0x100; _0x35889e++) {
		_0x8e9e97[_0x35889e] = _0x35889e; 
	}
	for (_0x35889e = 0x0; _0x35889e < 0x100; _0x35889e++) {
		_0x3b4eb2 = (_0x3b4eb2 + _0x8e9e97[_0x35889e] + _0x3c0f88['charCodeAt'](_0x35889e % _0x3c0f88['length'])) % 0x100, _0x2ebcde = _0x8e9e97[_0x35889e], _0x8e9e97[_0x35889e] = _0x8e9e97[_0x3b4eb2], _0x8e9e97[_0x3b4eb2] = _0x2ebcde;
	}
	_0x35889e = 0x0, _0x3b4eb2 = 0x0;
	for (let _0x8d4ec6 = 0x0; _0x8d4ec6 < _0x33deb3['length']; _0x8d4ec6++) { 
		_0x35889e = (_0x35889e + 0x1) % 0x100, _0x3b4eb2 = (_0x3b4eb2 + _0x8e9e97[_0x35889e]) % 0x100, _0x2ebcde = _0x8e9e97[_0x35889e], _0x8e9e97[_0x35889e] = _0x8e9e97[_0x3b4eb2], _0x8e9e97[_0x3b4eb2] = _0x2ebcde, _0x514c3b += String['fromCharCode'](_0x33deb3['charCodeAt'](_0x8d4ec6) ^ _0x8e9e97[(_0x8e9e97[_0x35889e] + _0x8e9e97[_0x3b4eb2]) % 0x100]);
	}
	return _0x514c3b;
};

function _0x52aa(_0x52aa41, _0x3728c1) {
	let data = [];
	const _0x17af80 = _0x17af;
	_0x52aa41 = _0x52aa41 - 0x1da;
	let _0x34a1cf = _0x17af80[_0x52aa41];
	if (data['WwUxJQ'] == null) {
		const _0x1809cd = decode5;
		data['JzzYJi'] = _0x1809cd, _0x3c83db = [], data['WwUxJQ'] = !![];
	}
	const _0xd630f0 = _0x17af80[0x0], _0x45e113 = _0x52aa41 + _0xd630f0, _0x4c2862 = _0x3c83db[_0x45e113]; 
	return !_0x4c2862 ? (data['usUPJs'] == null && (data['usUPJs'] = !![]), _0x34a1cf = data['JzzYJi'](_0x34a1cf, _0x3728c1), _0x3c83db[_0x45e113] = _0x34a1cf) : _0x34a1cf = _0x4c2862, _0x34a1cf;
}
```

### 总结
1. 函数嵌套不利于理解代码，将嵌套的函数独立出来提高可读性
2. IIFE结合全局变量可隐藏全局变量原始值
3. 无意义变量名极大降低了可读性，可批量替换变量名提高可读性
4. 函数改造结合实时调试，可提高改造效率