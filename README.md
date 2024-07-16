# Navon实验教程（Jspsych代码示例）

## 实验简介

Navon实验是一种广泛应用于认知心理学研究的实验范式，旨在研究人类的注意力和知觉处理方式。

此范式是多个较小的局部刺激组成一个较大的复合的整体刺激，局部刺激与整体刺激存在一致、不一致或者中性关系，其中一致性条件是构成整体的局部刺激与整体所表示的刺激一致，如H组成H，不一致性条件是构成整体的局部刺激与整体所表示的刺激不一致，如L组成H，中性条件是指由方块等组成的局部刺激与整体不构成冲突。本实验中整体刺激为3°47×3°12’(长×宽)，即33mm×28mm，局部刺激为整体刺激的1/8。本实验共有两种情况，一致条件(TT、LL，前为局部刺激，后为整体刺激，下同)和不一致条件(TL、LT)。刺激示例如下图所示。

<img src="D:\SCNU\Second semester of the first year of graduate school\用户体验与消费心理研究——JSpshch\Navon-3\刺激示例" alt="d0d1c36378aa665b945a40990b20eba" style="zoom: 50%;" />



## 代码解释（index.html）

1. 代码声明部分。

   声明文档类型为HTML，设置文档的语言为英语。设置视口，确保页面在不同设备上正确显示，并设置网页标题为 "Navon Experiment"。引入外部 CSS 文件，用于设置页面样式。

   引入 JsPsych 库及其插件，包括用于创建和管理心理学实验的 JsPsych 库的核心文件；允许显示 HTML 内容并记录键盘响应的插件；允许显示图片并记录键盘响应的插件；允许创建 HTML 表单并记录用户输入的插件和允许实验在全屏模式下运行的插件。并且引入可以将实验数据保存为 Excel 文件格式的库。

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Navon Experiment</title>
    <link rel="stylesheet" href="style.css">
    <script src="https://cdn.jsdelivr.net/npm/jspsych@6.3.1/jspsych.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/jspsych@6.3.1/plugins/jspsych-html-keyboard-response.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/jspsych@6.3.1/plugins/jspsych-image-keyboard-response.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/jspsych@6.3.1/plugins/jspsych-survey-html-form.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/jspsych@6.3.1/plugins/jspsych-fullscreen.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.17.0/xlsx.full.min.js"></script>
</head>
<body>
    <div id="jspsych-target"></div>
