# 关于
::: tip
This is a tip
:::
::: warning
This is a warning
:::
::: danger
This is a dangerous tip
:::
``` js{2}
export default {
  name: 'MyComponent',
  // ...
}
```
``` html
<ul>
  <li
    v-for="todo in todos"
    :key="todo.id"
  >
    {{ todo.text }}
  </li>
</ul>
```
<test></test>
### Badge <Badge text="beta" type="warn"/> <Badge text="0.10.1+" type="danger"/> <Badge text="默认主题"/>