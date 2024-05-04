## 1. Method

- Method là danh sách các phương thức của component
- Method trong Vue không khác gì mấy với hàm (function) trong javascript khi ta muốn

```javascript
<script setup>
import { ref } from 'vue'
const msg = ref('Hello World!')
function updateMsg() {
  msg.value = "My name is James"
}
</script>
<template>
  <div class="my-component">{{ msg }}</div>
  <button @click="updateMsg">
    Update
  </button>
</template>
<style lang="scss" scoped>
.my-component {
  color: red;
}
</style>
```

## 2. Computed

- `computed` sẽ theo dõi các reactive state bên trong nó và tự tính toán nếu như có bất kì reactive thay đổi.
- Đối số truyền vào của `computed` là một function callback, không nên truyền vào đây một bất đồng bộ hay truy cập thao tác với DOM
- Hạn chế thay đổi giá trị của computed, rất ít khi cần đến `writeable computed`, nếu cần cập nhật computed thì chỉ cần thay đổi bất kì reactive state trong nó là được

```javascript
<script setup>
import { ref, computed } from 'vue'

const age = ref(16)

const msg = computed(() => {
  return age.value >= 50 ? 'Senior' : age.value > 25 && age.value < 50 ? 'Adult' : 'Young'
})

function updateAge() {
  age.value = 50
}
</script>

- Thực thể `msg` là một `ComputedRef`, nên khi muốn lấy giá trị ở `<script></script>` cần `.value`

<template>
  <div class="my-component">{{ msg }}</div>
  <button @click="updateAge">
    Update
  </button>
</template>
```

### ???

Lúc này có một câu hỏi đặt ra? Giữa `computed` và `method` có gì khác nhau? Tại sao lại không sử dụng `method` để tính toán nếu như kết quả đầu ra đều như nhau?

> Giữa chúng thức sự có sự khác nhau. `computed` theo dõi các reactive state trong nó, nếu chỉ tính toán lại khi các reactive state trong nó thay đổi. Trong khi `method` luôn thực hiện lại mỗi khi có reactive state của component thay đổi. Điều này dẫn tới việc _**performance**_ hay hiệu suất có sự khác nhau.

## 3. Watcher

### 3.1. Basic

- Khi ta cần thực hiện các `side effect` khi reactive state thay đổi (call API, DOM API hay thay đổi 1 reactive state) thì `computed` không thích hợp làm điều này

```javascript
<script setup>
import { ref, watch } from 'vue'

const count = ref(0)
// const count = reactive({ count: 0}); - reactive

watch(count, (current) => {
  console.log('Current value of count:', current)
})

function updateCount() {
  count.value++
}
</script>

<template>
  <h1>{{ count }}</h1>
  <button @click="updateCount">
    Update Count
  </button>
</template>

```

> Tham số đầu tiên truyền vào gọi là `source`, mục đích để `watch` hiểu ta muốn theo dõi cái nguồn nào. `source` có thể là ref, computed ref, reactive, hàm getter hoặc một mảng chứa các nguồn <br> <br> Tham số thứ 2 là 1 callback với tham số trả về là giá trị hiện tại của `source` ta đang theo dõi <br> <br> Tham số thứ 2 của callback sẽ là giá trị cũ của `source`

!!!. Với `source` là một mảng:

```javascript
<script setup>
import { ref, watch } from 'vue'

const count1 = ref(0)
const count2 = ref(0)

watch([count1, count2], (current) => {
  console.log('Current value:', current)
})

function updateCounts() {
  count1.value++
  count2.value += 2
}
</script>

<template>
  <h1>{{ count1 }} - {{ count2 }}</h1>
  <button @click="updateCounts">
    Update Count
  </button>
</template>
```

### Chú ý:

- Không nên watch trực tiếp giá trị của một reactive state vì khi ta truyền vào giá trị thì nó chỉ là một giá trị thường mà không hề có sự thay đổi

### 3.2. Deep Watcher

- Khi ta muốn thay đổi 1 thuộc tính mãi bên trong một reactive state, `watch` không thể hiểu được vì nó chỉ đang theo dõi sự thay đổi của reactive state. Vì vậy để làm được điều đó ta thêm lựa chọn `{deep: true}` cho `watch`

