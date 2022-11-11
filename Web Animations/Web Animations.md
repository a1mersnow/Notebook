WAAPI 让我们能够构建动画并控制动画的播放。

# 认识 Web Animations API
WAAPI 向开发者开放了浏览器的动画引擎，开发者可以通过 Javascript 接口来操作该引擎。该 API 是基于 CSS 动画和 CSS 过渡实现的，并且考量了未来可能会增加的动画效果。它是 web 平台支持的实现动画的最高效的方式之一，在该方式下浏览器能够进行内部优化，因而不再需要所谓的黑科技、强制性的技巧和 requestAnimationFrame 方法。

通过 WAAPI，我们可以很大程度上将交互动画从样式转移到 JS 上，将行为与表现分离。我们不再需要依赖很多的 DOM 技巧来控制动画的播放，比如直接赋值 CSS 属性或者切换类名。并且与简单、声明式的 CSS 不同，JS 允许我们动态地改变样式属性或播放选项。对需要构建自定义动画库和开发动画接口的开发者来说，WAAPI 可能是最酷的工具了。让我们来看看它都可以做什么。  

# 浏览器支持
不多说，直接上 can i use。
http://caniuse.com/#search=web%20animations%20api

# 用 WAAPI 来写 CSS 动画
对大多数 Web 开发者来说，CSS 动画是我们很熟悉的东西，因此学习 WAAPI 最好的方式莫过于从 CSS 动画开始。CSS 动画有着与 WAAPI 相似的语法，因此很适合作为我们讲解的开始。

