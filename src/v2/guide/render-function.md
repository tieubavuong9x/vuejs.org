---
title: Các hàm render & JSX
type: guide
order: 303
---

## Cơ bản

Trong đại đa số các trường hợp, Vue khuyến khích sử dụng template để xây dựng HTML. Tuy nhiêu có một số trường hợp bạn cần dùng đến sức mạnh của JavaScript. Những lúc này bạn có thể dùng **hàm render** (render function), một giải pháp gần hơn với trình biên dịch, để thay thế cho template.

Chúng ta hãy xem một ví dụ đơn giản trong đó một hàm `render` trở nên hữu ích. Chẳng hạn, bạn muốn tạo các tiêu đề `h1`, `h2`, `h3`… chứa liên kết, như sau:

``` html
<h1>
  <a name="hello-world" href="#hello-world">
    Hello world!
  </a>
</h1>
```

Từ đoạn HTML trên, bạn quyết định tạo một giao diện component như sau:

``` html
<anchored-heading :level="1">Hello world!</anchored-heading>
```

Khi bắt đầu với một component chỉ tạo thẻ tiêu đề dựa trên prop `level` được truyền vào, bạn sẽ nhanh chóng đi đến một kết quả trông như thế này:

``` html
<script type="text/x-template" id="anchored-heading-template">
  <h1 v-if="level === 1">
    <slot></slot>
  </h1>
  <h2 v-else-if="level === 2">
    <slot></slot>
  </h2>
  <h3 v-else-if="level === 3">
    <slot></slot>
  </h3>
  <h4 v-else-if="level === 4">
    <slot></slot>
  </h4>
  <h5 v-else-if="level === 5">
    <slot></slot>
  </h5>
  <h6 v-else-if="level === 6">
    <slot></slot>
  </h6>
</script>
```

``` js
Vue.component('anchored-heading', {
  template: '#anchored-heading-template',
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```

Rõ ràng đây không phải là một template tốt. Chẳng những nó quá rườm rà, mà ở đây chúng ta còn lặp lại `<slot></slot>` cho mỗi level, và lại phải thực hiện một quá trình tương tự khi thêm phần tử `<a>`. Vì thế, hay thử viết lại với một hàm `render`:

``` js
Vue.component('anchored-heading', {
  render: function (createElement) {
    return createElement(
      'h' + this.level,   // tên thẻ
      this.$slots.default // mảng các phần tử con
    )
  },
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```

