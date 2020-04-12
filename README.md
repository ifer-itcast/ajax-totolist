# 补充

## 底部按钮的切换效果

为了方便控制，高亮的操作放到了 li 上面，需要修改对应的 CSS，例如：

```html
<ul class="filters">
    <li class="selected">
        <a href="javascript:;">All</a>
    </li>
    <li>
        <a href="javascript:;">Active</a>
    </li>
    <li>
        <a href="javascript:;">Completed</a>
    </li>
</ul>
```

```css
.filters li:hover a {
    border-color: rgba(175, 47, 47, 0.1);
}
.filters li.selected a {
    border-color: rgba(175, 47, 47, 0.2);
}
```

```javascript
$('.filters li').on('click', function () {
    $(this).addClass('selected').siblings().removeClass('selected');
});
```

## 切换底部按钮时获取数据

```javascript
<ul class="filters">
    <li class="selected">
        <a href="#task">All</a>
    </li>
    <li>
        <a href="#active">Active</a>
    </li>
    <li>
        <a href="#completed">Completed</a>
    </li>
</ul>
```

```javascript
// 监听 hash 的变化，获取数据并渲染（渲染数据的操作在 getData 里面）
window.addEventListener('hashchange', function() {
    getData();
});
```

```javascript
function getData() {
    // 根据当前的 hash 去请求对应的接口
    let hash = location.hash.substring(1);
    $.ajax({
        url: `/todo/${hash}`,
        type: 'get',
        success: function (response) {
            // 渲染
            render(response);
        }
    });
}
```

```javascript
function render(tasks) {
    let html = template('taskTpl', {
        tasks
    });
    // 任务列表显示在页面中
    taskBox.html(html);
}
```

后端新增 `/active` 和 `/completed` 接口

```javascript
// 未完成
todoRouter.get('/active', async (req, res) => {
    const result = await Task.find({ completed: false });
    res.send(result);
});

// 已完成
todoRouter.get('/completed', async (req, res) => {
    const result = await Task.find({ completed: true });
    res.send(result);
});
```

## 页面刷新时的问题

页面没有数据，需要获取数据并渲染

```javascript
getData();
```

底部切换的状态没有保持

```javascript
// 刷新时根据 hash 高亮对应的 li
function keepBtnStatus() {
    let num = location.hash === '#active' ? 1 : (location.hash === '#completed' ? 2 : 0);
    $('.filters li').eq(num).addClass('selected').siblings().removeClass('selected');
}
keepBtnStatus();
```

## 直接输入 `http://localhost:3000/todo/` 时的问题

调用的是 todo 接口，返回的就是 /todo 对应的 HTML 页面（字符串），最后循环的是这堆字符串，模板中的 each 也能循环 'abc' 这种，可以测试下！

解决：当打开地址为 http://localhost:3000/todo 时，为了匹配 task 接口，需要进行如下操作

```javascript
if (!location.hash) {
    location.hash = '#task';
}
```

## 计算未完成的数量，从后端接口获取

```javascript
function calcCount() {
    $.ajax({
        url: '/todo/unCompletedTaskCount',
        success: function (res) {
            strong.text(res.num);
        }
    })
}
```

## 清除已完成的数据

```javascript
$('.clear-completed').on('click', function () {
    $.ajax({
        url: '/todo/clearTask',
        success: function () {
            getData();
        }
    });
});
```

## 其他变化

增加、删除、修改数据等的操作，只需 AJAX 请求成功后调用 `getData()` 即可！**所有数据状态的变化都应该依赖后端返回的结果，后端成功了才是真正的成功，前端才能进行数据的渲染！**


