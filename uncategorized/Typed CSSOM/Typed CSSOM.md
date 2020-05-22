这是一种新的 `CSSOM`，可以让我们更加方便的去查询和修改样式。
```js
// Element styles.
el.attributeStyleMap.set('opacity', 0.3);
typeof el.attributeStyleMap.get('opacity').value === 'number' // Yay, a number!

// Stylesheet rules.
const stylesheet = document.styleSheets[0];
stylesheet.cssRules[0].styleMap.set('background', 'blue');
```
像这样通过元素的 `attributeMap` 或者 styleSheet rule 的 `styleMap` 就可以获取到一个 Map-like 的对象，支持常用的 map 类型的操作：get/set/keys/values/entries/clear。

支持数字格式的CSS属性，最终将被转换为数字。


### Typed CSSOM 尝试解决的问题
- 更少的bug
- 计算值可以被自动拦截和取整
- 更好的性能，带来更高效的JS动画
- 通过解析函数为CSS世界带来错误处理机制
- STYLEMAP.(g|s)et('css_property_name') 而不是为 `STYLE.backgroundColor` 还是 `STYLE['background-color']` 纠结，更加统一

### 浏览器侦测
```js
if (window.CSS && CSS.number) {
  // Supports CSS Typed OM
}
```

### 基础的 API

##### 获取样式
在 `Typed CSSOM` 中，值和单位是分离的，一个简单的样式就是一个 `CSSUnitValue`，包含一个 `value` 和一个 `unit` 属性。
```js
el.attributeStyleMap.set('margin-top', CSS.px(10));
// el.attributeStyleMap.set('margin-top', '10px'); // string arg also works.
el.attributeStyleMap.get('margin-top').value  // 10
el.attributeStyleMap.get('margin-top').unit // 'px'

// Use CSSKeyWorldValue for plain text values:
el.attributeStyleMap.set('display', new CSSKeywordValue('initial'));
el.attributeStyleMap.get('display').value // 'initial'
el.attributeStyleMap.get('display').unit // undefined
```
##### 计算样式

**Old CSSOM**
```js
el.style.opacity = 0.5;
window.getComputedStyle(el).opacity === "0.5" // Ugh, more strings!
```
**New Typed CSSOM**
```js
el.attributeStyleMap.set('opacity', 0.5);
el.computedStyleMap().get('opacity').value // 0.5
```
需要注意的一个重要区别是， `window.getComputedStyle()` 返回的是一个转换后的值，`element.computedStyleMap()` 返回的是一个计算值，举例来说，一个百分比的宽度，Typed OM 会返回百分比，而 CSSOM 会返回一个计算后的像素值。

*计算值才会被拦截*
```js
el.attributeStyleMap.set('opacity', 3);
el.attributeStyleMap.get('opacity').value === 3  // val not clamped.
el.computedStyleMap().get('opacity').value === 1 // computed style clamps value.

el.attributeStyleMap.set('z-index', CSS.number(15.4));
el.attributeStyleMap.get('z-index').value  === 15.4 // val not rounded.
el.computedStyleMap().get('z-index').value === 15   // computed style is rounded.
```

### 数字值
数字值（`CSSNumericValue`）有两种：`CSSUnitValue` 和 `CSSMathValue`。

##### Unit values
简单的如 `50%` 的值使用 `CSSUnitValue` 表示，你可以使用构造函数 `new CSSUnitValue(10, 'px')` 来创建，也可以使用更方便简洁的工厂函数来创建：
```js
const {value, unit} = CSS.number('10');
// value === 10, unit === 'number'

const {value, unit} = CSS.px(42);
// value === 42, unit === 'px'

const {value, unit} = CSS.vw('100');
// value === 100, unit === 'vw'

const {value, unit} = CSS.percent('10');
// value === 10, unit === 'percent'

const {value, unit} = CSS.deg(45);
// value === 45, unit === 'deg'

const {value, unit} = CSS.ms(300);
// value === 300, unit === 'ms'
```

