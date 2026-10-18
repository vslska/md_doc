# Community_Plugins

> Здесь будут публиковаться плагины и испарления к ним.

---

## Table of Contents

Для создания Оглавеления можно использовать плагин [`Table of Contents`](github.com/hipstersmoothie/obsidian-plugin-toc)

### Подробные вложенные упорядоченные списки
Если вы хотите, чтобы в оглавлении использовался подсчет вложенных списков (например, 1.1, 1.2), добавьте следующий фрагмент CSS в Obsidian. Это повлияет на все упорядоченные списки в ваших заметках. ( СSS *исправлен*, плагин не обновлялся 3 года, стандарный css код по ссылке, ломает нумерацию, так как применяет на весь контейнер, так как селектор голый `ol`, а нужен `el-ol` ) 
 
```css
/* Нацеливаемся ТОЛЬКО на блоки списка, внутри которых есть ссылки-якоря (то есть наше оглавление) */
.markdown-rendered .el-ol:has(a[data-href^="#"]) ol,
.markdown-rendered .el-ol:has(a[data-href^="#"]) ul {
  counter-reset: item;
  list-style-type: none !important;
  padding-left: 20px !important;
}

.markdown-rendered .el-ol:has(a[data-href^="#"]) li {
  display: block !important;
}

.markdown-rendered .el-ol:has(a[data-href^="#"]) li::before {
  content: counters(item, ".") ". ";
  counter-increment: item;
  padding-right: 8px;
  
  /* Твой цветной кастом для цифр оглавления */
  color: #8bd5ca; 
  font-weight: bold;
  display: inline-block;
}
```