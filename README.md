<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>一封无法拒绝的请假申请</title>
    <style>
        body {
            background-color: #f0f2f5;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            overflow: hidden;
        }

        .container {
            background-color: white;
            padding: 40px;
            border-radius: 12px;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
            text-align: center;
            width: 90%;
            max-width: 500px;
        }

        .screen { display: none; }
        .screen.active { display: block; }

        h1 {
            font-size: 24px;
            color: #333;
            margin-bottom: 10px;
        }
        p {
            font-size: 16px;
            color: #666;
            margin-bottom: 30px;
        }

        .emoji-container {
            margin-bottom: 40px;
            height: 120px;
            position: relative;
        }
        .emoji-container img {
            width: 120px;
            height: 120px;
            position: absolute;
            top: 0;
            left: 50%;
            transform: translateX(-50%);
            transition: opacity 0.3s ease-in-out;
        }
        .emoji-container .hidden {
            opacity: 0;
        }

        .button-container {
            position: relative;
            height: 120px; /* 增加逃跑空间 */
            display: flex;
            justify-content: center;
            align-items: center;
        }

        .btn {
            border: none;
            border-radius: 8px;
            font-size: 18px;
            font-weight: bold;
            color: white;
            cursor: pointer;
            transition: box-shadow 0.2s ease;
        }
        /* 移除通用的transform，因为拒绝按钮的transform由JS精确控制 */
        .btn:hover {
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
        }

        .approve {
            background-color: #28a745;
            padding: 12px 28px;
            position: absolute; /* 也设为绝对定位，方便计算位置 */
        }

        .reject {
            background-color: #dc3545;
            padding: 12px 28px;
            position: absolute;
            /* JS会控制transform, top, left */
            transition: top 0.3s ease-out, left 0.3s ease-out, transform 0.3s ease;
        }

        .back {
            background-color: #007bff;
            padding: 12px 28px;
        }
    </style>
