# 补充

**数据状态变化都应该依赖后端返回的结果，而不是前端操作！**

## 底部按钮的数据切换

### HTML 更改

为了方便把控制高亮的操作放到了 li 上面，需要改变对应的 CSS

```html
<ul class="filters">
    <li class="selected" data-status="task">
        <a href="#/task">All</a>
    </li>
    <li data-status="active">
        <a href="#/active">Active</a>
    </li>
    <li data-status="completed">
        <a href="#/completed">Completed</a>
    </li>
</ul>
```

### JS 更改

```javascript
$('.filters li').on('click', function () {
    // 点击时会导致 hash 变化
    $(this).addClass('selected').siblings().removeClass('selected');
});
```

```javascript
// hash 变化的时候获取数据并渲染（渲染数据的操作在 getData 里面）
window.addEventListener('hashchange', function() {
    getData();
});
```

```javascript
function getData() {
    // 根据当前的 hash 去请求对应的接口
    let hash = location.hash.substring(2);
    $.ajax({
        url: `/todo/${hash}`,
        type: 'get',
        success: function (response) {
            // 渲染
            render(response);
            // 计算数量
            calcUncompletedCount();
        }
    });
}
```

当打开地址为 http://localhost:3000/ 时，为了进行上面的接口匹配需要进行如下操作

```javascript
if (!location.hash) {
    location.hash = '#/task';
    // location.href = '#/task';
}
```

```javascript
// 不需要依赖全局的 taskAry，数据应该是从后端获取的，这样才能真正的保持数据同步
function render(tasks) {
    let html = template('taskTpl', {
        tasks
    });
    // 任务列表显示在页面中
    taskBox.html(html);
}
```

```javascript
// 计算数量的操作，后端获取
function calcUncompletedCount() {
    $.ajax({
        url: '/todo/unCompletedTaskCount',
        success: function (res) {
            strong.text(res.num);
        }
    })
}
```

### 后端代码更改

新增 `/active` 和 `/completed` 接口

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

## 页面加载/刷新时的处理

```javascript
// 获取数据
getData();
```

```javascript
// 刷新时根据 hash 高亮对应的 li
function keepBtnStatus() {
    let num = location.hash === '#/active' ? 1 : (location.hash === '#/completed' ? 2 : 0);
    $('.filters li').eq(num).addClass('selected').siblings().removeClass('selected');
}
keepBtnStatus();
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

## 其他

增加、删除、修改数据等的操作，只需 AJAX 请求成功后调用 `getData()` 即可！