### CSS 版本
这是一个翻跟头（wut?）的动画，用 CSS 编写的，表现了爱丽丝坠入兔子洞从而进入仙境的场景（要看完整的代码请点击[codepen](http://codepen.io/rachelnabors/pen/QyOqqW)）：

![爱丽丝梦游仙境](https://mdn.mozillademos.org/files/13843/tumbling-alice_optimized.gif)

我们观察到背景的移动，爱丽丝的旋转以及她身体颜色随着旋转产生的变化。我们这里只关注爱丽丝自身旋转的动画。这是一份简单的 CSS 实现：

```css
#alice {
  animation: aliceTumbling infinite 3s linear;
}

@keyframes aliceTumbling {
  0% {
    color: #000;
    transform: rotate(0) translate3D(-50%, -50%, 0);
  }
  30% {
    color: #431236;
  }
  100% {
    color: #000;
    transform: rotate(360deg) translate3D(-50%, -50%, 0);
  }
}
```

这段代码的效果就是让爱丽丝身体的颜色和她自身的旋转以恒定的速率每3s完成一个循环。在 @keyframes 代码中我们可以看到，在动画进行到 30% 的时候，爱丽丝身体的颜色从黑色变成了深紫红色，随后 70% 的时间里，有逐渐变回黑色。

### JS 版本
现在让我们尝试一下，通过 WAAPI 来完成相同的动画。

_刻画关键帧_
首先我们需要创建与 CSS 关键帧一致的关键帧数组：

```js
var aliceTumbling = [
  { transform: 'rotate(0) translate3D(-50%, -50%, 0)', color: '#000' },
  { color: '#431236', offset: 0.333},
  { transform: 'rotate(360deg) translate3D(-50%, -50%, 0)', color: '#000' }
];
```

这里我们使用了一个包含很多关键帧对象的数组。与 CSS 不同的是，WAAPI 不需要你通过百分比去指定每一帧发生的时间点，WAAPI 会自动将动画按照你定义的关键帧数量等分。这意味着一个关键帧数组如果包含了三个关键帧，中间的关键帧将呈现在整个动画过程的 50% 处。

当我们想明确地设置某一帧的呈现时间时，可以直接在这一帧的对象中指定一个 `offset` 属性。在上面的例子中，为了确保爱丽丝身体的颜色在 30% 而不是 50% 的位置变成深紫红色，我们将这一帧的 `offset` 设置为 0.3。

显然，每一个动画过程必须包含两个或两个以上的关键帧，如果只定义了一个关键帧，WAAPI 会抛出一个 `NotSupportedError` 异常。

总结一下，除非你设置 `offset` 属性，否则动画过程会按照帧数等分。

_配置时序属性_
这里我们需要参照上面的 CSS 动画创建一个包含时序属性的对象（可以看作是 AnimationEffectTimingProperties 的一个实例）
```js
var aliceTiming = {
  duration: 3000,
  iterations: Infinity
}
```
你可能会注意到，这里与 CSS 动画时序等价的 JS 时序表达有些许的不同：
- `duration` 是以 mm(毫秒) 为单位的。
- `iterations` 对应于 CSS 中的 `iteration-count`

PS：其实 CSS 动画和 WAAPI 之间还有很多小的差异。比如说，WAAPI 不使用 'infinite' 字符串，而是使用 JS 中的关键字 `Infinity`。不使用 `animation-timing-function` 而是使用 `easing`。CSS 动画中 `animation-timing-function` 的默认值是 `ease`，而在 WAAPI 中，默认值是 `linear`。

_组合一下_
现在是时候把这两个配置用 `Element.animate()` 方法组合起来了：
```js
document.getElementById("alice").animate(
  aliceTumbling,
  aliceTiming
)
```

动画效果在[这里](http://codepen.io/rachelnabors/pen/rxpmJL)

`animate()` 方法可以在任何的 DOM 元素上调用，CSS 能完成的 WAAPI 都可以完成。

还有，如果我们只想指定 `duration` 属性，可以直接传一个数字值：
```js
document.getElementById("alice").animate(
  [
    { transform: 'rotate(0) translate3D(-50%, -50%, 0)', color: '#000' },
    { color: '#431236', offset: 0.333},
    { transform: 'rotate(360deg) translate3D(-50%, -50%, 0)', color: '#000' }
  ], 3000);
```

# 使用 play(), pause(), reverse() 和 playbackRate 控制播放
现在我们已经能用 WAAPI 写 CSS 动画了，不过，WAAPI 真正强大的地方在于，控制动画的播放。WAAPI 为此提供了几个非常有用的方法。让我们通过下面的爱丽丝变大变小动画来看一看如何暂停和播放动画（要看完整的代码请点击[codepen](http://codepen.io/rachelnabors/pen/PNYGZQ)）：

![爱丽丝梦游仙境](https://mdn.mozillademos.org/files/13845/growing-shrinking_article_optimized.gif)

在这个游戏中，我们通过瓶子里的药水使爱丽丝变小，通过蛋糕使爱丽丝变大。

### 暂停和播放动画
我们过会儿再讨论爱丽丝的动画，现在我们先来看蛋糕的动画：
```js
var nommingCake = document.getElementById('eat-me_sprite').animate(
[
  { transform: 'translateY(0)' },
  { transform: 'translateY(-80%)' }
], {
  fill: 'forwards',
  easing: 'steps(4, end)',
  duration: aliceChange.effect.timing.duration / 2
});
```
`Element.animate()` 方法调用后将立即运行。为了给玩家一个点击蛋糕的机会，而不是让蛋糕自己消失掉，我们在 `Element.animate()` 方法之后立即调用 `Animation.pause()` 方法：
```js
nommingCake.pause();
```

之后我们可以用 `Animation.play()` 方法在适当的时机播放动画：
```js
nommingCake.play();
```
这里我们想把蛋糕的动画跟爱丽丝的动画联系起来，爱丽丝吃了蛋糕后变大。可以通过如下函数实现：
```js
var growAlice = function() {

  // Play Alice's animation.
  aliceChange.play();

  // Play the cake's animation.
  nommingCake.play();

}
```
当用户在蛋糕上按下鼠标或者用手指按蛋糕时，我们就可以调用上面的 `growAlice` 函数来播放动画：
```js
cake.addEventListener("mousedown", growAlice, false);
cake.addEventListener("touchstart", growAlice, false);
```

PS：当然，你得在用户抬起鼠标或手指时，暂停播放动画，不然...

### 其他有用的方法
除了暂停和播放，我们还可以用以下的方法：
- `Animation.finish()` 跳过动画过程，直接跳到动画的结束位置。
- `Animation.cancel()` 取消动画过程，直接跳到动画的开始位置。
- `Animation.reverse()` 设置动画的 `playbackRate` 属性为负值，即回放动画。

让我们先看一下 `playbackRate` 这个属性，一个负的 `playbackRate` 属性会导致动画以相反的方向播放。当爱丽丝喝了药水，她就变小了。这是因为喝下药水的函数将 `playbackRate` 从 1 变成 -1：
```js
var shrinkAlice = function() {
  aliceChange.playbackRate = -1;
  aliceChange.play();
}

bottle.addEventListener("mousedown", shrinkAlice, false);
bottle.addEventListener("touchstart", shrinkAlice, false);
```

在 `镜中缘` 情节中，爱丽丝来到了另一个世界，在这个世界中，她必须奔跑才能保持在原地，奔跑得更快才能前进。在下面这个 “跟红皇后比赛” 的例子中，爱丽丝和红皇后奔跑以使自己保持在原地（要看完整的代码请点击[codepen](http://codepen.io/rachelnabors/pen/PNGGaV)）：

![爱丽丝梦游仙境](https://mdn.mozillademos.org/files/13847/red-queen-race_optimized.gif)

因为身体变小很容易累，爱丽丝会不断地慢下来。我们可以通过如下代码减小动画的 `playbackRate` 属性：

```js
setInterval( function() {

  if (redQueen_alice.playbackRate > .4) {
    redQueen_alice.playbackRate *= .9;
  }

}, 3000);
```

但是我们要允许用户通过点击使动画加速，这可以通过扩大动画的 `playbackRate` 属性来完成：
```js
var goFaster = function() {

  redQueen_alice.playbackRate *= 1.1;

}

document.addEventListener("click", goFaster);
document.addEventListener("touchstart", goFaster);
```
当你点击的时候，背景中的物体也会受到影响。当你让爱丽丝和红皇后的速度快一些时会发生什么呢？慢一些时呢？

# 获取动画的信息
想想我们还可以如何利用 `playbackRate` 属性。比如通过允许放慢整个网站的动画速率来提高前庭功能障碍患者的可访问性。通过 CSS 来完成这项功能是不可能的，除非重新计算每个 CSS 动画规则中的 `duration` 属性。而通过 WAAPI，我们可以用即将到来的 `document.getAnimations()` 方法遍历页面中的每个动画并调整它们的 `playbackRate` 属性：
```js
document.getAnimations().forEach(
  function (animation) {
    animation.playbackRate *= .5;
  }
);
```
瞧，使用 WAAPI 的话，你需要做的全部就是改变一个数值而已。

另一个 CSS 动画难以独自完成的事情是，创建依赖于另一个动画的动画。举个例子，在爱丽丝变大变小的代码中，你可能注意到了这个奇怪的 `duration` 属性：
```js
duration: aliceChange.effect.timing.duration / 2
```
为了理解这样写的原因，让我们看看爱丽丝动画的代码：
```js
var aliceChange = document.getElementById('alice').animate(
  [
    { transform: 'translate(-50%, -50%) scale(.5)' },
    { transform: 'translate(-50%, -50%) scale(2)' }
  ], {
    duration: 8000,
    easing: 'ease-in-out',
    fill: 'both'
  });
```
爱丽丝的动画是从原来身体大小的一半变成原来身体大小的两倍。我们先在动画开始的地方暂停：
```js
aliceChange.pause();
```

现在她只有原来身体一半的大小，看起来就像已经喝下了整瓶的药水。我们想要的是，一开始身体处在正常的大小，这就需要动画过程已经进行到50%的位置。我们可以通过设置 `Animation.currentTime` 为 4000 来解决这个问题：
```js
aliceChange.currentTime = 4000;
```

在制作整个动画的过程中，我们可能需要多次修改这个值以达到最好的效果。如果能自动地设置这个值就好了，这样我们就不需要每次手动编辑了。实际上我们可以通过引用 `Animation.effect` 属性来完成这件事，它返回一个包含当前动画所有细节的对象：
```js
aliceChange.currentTime = aliceChange.effect.timing.duration / 2;
```

通过 `effect` 属性，我们可以获取动画的关键帧对象和时序对象。`aliceChange.effect.timing` 指向爱丽丝动画的时序对象（`AnimationEffectTimingReadOnly` 的实例）—— 这个时序对象中包含了 `duration` 属性。我们可以将这个 `duration` 属性除以 2 之后赋值给 `aliceChange.currentTime`，使爱丽丝一开始是正常大小。

我们也可以用同样的方式设置蛋糕和药水动画的 `duration` 属性：
```js
var drinking = document.getElementById('liquid').animate(
[
  { height: '100%' },
  { height: '0' }
], {
  fill: 'forwards',
  duration: aliceChange.effect.timing.duration / 2
});
drinking.pause();
```

现在三个动画都链接到同一个 `duration`，我们每次只需要修改一处就可以了。

我们还可以用 WAAPI 计算出动画的当前进度。当爱丽丝把蛋糕吃完或者把药水喝完，游戏就结束了。玩家看到的画面应该取决于爱丽丝动画的进度，不管是她变得太大而不能通过小门还是变得太小而不能拿到桌子上的钥匙。我们可以通过用 `aliceChange.effect.activeDuration` 除 `Animation.currentTime` 从而计算出她处于什么状态，是最大、最小还是中间。
```js
var endGame = function() {

  // get Alice's timeline's playhead location
  var alicePlayhead = aliceChange.currentTime;
  var aliceTimeline = aliceChange.effect.activeDuration;

  // stops Alice's and other animations
  stopPlayingAlice();

  // depending on which third it falls into
  var aliceHeight = alicePlayhead/aliceTimeline;

  if (aliceHeight <= .333){
    // Alice got smaller!
    ...

  } else if (aliceHeight >= .666) {
    // Alice got bigger!
    ...

  } else {
    // Alice didn't change significantly
    ...

  }
}
```

# Callbacks and promises
CSS 动画和 CSS 过渡都有它们自己的事件监听，同样地，WAAPI 也有：
- `onfinish` 用于注册完成事件，并且可以通过 `finish()` 方法手动触发
- `oncancel` 用于注册取消事件，并且可以通过 `cancel()` 方法手动触发

这里我们为蛋糕、药水和爱丽丝注册完成事件，事件处理器指定为 `endGame` 函数：

```js
// When the cake or runs out...
nommingCake.onfinish = endGame;
drinking.onfinish = endGame;

// ...or Alice reaches the end of her animation
aliceChange.onfinish = endGame;
```
未来会针对这两个事件增加对 promise 的支持。

# 结论
以上介绍的是 WAAPI 的基本特性，大多数都已经被现代浏览器所支持。现在你应该已经准备好了在浏览器中进行一番探索，创造出你自己的动画！

# 更新 2018-12-11
新版API有了一些变化：

| old | new |
| ------ | ------ |
| effect.timing.duration | effect.getTiming().duration |
| effect.activeDuration | effect.getComputedTiming().activeDuration |
| - | Animation |
| - | KeyframeEffect |
| - | document.timeline |

注：原来的 `el.animate` 方法继续保留

```js
const animation = new Animation(
  new KeyframeEffect(
    document.querySelector('#a'),
    [
      {
        transform: 'translateX(0)'
      },
      {
        transform: 'translateX(500px)'
      },
      {
        transform: 'translateY(500px)'
      }
    ],
    {
      delay: 3000,
      duration: 2000,
      iterations: 3
    }
  ),
  document.timeline
);

animation.play();
```