</head>
<body>

    <div class="container">
        <!-- 屏幕一：请假申请界面 -->
        <div id="request-screen" class="screen active">
            <h1>脆皮大学生在线向你发送了一封请假申请</h1>
            <p>希望能得到辅导员的批准</p>
            <div class="emoji-container">
                <img id="happy-emoji" src="https://www.emojiall.com/images/120/apple/1f604.png" alt="开心">
                <img id="sad-emoji" src="https://www.emojiall.com/images/120/apple/1f628.png" alt="惊恐" class="hidden">
            </div>
            <div class="button-container">
                <button id="approve-btn" class="btn approve">批准</button>
                <button id="reject-btn" class="btn reject">拒绝</button>
            </div>
        </div>

        <!-- 屏幕二：感谢界面 -->
        <div id="success-screen" class="screen">
            <h1>谢谢最亲爱的辅导员批假！</h1>
            <p>非常开心！祝辅导员天天快乐！</p>
            <div class="emoji-container">
                <img src="https://www.emojiall.com/images/120/apple/1f60d.png" alt="感谢">
            </div>
            <button class="btn back" onclick="location.reload()">返回</button>
        </div>
    </div>

    <script>
        const rejectBtn = document.getElementById('reject-btn');
        const approveBtn = document.getElementById('approve-btn');
        const buttonContainer = document.querySelector('.button-container');
        const requestScreen = document.getElementById('request-screen');
        const successScreen = document.getElementById('success-screen');
        const happyEmoji = document.getElementById('happy-emoji');
        const sadEmoji = document.getElementById('sad-emoji');

        let sadTimeout;
        let clickCount = 0;
        let currentScale = 1.0; // 用一个变量来存储当前的缩放比例

        // 初始化按钮位置
        function setInitialPositions() {
            const containerWidth = buttonContainer.offsetWidth;
            approveBtn.style.left = `${containerWidth / 2 - approveBtn.offsetWidth - 10}px`;
            rejectBtn.style.left = `${containerWidth / 2 + 10}px`;
        }
        window.onload = setInitialPositions;
        window.onresize = setInitialPositions; // 窗口大小变化时也重新计算

        // 智能移动按钮，避开批准按钮
        function moveButton() {
            const containerRect = buttonContainer.getBoundingClientRect();
            const approveRect = approveBtn.getBoundingClientRect();

            // 将批准按钮的坐标转换为相对于容器的坐标
            const approveLeft = approveRect.left - containerRect.left;
            const approveRight = approveLeft + approveRect.width;
            
            let newLeft, newTop;
            let overlaps = true;
            
            // 循环生成新位置，直到不与批准按钮重叠为止
            while(overlaps) {
                const btnWidth = rejectBtn.offsetWidth * currentScale;
                const btnHeight = rejectBtn.offsetHeight * currentScale;

                newTop = Math.random() * (containerRect.height - btnHeight);
                newLeft = Math.random() * (containerRect.width - btnWidth);
                
                // 检查是否重叠（简单矩形碰撞检测）
                if (newLeft + btnWidth < approveLeft || newLeft > approveRight) {
                    overlaps = false; // 水平方向不重叠，就OK
                } else {
                    // 如果水平重叠，再检查垂直方向，给它更多机会
                    if (newTop + btnHeight < approveBtn.offsetTop || newTop > approveBtn.offsetTop + approveBtn.offsetHeight){
                        overlaps = false;
                    }
                }
            }
            
            rejectBtn.style.top = `${newTop}px`;
            rejectBtn.style.left = `${newLeft}px`;
        }

        // 鼠标移入拒绝按钮
        rejectBtn.addEventListener('mouseover', () => {
            clearTimeout(sadTimeout);
            sadTimeout = setTimeout(() => {
                happyEmoji.classList.add('hidden');
                sadEmoji.classList.remove('hidden');
            }, 300);
            moveButton();
        });
        
        // 鼠标移出按钮容器
        buttonContainer.addEventListener('mouseleave', () => {
            clearTimeout(sadTimeout);
            happyEmoji.classList.remove('hidden');
            sadEmoji.classList.add('hidden');
        });

        // 点击拒绝按钮
        rejectBtn.addEventListener('click', () => {
            clickCount++;
            const scaleReduction = 0.1; // 每次缩小10%
            currentScale = Math.max(0.3, currentScale - scaleReduction); // 计算新缩放，最小为0.3
            
            // 直接应用新的缩放比例
            rejectBtn.style.transform = `scale(${currentScale})`;
            
            moveButton(); // 点击后也让它逃跑
        });

        // 点击批准按钮
        approveBtn.addEventListener('click', () => {
            requestScreen.classList.remove('active');
            successScreen.classList.add('active');
        });

    </script>
</body>
</html>

一、HTML (结构层) - 网页的骨架
HTML (HyperText Markup Language) 负责定义网页的内容和结构。

标签 (Tags): 像 <body>, <div>, <h1>, <p>, <img>, <button> 都是标签。它们告诉浏览器“这里是一个段落”、“这里是一张图片”等。

<div>: 一个通用的“容器”标签，用来组织和包裹其他元素，非常适合用于布局。在这个例子里，我们用它来创建了 .container, .screen, .emoji-container 等。

<button>: 创建一个可点击的按钮。

<img>: 插入一张图片，src 属性指定图片的地址。

属性 (Attributes): 比如 id, class, src, alt。它们为标签提供额外信息。

id: 为一个元素提供唯一的标识符，如 id="reject-btn"。JavaScript 可以通过 id 精确地找到这一个按钮。

class: 为一个或多个元素提供一个“类名”，如 class="btn"。CSS 可以通过类名统一设置样式，JS 也可以通过类名找到所有具有这个类名的元素。

onclick: 一个事件属性，可以直接在HTML里绑定一个JavaScript函数，比如 <button onclick="location.reload()">。

二、CSS (样式层) - 网页的皮肤
CSS (Cascading Style Sheets) 负责网页的美化和布局，决定了元素长什么样、在哪里。