##### Math values
复杂的包含两个及两个以上数字的表达式则需要使用 `CSSMathValue` 来表示。
同样的，可以使用构造函数的调用来生成 `CSSMathValue`。
```js
new CSSMathSum(CSS.vw(100), CSS.px(-10)).toString(); // "calc(100vw + -10px)"

new CSSMathNegate(CSS.px(42)).toString() // "calc(-42px)"

new CSSMathInvert(CSS.s(10)).toString() // "calc(1 / 10s)"

new CSSMathProduct(CSS.deg(90), CSS.number(Math.PI/180)).toString();
// "calc(90deg * 0.0174533)"

new CSSMathMin(CSS.percent(80), CSS.px(12)).toString(); // "min(80%, 12px)"

new CSSMathMax(CSS.percent(80), CSS.px(12)).toString(); // "max(80%, 12px)"
```
还可以使用 `CSSNumericValue` 的链式调用。
```js
CSS.deg(45).mul(2) // {value: 90, unit: "deg"}

CSS.percent(50).max(CSS.vw(50)).toString() // "max(50%, 50vw)"

// Can Pass CSSUnitValue:
CSS.px(1).add(CSS.px(2)) // {value: 3, unit: "px"}

// multiple values:
CSS.s(1).sub(CSS.ms(200), CSS.ms(300)).toString() // "calc(1s + -200ms + -300ms)"

// or pass a `CSSMathSum`:
const sum = new CSSMathSum(CSS.percent(100), CSS.px(20)));
CSS.vw(100).add(sum).toString() // "calc(100vw + (100% + 20px))"
```
说到这个 `CSSNumericValue` 上的原型方法，还有两种比较常见的。
*转换*
```js
// Convert px to other absolute/physical lengths.
el.attributeStyleMap.set('width', '500px');
const width = el.attributeStyleMap.get('width');
width.to('mm'); // CSSUnitValue {value: 132.29166666666669, unit: "mm"}
width.to('cm'); // CSSUnitValue {value: 13.229166666666668, unit: "cm"}
width.to('in'); // CSSUnitValue {value: 5.208333333333333, unit: "in"}

CSS.deg(200).to('rad').value // 3.49066...
CSS.s(2).to('ms').value // 2000
```
*比较*
```js
const width = CSS.px(200);
CSS.px(200).equals(width) // true

const rads = CSS.deg(180).to('rad');
CSS.deg(180).equals(rads.to('deg')) // true

CSS.deg(180).equals(rads) // false
```

###  Transform 值
Old CSSOM
```css
transform: rotateZ(45deg) scale(0.5) translate3d(10px,10px,10px);
```
New Typed CSSOM
```js
const transform =  new CSSTransformValue([
  new CSSRotate(CSS.deg(45)),
  new CSSScale(CSS.number(0.5), CSS.number(0.5)),
  new CSSTranslate(CSS.px(10), CSS.px(10), CSS.px(10))
]);
```

`CSSTransformValue` 有很多酷的特性：
```js
new CSSTranslate(CSS.px(10), CSS.px(10)).is2D // true
new CSSTranslate(CSS.px(10), CSS.px(10), CSS.px(10)).is2D // false
new CSSTranslate(CSS.px(10), CSS.px(10)).toMatrix() // DOMMatrix
```

一个简单的动画例子，相当于直接修改底层的值，效率更高：
```js
const rotate = new CSSRotate(0, 0, 1, CSS.deg(0));
const transform = new CSSTransformValue([rotate]);

const box = document.querySelector('#box');
box.attributeStyleMap.set('transform', transform);

(function draw() {
  requestAnimationFrame(draw);
  transform[0].angle.value += 5; // Update the transform's angle.
  // rotate.angle.value += 5; // Or, update the CSSRotate object directly.
  box.attributeStyleMap.set('transform', transform); // commit it.
})();
```
### CSS 自定义属性的值
CSS 中的 `var()` 对应 Typed OM 中的 `CSSVariableReferenceValue` ，“它的值”最终会被解析为 `CSSUnparsedValue`，因为它的值可以是任何类型。我们知道 `var()` 函数还有第二个参数，可以是任意值，所以，`CSSVariableReferenceValue` 也有第二个参数，它是一个前面提到过的 `CSSUnparsedValue`，仔细想想，是不是很合理。下面是一个例子。
```js
const foo = new CSSVariableReferenceValue('--foo');
// foo.variable === '--foo'

// Fallback values:
const padding = new CSSVariableReferenceValue(
    '--default-padding', new CSSUnparsedValue(['8px']));
// padding.variable === '--default-padding'
// padding.fallback instanceof CSSUnparsedValue === true
// padding.fallback[0] === '8px'
```
如果你想要获取一个自定义属性的值，可以这么做：
```html
<style>
  body {
    --foo: 10px;
  }
</style>
<script>
  const styles = document.querySelector('style');
  const foo = styles.sheet.cssRules[0].styleMap.get('--foo').trim();
  console.log(CSSNumericValue.parse(foo).value); // 10
</script>
```
### 表示位置的值
一些空格分隔的位置值可以用 `CSSPositionValue` 表示。
```js
const position = new CSSPositionValue(CSS.px(5), CSS.px(10));
el.attributeStyleMap.set('object-position', position);

console.log(position.x.value, position.y.value);
// → 5, 10
```
### 值的解析
解析一整条样式：
```js
const css = CSSStyleValue.parse(
    'transform', 'translate3d(10px,10px,0) scale(0.5)');
// → css instanceof CSSTransformValue === true
// → css.toString() === 'translate3d(10px, 10px, 0) scale(0.5)'
```

解析一个值：
```js
CSSNumericValue.parse('42.0px') // {value: 42, unit: 'px'}

// But it's easier to use the factory functions:
CSS.px(42.0) // '42px'
```
利用解析的机制，我们可以捕获 CSS 的错误：
```js
try {
  const css = CSSStyleValue.parse('transform', 'translate4d(bogus value)');
  // use css
} catch (err) {
  console.err(err);
}
```

以上