Đơn giản hơn nhiều! Đại để thế. Code trở nên ngắn hơn, nhưng cũng đòi hỏi bạn phải quen thuộc hơn với các thuộc tính của đối tượng Vue. Trong trường hợp này, bạn phải biết rằng khi truyền các phần tử con không có thuộc tính `slot` vào trong một component, ví dụ `Hello world!` trong `anchored-heading`, các phần tử con này được chứa trong đối tượng component tại `$slots.default`. Chúng tôi khuyên bạn **nên đọc về [các API của các thuộc tính của đối tượng Vue](../api/#Thuoc-tinh-cua-doi-tuong) trước khi đi sâu nghiên cứu về các hàm render**.

## Node, tree, và virtual DOM

Trước khi đi sâu vào các hàm render, ít nhiều kiến thức về cách hoạt động của trình duyệt là rất quan trọng. Ví dụ chúng ta có đoạn HTML sau:

```html
<div>
  <h1>Bài đồng dao</h1>
  Năm mới năm me
  <!-- TODO: Hoàn thành bài đồng dao -->
</div>
```

Khi đọc đoạn code này, trình duyệt sẽ xây dựng cấu trúc dạng cây (DOM tree) bao gồm các ["DOM node"](https://javascript.info/dom-nodes) để giúp quản lí mọi thứ, tương tự như việc bạn xây dựng một cây gia phả để giữ thông tin về mọi người trong dòng họ vậy.

Cấu trúc cây của đoạn HTML trên sẽ giống như sau:

![Sơ đồ DOM tree](/images/dom-tree.png)

Mỗi phần tử trên DOM tree là một node. Mỗi text là một node. Ngay cả comment cũng là node! Một node đơn giản chỉ là một "mảnh" trên trang web. Và cũng tương tự như trong một cây gia phả, mỗi node có thể có các node con (nghĩa là một mảnh có thể chứa các mảnh khác).

Cập nhật tất cả các node này một cách hiệu quả có thể là một việc khó khăn, nhưng may thay, bạn không bao giờ phải làm việc này một cách thủ công. Thay vào đó, chỉ cần báo cho Vue biết bạn muốn có HTML gì trên trang, trong một template:

```html
<h1>{{ blogTitle }}</h1>
```

hoặc trong một hàm render:

``` js
render: function (createElement) {
  return createElement('h1', this.blogTitle)
}
```

Và trong cả hai trường hợp, Vue sẽ tự động giữ cho trang web được cập nhật, ngay cả khi `blogTitle` thay đổi.

### Virtual DOM

Vue làm được điều này nhờ vào việc xây dựng một **virtual DOM** (DOM ảo) để theo dõi tất cả những thay đổi cần thực hiện đối với DOM thật. Bạn hãy nhìn kĩ dòng này:

``` js
return createElement('h1', this.blogTitle)
```

Lệnh `createElement` ở đây thực chất là đang trả về cái gì? _Không hẳn_ là một element (phần tử) DOM thật sự. Nếu nói cho đúng, chúng ta có thể đặt lại tên cho hàm này một cách chính xác hơn là `tạoMôTảChoNode`, vì nó chứa những thông tin mô tả node mà Vue cần biết để render, bao gồm cả mô tả cho các node con. Chúng ta gọi mô tả này là một "virtual node" (node ảo), thường viết tắt là **VNode**. "Virtual DOM" là tên của toàn bộ một cây các Vnode, được xây dựng từ một cây các component Vue.

## Các tham số của `createElement`

Điều tiếp theo mà bạn cần thông thạo là cách dùng các tính năng của template trong hàm `createElement`. Đây là danh sách các tham số mà `createElement` nhận:

``` js
// @returns {VNode}
createElement(
  // {String | Object | Function}
  // Một tên thẻ HTML, các tùy chọn cho component, hoặc một hàm
  // trả về một trong những thứ này. Bắt buộc.
  'div',

  // {Object}
  // Một "data object" chứa dữ liệu tương ứng với các thuộc tính
  // mà bạn muốn dùng trong một template. Không bắt buộc.
  {
    // (Xem chi tiết ở phần tiếp theo bên dưới)
  },

  // {String | Array}
  // Các vnode con, được tạo ra bằng cách dùng `createElement()`,
  // hoặc dùng chuỗi để tạo các 'text VNode'. Không bắt buộc.
  [
    'Một ít text trước.',
    createElement('h1', 'Một tiêu đề'),
    createElement(MyComponent, {
      props: {
        someProp: 'foobar'
      }
    })
  ]
)
```

### Chi tiết về data object

Một điểm cần lưu ý: tương tự với việc được [đối xử đặc biệt](class-and-style.html) trong template, `v-bind:class` và `v-bind:class` cũng có các field (trường) riêng ở top-level (cấp cao nhất) trong data object của Vnode. Object này cũng cho phép bạn bind (ràng buộc) các thuộc tính HTML thông thường cũng như các thuộc tính DOM như `innerHTML` (thay thế cho directive `v-html`):

``` js
{
  // Có cùng API với `v-bind:class`
  'class': {
    foo: true,
    bar: false
  },
  // Có cùng API với `v-bind:style`
  style: {
    color: 'red',
    fontSize: '14px'
  },
  // Các thuộc tính HTML thông thường
  attrs: {
    id: 'foo'
  },
  // Các prop cho component
  props: {
    myProp: 'bar'
  },
  // Các thuộc tính DOM
  domProps: {
    innerHTML: 'baz'
  },
  // Các hàm xử lí sự kiện được lồng vào bên trong `on`,
  // tuy rằng modifier (như trong `v-on:keyup.enter`) không được
  // hỗ trợ. Thay vào đó, bạn sẽ phải tự kiểm tra keyCode bên
  // trong hàm.
  on: {
    click: this.clickHandler
  },
  // Chỉ dành riêng cho component. Cho phép bạn lắng nghe các
  // sự kiện native, thay vì các sự kiện được phát ra từ component
  // bằng cách sử dụng `vm.$emit`.
  nativeOn: {
    click: this.nativeClickHandler
  },
  // Các directive tùy biến. Lưu ý là bạn không thể gán giá trị
  // `oldValue`, vì Vue sẽ tự động quản lí giá trị này thay bạn.
  directives: [
    {
      name: 'my-custom-directive',
      value: '2',
      expression: '1 + 1',
      arg: 'foo',
      modifiers: {
        bar: true
      }
    }
  ],
  // Các scoped slot dưới dạng
  // { name: props => VNode | Array<VNode> }
  scopedSlots: {
    default: props => createElement('span', props.text)
  },
  // Tên của slot, nếu component này là con của một component khác
  // child of another component
  slot: 'name-of-slot',
  // Các thuộc tính top-level khác
  key: 'myKey',
  ref: 'myRef'
}
```

### Ví dụ hoàn chỉnh

With this knowledge, we can now finish the component we started:

``` js
var getChildrenTextContent = function (children) {
  return children.map(function (node) {
    return node.children
      ? getChildrenTextContent(node.children)
      : node.text
  }).join('')
}

Vue.component('anchored-heading', {
  render: function (createElement) {
    // create kebabCase id
    var headingId = getChildrenTextContent(this.$slots.default)
      .toLowerCase()
      .replace(/\W+/g, '-')
      .replace(/(^\-|\-$)/g, '')

    return createElement(
      'h' + this.level,
      [
        createElement('a', {
          attrs: {
            name: headingId,
            href: '#' + headingId
          }
        }, this.$slots.default)
      ]
    )
  },
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```

### Constraints

#### VNodes Must Be Unique

All VNodes in the component tree must be unique. That means the following render function is invalid:

``` js
render: function (createElement) {
  var myParagraphVNode = createElement('p', 'hi')
  return createElement('div', [
    // Yikes - duplicate VNodes!
    myParagraphVNode, myParagraphVNode
  ])
}
```

If you really want to duplicate the same element/component many times, you can do so with a factory function. For example, the following render function is a perfectly valid way of rendering 20 identical paragraphs:

``` js
render: function (createElement) {
  return createElement('div',
    Array.apply(null, { length: 20 }).map(function () {
      return createElement('p', 'hi')
    })
  )
}
```

## Replacing Template Features with Plain JavaScript

### `v-if` and `v-for`

Wherever something can be easily accomplished in plain JavaScript, Vue render functions do not provide a proprietary alternative. For example, in a template using `v-if` and `v-for`:

``` html
<ul v-if="items.length">
  <li v-for="item in items">{{ item.name }}</li>
</ul>
<p v-else>No items found.</p>
```

This could be rewritten with JavaScript's `if`/`else` and `map` in a render function:

``` js
render: function (createElement) {
  if (this.items.length) {
    return createElement('ul', this.items.map(function (item) {
      return createElement('li', item.name)
    }))
  } else {
    return createElement('p', 'No items found.')
  }
}
```

### `v-model`

There is no direct `v-model` counterpart in render functions - you will have to implement the logic yourself:

``` js
render: function (createElement) {
  var self = this
  return createElement('input', {
    domProps: {
      value: self.value
    },
    on: {
      input: function (event) {
        self.value = event.target.value
        self.$emit('input', event.target.value)
      }
    }
  })
}
```

This is the cost of going lower-level, but it also gives you much more control over the interaction details compared to `v-model`.

### Event & Key Modifiers

For the `.passive`, `.capture` and `.once` event modifiers, Vue offers prefixes that can be used with `on`:

| Modifier(s) | Prefix |
| ------ | ------ |
| `.passive` | `&` |
| `.capture` | `!` |
| `.once` | `~` |
| `.capture.once` or<br>`.once.capture` | `~!` |

For example:

```javascript
on: {
  '!click': this.doThisInCapturingMode,
  '~keyup': this.doThisOnce,
  `~!mouseover`: this.doThisOnceInCapturingMode
}
```

For all other event and key modifiers, no proprietary prefix is necessary, because you can use event methods in the handler:

| Modifier(s) | Equivalent in Handler |
| ------ | ------ |
| `.stop` | `event.stopPropagation()` |
| `.prevent` | `event.preventDefault()` |
| `.self` | `if (event.target !== event.currentTarget) return` |
| Keys:<br>`.enter`, `.13` | `if (event.keyCode !== 13) return` (change `13` to [another key code](http://keycode.info/) for other key modifiers) |
| Modifiers Keys:<br>`.ctrl`, `.alt`, `.shift`, `.meta` | `if (!event.ctrlKey) return` (change `ctrlKey` to `altKey`, `shiftKey`, or `metaKey`, respectively) |

Here's an example with all of these modifiers used together:

```javascript
on: {
  keyup: function (event) {
    // Abort if the element emitting the event is not
    // the element the event is bound to
    if (event.target !== event.currentTarget) return
    // Abort if the key that went up is not the enter
    // key (13) and the shift key was not held down
    // at the same time
    if (!event.shiftKey || event.keyCode !== 13) return
    // Stop event propagation
    event.stopPropagation()
    // Prevent the default keyup handler for this element
    event.preventDefault()
    // ...
  }
}
```

### Slots

You can access static slot contents as Arrays of VNodes from [`this.$slots`](../api/#vm-slots):

``` js
render: function (createElement) {
  // `<div><slot></slot></div>`
  return createElement('div', this.$slots.default)
}
```

And access scoped slots as functions that return VNodes from [`this.$scopedSlots`](../api/#vm-scopedSlots):

``` js
render: function (createElement) {
  // `<div><slot :text="msg"></slot></div>`
  return createElement('div', [
    this.$scopedSlots.default({
      text: this.msg
    })
  ])
}
```

To pass scoped slots to a child component using render functions, use the `scopedSlots` field in VNode data:

``` js
render (createElement) {
  return createElement('div', [
    createElement('child', {
      // pass `scopedSlots` in the data object
      // in the form of { name: props => VNode | Array<VNode> }
      scopedSlots: {
        default: function (props) {
          return createElement('span', props.text)
        }
      }
    })
  ])
}
```

## JSX

If you're writing a lot of `render` functions, it might feel painful to write something like this:

``` js
createElement(
  'anchored-heading', {
    props: {
      level: 1
    }
  }, [
    createElement('span', 'Hello'),
    ' world!'
  ]
)
```

Especially when the template version is so simple in comparison:

``` html
<anchored-heading :level="1">
  <span>Hello</span> world!
</anchored-heading>
```

That's why there's a [Babel plugin](https://github.com/vuejs/babel-plugin-transform-vue-jsx) to use JSX with Vue, getting us back to a syntax that's closer to templates:

``` js
import AnchoredHeading from './AnchoredHeading.vue'

new Vue({
  el: '#demo',
  render (h) {
    return (
      <AnchoredHeading level={1}>
        <span>Hello</span> world!
      </AnchoredHeading>
    )
  }
})
```

<p class="tip">Aliasing `createElement` to `h` is a common convention you'll see in the Vue ecosystem and is actually required for JSX. If `h` is not available in the scope, your app will throw an error.</p>

For more on how JSX maps to JavaScript, see the [usage docs](https://github.com/vuejs/babel-plugin-transform-vue-jsx#usage).

## Functional Components

The anchored heading component we created earlier is relatively simple. It doesn't manage any state, watch any state passed to it, and it has no lifecycle methods. Really, it's only a function with some props.

In cases like this, we can mark components as `functional`, which means that they're stateless (no `data`) and instanceless (no `this` context). A **functional component** looks like this:

``` js
Vue.component('my-component', {
  functional: true,
  // To compensate for the lack of an instance,
  // we are now provided a 2nd context argument.
  render: function (createElement, context) {
    // ...
  },
  // Props are optional
  props: {
    // ...
  }
})
```

> Note: in versions before 2.3.0, the `props` option is required if you wish to accept props in a functional component. In 2.3.0+ you can omit the `props` option and all attributes found on the component node will be implicitly extracted as props.

Everything the component needs is passed through `context`, which is an object containing:

- `props`: An object of the provided props
- `children`: An array of the VNode children
- `slots`: A function returning a slots object
- `data`: The entire data object passed to the component
- `parent`: A reference to the parent component
- `listeners`: (2.3.0+) An object containing parent-registered event listeners. This is an alias to `data.on`
- `injections`: (2.3.0+) if using the [`inject`](../api/#provide-inject) option, this will contain resolved injections.

After adding `functional: true`, updating the render function of our anchored heading component would require adding the `context` argument, updating `this.$slots.default` to `context.children`, then updating `this.level` to `context.props.level`.

Since functional components are just functions, they're much cheaper to render. However, the lack of a persistent instance means they won't show up in the [Vue devtools](https://github.com/vuejs/vue-devtools) component tree.

They're also very useful as wrapper components. For example, when you need to:

- Programmatically choose one of several other components to delegate to
- Manipulate children, props, or data before passing them on to a child component

Here's an example of a `smart-list` component that delegates to more specific components, depending on the props passed to it:

``` js
var EmptyList = { /* ... */ }
var TableList = { /* ... */ }
var OrderedList = { /* ... */ }
var UnorderedList = { /* ... */ }

Vue.component('smart-list', {
  functional: true,
  render: function (createElement, context) {
    function appropriateListComponent () {
      var items = context.props.items

      if (items.length === 0)           return EmptyList
      if (typeof items[0] === 'object') return TableList
      if (context.props.isOrdered)      return OrderedList

      return UnorderedList
    }

    return createElement(
      appropriateListComponent(),
      context.data,
      context.children
    )
  },
  props: {
    items: {
      type: Array,
      required: true
    },
    isOrdered: Boolean
  }
})
```

### `slots()` vs `children`

You may wonder why we need both `slots()` and `children`. Wouldn't `slots().default` be the same as `children`? In some cases, yes - but what if you have a functional component with the following children?

``` html
<my-functional-component>
  <p slot="foo">
    first
  </p>
  <p>second</p>
</my-functional-component>
```

For this component, `children` will give you both paragraphs, `slots().default` will give you only the second, and `slots().foo` will give you only the first. Having both `children` and `slots()` therefore allows you to choose whether this component knows about a slot system or perhaps delegates that responsibility to another component by passing along `children`.

## Template Compilation

You may be interested to know that Vue's templates actually compile to render functions. This is an implementation detail you usually don't need to know about, but if you'd like to see how specific template features are compiled, you may find it interesting. Below is a little demo using `Vue.compile` to live-compile a template string:

{% raw %}
<div id="vue-compile-demo" class="demo">
  <textarea v-model="templateText" rows="10"></textarea>
  <div v-if="typeof result === 'object'">
    <label>render:</label>
    <pre><code>{{ result.render }}</code></pre>
    <label>staticRenderFns:</label>
    <pre v-for="(fn, index) in result.staticRenderFns"><code>_m({{ index }}): {{ fn }}</code></pre>
    <pre v-if="!result.staticRenderFns.length"><code>{{ result.staticRenderFns }}</code></pre>
  </div>
  <div v-else>
    <label>Compilation Error:</label>
    <pre><code>{{ result }}</code></pre>
  </div>
</div>
<script>
new Vue({
  el: '#vue-compile-demo',
  data: {
    templateText: '\
<div>\n\
  <header>\n\
    <h1>I\'m a template!</h1>\n\
  </header>\n\
  <p v-if="message">\n\
    {{ message }}\n\
  </p>\n\
  <p v-else>\n\
    No message.\n\
  </p>\n\
</div>\
    ',
  },
  computed: {
    result: function () {
      if (!this.templateText) {
        return 'Enter a valid template above'
      }
      try {
        var result = Vue.compile(this.templateText.replace(/\s{2,}/g, ''))
        return {
          render: this.formatFunction(result.render),
          staticRenderFns: result.staticRenderFns.map(this.formatFunction)
        }
      } catch (error) {
        return error.message
      }
    }
  },
  methods: {
    formatFunction: function (fn) {
      return fn.toString().replace(/(\{\n)(\S)/, '$1  $2')
    }
  }
})
console.error = function (error) {
  throw new Error(error)
}
</script>
<style>
#vue-compile-demo {
  -webkit-user-select: inherit;
  user-select: inherit;
}
#vue-compile-demo pre {
  padding: 10px;
  overflow-x: auto;
}
#vue-compile-demo code {
  white-space: pre;
  padding: 0
}
#vue-compile-demo textarea {
  width: 100%;
  font-family: monospace;
}
</style>
{% endraw %}