</body>
</html>
```

2. 获取被试的基本信息，即姓名、年龄以及优势手，并将这些信息添加到实验数据中。其中使用`'survey-html-form'`插件，显示 HTML 表单并记录用户输入。

```javascript
const participant_info = {
    type: 'survey-html-form',
    preamble: '<h2>请输入您的信息</h2>',
    html: `
        <p>您的姓名是：<input name="name" type="text" required></p>
        <p>您的年龄是：<input name="age" type="number" required></p>
        <p>您的优势手是：<input name="hand" type="text" required></p>
    `,
    button_label: '继续',
    on_finish: function(data){
        console.log("Raw responses: ", data.responses); // 输出原始 responses 以便调试
        if (data.responses) {
            try {
                var responses = JSON.parse(data.responses);
                console.log("Parsed responses: ", responses); // 输出解析后的 responses 以便调试
                jsPsych.data.addProperties({
                    name: responses.name,
                    age: responses.age,
                    hand: responses.hand
                });
            } catch (error) {
                console.error("Error parsing responses: ", error);
            }
        } else {
            console.error("No responses received.");
        }
    }
};
```

3. 设置全屏，使实验全屏显示

```javascript
const fullscreen = {
    type: 'fullscreen',
    fullscreen_mode: true
};
```

4. 显示指导语的图片并等待键盘响应，被试按下空格键继续

```javascript
const instructions = {
    type: 'image-keyboard-response',
    stimulus: '总指导语.png',
    choices: [' ']
};
```

5. 加载所有实验刺激及其对应的正确反应键的列表

```javascript
const stimuli = [
    { stimulus: 'L-L.png', correct_response: 'j' },
    { stimulus: 'L-T.png', correct_response: 'f' },
    { stimulus: 'T-L.png', correct_response: 'j' },
    { stimulus: 'T-T.png', correct_response: 'f' }
];
```

6. 生成试次，每个刺激呈现两次

```javascript
function generate_trials(part) {
    let trials = [];
    for (let i = 0; i < 2; i++) {
        stimuli.forEach(stimulus => {
            trials.push({
                type: 'image-keyboard-response',
                stimulus: stimulus.stimulus,
                choices: ['f', 'j'],
                trial_duration: 3000,
                data: { correct_response: stimulus.correct_response, part: part },
                on_finish: function(data) {
                    data.correct = data.response === data.correct_response ? 1 : 0;
                }
            });
        });
    }
    return trials;
}
```

7. 设置休息屏，持续三十秒或者被试按空格键继续。

```javascript
const rest_screen = {
    type: 'html-keyboard-response',
    stimulus: '<p>该部分结束，请休息30s。或者按空格键继续</p>',
    choices: [' '],
    trial_duration: 30000
};
```

8. 结束屏持续时间3s。

```javascript
const end_screen = {
    type: 'image-keyboard-response',
    stimulus: 'End.png',
    choices: jsPsych.NO_KEYS,
    trial_duration: 3000
};
```

9. 计算并显示正确率

```javascript
const show_accuracy = {
    type: 'html-keyboard-response',
    stimulus: function() {
        let correct_trials = jsPsych.data.get().filter({correct: 1}).count();
        let total_trials = jsPsych.data.get().filter({correct: 1}).count() + jsPsych.data.get().filter({correct: 0}).count();
        let accuracy = Math.round((correct_trials / total_trials) * 100);
        return `<p>您的正确率为${accuracy}%</p>`;
    },
    choices: jsPsych.NO_KEYS,
    trial_duration: 3000
};
```

10. 保存结果为Excel文件

```javascript
function save_data() {
    let data = jsPsych.data.get().json();
    let blob = new Blob([data], { type: 'application/json' });
    let url = URL.createObjectURL(blob);
    let a = document.createElement('a');
    a.href = url;
    a.download = 'results.json';
    a.click();
}
```

## 代码解释（style.css）

1. 使用弹性盒子来对齐全页面内容。将 `body` 和 `html` 元素设置为弹性盒子容器。页面内容将会在浏览器窗口的中心方向和垂直方向上居中对齐。移除默认的外边距和内边距，以确保页面内容从最边缘开始。将 `body` 和 `html` 元素的宽度和高度设置为 100%，以覆盖整个浏览器窗口。

```css
body, html {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  font-family: Arial, sans-serif;
}
```

2. JsPsych 目标容器样式，使用 Flexbox，在水平和垂直方向上居中

```css
#jspsych-target {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}
```

3. 表单样式，使用 Flexbox布局，设置主轴为垂直方向，使子元素垂直排列，使表单中的元素在水平方向上居中。

```css
form {
  display: flex;
  flex-direction: column;
  align-items: center;
}
```

4. 表单段落样式。选择 `form` 元素内的所有 `p` 标签（段落），设置这些段落上下外边距为 10px，确保段落之间有适当的间距。

```css
form p {
  margin: 10px 0;
}
```

5. 标题样式。选择所有 `h2` 标签（标题）。将这些标题文本内容居中对齐，使标题在页面上居中显示。

```css
h2 {
  text-align: center;
}
```

6. 图片样式。设置图片最大宽度为父容器的 100%，防止图片超出容器，使图片高度自动调整，保持原始比例，将图片显示为块级元素，使图片水平居中。确保页面在不同设备和屏幕尺寸上都能以良好的布局和样式显示。

```css
img {
  max-width: 100%;
  height: auto;
  display: block;
  margin: auto;
}
```