选择器 (Selectors): 用来选中要添加样式的HTML元素。

标签选择器: body { ... } (选中所有<body>标签)

类选择器: .container { ... } (选中所有class="container"的元素)

ID选择器: #approve-btn { ... } (选中id="approve-btn"的那个唯一元素)

盒模型 (Box Model): 每个HTML元素都可以看作一个矩形的盒子，CSS通过 padding (内边距), border (边框), margin (外边距) 来控制这个盒子的大小和间距。

定位 (Positioning): 这是实现“按钮逃跑”的关键。

position: relative;: 相对定位。元素的位置相对于它正常应该在的位置进行调整。它的主要作用是为内部的绝对定位元素提供一个“坐标系”。

position: absolute;: 绝对定位。元素会脱离正常的文档流，它的位置相对于最近的、设置了 position (非 static) 的父元素进行定位。我们用它来让“拒绝”按钮可以自由地在 .button-container 内部移动。

弹性布局 (Flexbox): display: flex; 是一种现代的、强大的布局方式。我们用 justify-content: center; 和 align-items: center; 轻松地实现了容器内元素的水平和垂直居中。

过渡 (Transition): transition: all 0.3s ease; 是让网页动效变得平滑的关键。它告诉浏览器，当某个CSS属性（比如 top, left, transform, opacity）发生变化时，不要瞬间完成，而是在0.3秒内平滑地过渡。这使得按钮的移动、表情的淡入淡出都显得非常自然。

变换 (Transform): transform: scale(0.8); 用来对元素进行2D或3D变换，如缩放、旋转、移动等，而且它不影响布局（不会推开其他元素），性能也很好。我们用它来实现按钮的缩小效果。

三、JavaScript (行为层) - 网页的大脑
JavaScript (JS) 负责网页的交互逻辑，让网页“活”起来。

DOM 操作 (Document Object Model): 浏览器把HTML文档解析成一个树状结构（DOM树），JavaScript 可以通过这个模型来访问和修改网页的任何内容。

document.getElementById('reject-btn'): 通过ID获取一个元素。

document.querySelector('.container'): 通过CSS选择器获取第一个匹配的元素。

element.style.left = '100px': 直接修改元素的CSS样式。

element.classList.add('hidden') / remove('active'): 动态地添加或移除元素的 class，这是控制显示/隐藏和样式切换的最佳实践。

事件监听 (Event Listeners): 这是实现互动的核心。JS可以“监听”用户的各种操作。

rejectBtn.addEventListener('mouseover', callbackFunction): 当鼠标移入“拒绝”按钮时，执行 callbackFunction 函数。

rejectBtn.addEventListener('click', ...): 当点击“拒绝”按钮时执行。

window.onload = ...: 当整个页面（包括图片等资源）加载完毕时执行。

函数 (Functions): function moveButton() { ... } 将一堆相关的代码打包成一个可复用的块。这让代码更清晰、更易于维护。

变量 (Variables): let clickCount = 0; 用 let 或 const 来声明变量，用于存储数据，比如点击次数、当前的缩放比例等。

定时器 (Timers):

setTimeout(callback, 300): 安排一个任务（callback函数），在300毫秒之后执行一次。我们用它来实现延迟变脸。

clearTimeout(timerId): 取消一个之前用 setTimeout 设置的定时器，防止它被执行。

数学与逻辑 (Math & Logic):

Math.random(): 生成一个0到1之间的随机数，这是实现随机位置的关键。

Math.max(): 返回一组数中的最大值，我们用它来确保按钮缩放不会小于一个最小值。

if...else... 和 while 循环：控制程序的流程，比如用 while 循环来确保新生成的位置不与“批准”按钮重叠。

总而言之，这个请假条是一个绝佳的“麻雀虽小，五脏俱全”的例子，它完美地展示了 HTML 构建结构、CSS 美化布局、JavaScript 驱动交互 这三大前端基石是如何协同工作的。
