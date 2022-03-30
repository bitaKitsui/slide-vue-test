---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
---

# Vueのテストについて
test-utilsとtesting-libraryの違い

---

# test-utilsとは？
- Vue公式の単体テスト用のライブラリ
- Vue2までと、Vue3用と、それぞれ異なるパッケージが用意されている

```
// Vue2用
npm install --save-dev @vue/test-utils@1
npm install --save-dev @vue/server-test-utils@1

// Vue3用
npm install @vue/test-utils --save-dev
```

---

# test-utilsの目的
- Vueコンポーネントのテストを簡素化するもの
- Vueコンポーネントをマウントし、そのコンポーネントと対話するためのメソッドなどを提供している


---

# 基本的な使い方
- まずはコンポーネントを`mount`する
  - test-utilsでコンポーネントをレンダリングできる
- その結果を`wrapper`という変数に割り当てる（慣例として）
- `wrapper`に含まれている便利なメソッドを使う

```js {all|5|7-9}
import { mount } from '@vue/test-utils'
import TodoApp from './TodoApp.vue'

test('renders a todo', () => {
  const wrapper = mount(TodoApp)

  const todo = wrapper.get('[data-test="todo"]')

  expect(todo.text()).toBe('Learn Vue.js 3')
})
```

---

# 基本的な使い方
- templateに`data-test`属性を追加する

```js
<template>
  <div>
    <div v-for="todo in todos" :key="todo.id" data-test="todo">
      {{ todo.text }}
    </div>
  </div>
</template>
```

---

# 疑問点
<h2>`data-test`属性は良く使うもの？</h2>

---

# 基本的なメソッド
- `setValue`: `<input>`の値を更新できる
- `trigger`: フォームの送信などユーザーの動作をシミュレートできる

```js {all|8|9}
import { mount } from '@vue/test-utils'
import TodoApp from './TodoApp.vue'

test('creates a todo', () => {
  const wrapper = mount(TodoApp)
  expect(wrapper.findAll('[data-test="todo"]')).toHaveLength(1)

  wrapper.get('[data-test="new-todo"]').setValue('New todo')
  wrapper.get('[data-test="form"]').trigger('submit')

  expect(wrapper.findAll('[data-test="todo"]')).toHaveLength(2)
})
```

---

# 基本的なメソッド
- `async`, `await`
  - Jestはテストを同期的に実行し、最後の関数が呼ばれると終了する
  - VueはDOMを非同期で更新する
  - `trigger`や`setValue`も同様

```js
import { mount } from '@vue/test-utils'
import TodoApp from './TodoApp.vue'

test('creates a todo', async () => {
  const wrapper = mount(TodoApp)

  await wrapper.get('[data-test="new-todo"]').setValue('New todo')
  await wrapper.get('[data-test="form"]').trigger('submit')

  expect(wrapper.findAll('[data-test="todo"]')).toHaveLength(2)
})

```

---

# 基本のまとめ
- `arrange`, `act`, `assert`の3つの流れ
  - `arrange`: テストのためのシナリオを設定する（仮のstoreやDBを作るなど）
  - `act`: コンポーネントやアプリケーションをどのように操作するかをシミュレートし、シナリオを実行する
  - `assert`: コンポーネントの現在の状態がどうであるかについてアサーション（断言、判定など）を行う
- 基本的に全てのテストはこの3つのフェーズに従うことになる

```js
import { mount } from '@vue/test-utils'
import TodoApp from './TodoApp.vue'

test('creates a todo', async () => {
  const wrapper = mount(TodoApp)

  await wrapper.get('[data-test="new-todo"]').setValue('New todo')
  await wrapper.get('[data-test="form"]').trigger('submit')

  expect(wrapper.findAll('[data-test="todo"]')).toHaveLength(2)
})
```

---

# testing-libraryとは？

<div style="display: flex; gap: 40px;">
  <ul>
    <li>Kent C. Dodds氏が携わるテストライブラリー</li>
    <li>凄腕エンジニア</li>
    <li>現Remixの開発者</li>
    <li>Testing Trophyの考案者</li>
  </ul>
  <img src="/testing_trophy.jpeg" class="h-100" alt="" >
</div>

---

# testing-libraryとは？

- シンプルで完璧なテストユーティリティで、優れたテストプラクティスを促進する
- メンテナンスしやすいテストを書ける
- 自身を持って開発できる
- アクセシビリティ性も考慮されている
- はっきりとした指針が打ち出されている

---

# testing-libraryの指針

- コンポーネントのレンダリングに関連するものについては、インスタンスではなくDOMノードを扱うべきである
- ユーザーが使用する方法でテストをすべきである
- 使用するAPIはシンプルで柔軟であるべきである
- テストするコンポーネントの内部など、実装の詳細をテストしないことを推奨する

ウェブページがユーザーによってどのように操作されるかについて重点を置いている<br>
DOMにアクセスする方法が細かく優先付けされている

---

# vue-testing-libraryとは？

- 基本となるDOM testing-libraryの上に、Vueコンポーネントを操作するためのAPIを追加して構築されている
  - test-utilsの上に軽いユーティリティ関数を提供した形になっている

---

# vue-testing-libraryを使う目的

- Vueのコンポーネントのために保守性の高いテストを書きたい
  - そのためにコンポーネントの実装の詳細をテストに含めたくない
  - むしろ、テストが意図した通りの信頼性を得られるようにすることに重点を置きたい
  - そこで、レンダリングされたVueコンポーネントのインスタンスを扱うのではなく、実際のDOMノードを用いてテストを行う
  - テキストからリンクを見つけたり、アクセシビリティが担保されているかなど、容易に確認ができるユーティリティ関数がある


---

# 軽くまとめ

- test-utilsはVueのコンポーネントを簡単にテストできるもの
  - Vueのルール内でテストをするイメージ
- testing-libraryはメンテナンス性やアクセシビリティを意識したテストができるもの
  - 開発者やユーザー目線というイメージ
