# 补充

## 底部按钮的切换效果

为了方便控制，高亮的操作放到了 li 上面，需要修改对应的 HTML 和 CSS，例如：

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

给底部 a 标签加一个跳转的 Hash，这样在点击时会触发 `hashChange` 事件

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

需要在页面刷新时获取数据并渲染，不然数据会丢失

```javascript
getData();
```

底部按钮的状态没有保持，需要根据 hash 高亮对应的 li

```javascript
function keepBtnStatus() {
    let num = location.hash === '#active' ? 1 : (location.hash === '#completed' ? 2 : 0);
    $('.filters li').eq(num).addClass('selected').siblings().removeClass('selected');
}
keepBtnStatus();
```

## 直接输入 `http://localhost:3000/todo/` 时的奇怪问题

调用的是 /todo 接口，返回的就是 /todo 对应的 HTML 页面（字符串），最后循环的是这堆字符串（模板中的 each 也能循环 'abc' 这种，可以测试下）

解决：当打开地址为 http://localhost:3000/todo 时，就获取全部数据，即匹配 task 接口即可，需要进行如下操作

```javascript
if (!location.hash) {
    location.hash = '#task';
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

## 一个原则

**所有数据状态的变化都应该依赖后端返回的结果，后端操作成功了才是真正的成功，前端才能进行数据的渲染！这样即便是多个用户对同一个列表进行操作时也能保证前端数据展示的正确性！**

```javascript
// 例如计算未完成的列表数量，需要从后端获取才是正确的姿势！
function calcCount() {
    $.ajax({
        url: '/todo/unCompletedTaskCount',
        success: function (res) {
            strong.text(res.num);
        }
    })
}
```

其他的增加、删除、修改数据等的操作，只需 AJAX 请求成功后调用 `getData()` 即可，前端无需依赖全局变量 `taskAry` 进行操作！`getData()` 请求数据成功后要调用 `calcCount()` 重新计算下未完成数量，但并不是所有获取数据的操作都需要调用 `calcCount()`（例如修改标题），可以通过传递参数的方式解决，例如 `getDate(true)` 代表需要重新获取，false 不获取即可！

## 思考，如何优化代码？

```javascript
const todoList = (function () {
	return {
		// 入口函数
		init: function () {
			this.bindEvent();
		},
		// 绑定事件
		bindEvent: function () {
			taskIpt.on('keyup', this.addTodo);
			taskIpt.on('click', '.destroy', this.deleteTodo);
			taskIpt.on('blur', '.editing', this.editTodo);
		},
		// 增
		addTodo: function () {},
		// 删
		deleteTodo: function () {},
		// 改
		editTodo: function () {}
	};
})();

todoList.init();
```