- Chú ý: Không nên thực hiện deep watch khi đối tượng mà `watch` theo dõi có quá nhiều thuộc tính bên trong. Chỉ nên thực hiện khi cần thiết vì sẽ dẫn tới ảnh hưởng đến hiệu suất

```js
watch(
  state,
  (current) => {
    console.log(current);
  },
  { deep: true }
);
```

### 3.3. Eager Watcher (Immediate Watch)

Mặc định `watch` sẽ chạy khi reactive thay đổi, nhưng cũng có những trường hợp ta muốn `watch` chạy ngay lập tức khi mà state chưa thay đổi. Trong trường hợp đó ta sử dụng option `immediate: true` cho `watch`

### 3.4. Once Watchers

`watch` sẽ thực hiện hàm callback mỗi khi state có sự thay đổi. Nếu bạn muốn callback chỉ thực hiện mỗi khi nguồn thay đổi thì sử dụng option `{once: true}`

### 3.5. watchEffect()

```js
<script setup>
import { ref, watch } from 'vue'

const count = ref(1)

watch(count, async (current) => {
  const response = await fetch(
    `https://jsonplaceholder.typicode.com/todos/${current}`
  )
  const json = await response.json()
  console.log(json)
}, { immediate: true })

function updateCount() {
  count.value++
}
</script>

<template>
  <h1>{{ count }}</h1>
  <button @click="updateCount">
    Update Count
  </button>
</template>
```

- Giải pháp khác:

```js
<script setup>
import { ref, watchEffect } from 'vue'

const count = ref(1)

watchEffect(async () => {
  const response = await fetch(
    `https://jsonplaceholder.typicode.com/todos/${count.value}`
  )
  const json = await response.json()
  console.log(json)
})

function updateCount() {
  count.value++
}
</script>

<template>
  <h1>{{ count }}</h1>
  <button @click="updateCount">
    Update Count
  </button>
</template>
```

- Chú ý: Không còn source trong `watch` nữa mà chỉ còn callback và callback này chạy ngay lập tức như option `{immediate: true}`. Vue sẽ tự theo dõi những reactive state trong callback

### ??? Khác biệt giữa `watch()` và `watchEffect()`

> `watch()`: chỉ theo dõi những thứ ta khai báo ở source, không theo dõi có gì bên trong callback, tách biệt 2 phần source và callback. Điều này giúp ta kiểm soát chính xác những gì ta cần theo dõi <br> `watchEffect()`: gộp 2 phần trên làm một, code sẽ gọn hơn và tiện hơn

### 3.6. Callback Flush Timing

```js
<script setup>
import { ref, watch } from 'vue'

const count = ref(0)

watch(count, (current) => {
  const textContent = document.getElementById('count').textContent
  const currentStr = current.toString()
  console.log(textContent === currentStr, 'textContent: ' + textContent, 'current: ' + current)
})

function updateCount() {
  count.value++
}
</script>

<template>
  <h1 id="count">{{ count }}</h1>
  <button @click="updateCount">
    Update Count
  </button>
</template>

```

- Trong thực tế, callback sẽ chạy trước khi Vue thực hiện update component, do vậy thời điểm chạy thì ở DOM HTML vẫn là giá trị cũ trước đó.
- Ta muốn callback được chạy sao khi Vue update component thì ta thêm option `{ flush: post }`
- Điều này tương tự với `watchEffect()` và vue cũng cung cấp cho ta 1 hàm là `watchPostEffect()` để tiện cho `watchEffect()`

### 3.7. Stopping a Watcher

- Mặc định thì watcher sẽ phụ thuộc theo component, khi component bị huyyr thì watcher cũng hủy theo.
- Nếu watcher được khai báo `async` như `setTimeOut, setInterval, Promise...`, nó sẽ không ràng buộc với component và dẫn tới tình trạng rò rỉ bộ nhớ. Để dừng lại điều này, ta thực hiện theo cách thủ công sau:

```js
const unwatch = watchEffect(() => {});

// ...later, when no longer needed
unwatch();
```

- Có rất ít trường hợp phải cần đến 1 bất đồng bộ và nên sử dụng đồng bộ khi có thể

!!! Take note

- Code UI nên tập trung vào code UI, nên đẩy các phần tình toán xử lý cho JS `<script></script>`, không nên để rườm rà dài dòng trong `<template></template>`
