# Composition API

> 이 섹션에서는 코드 예제에 [싱글 파일 컴포넌트(SFC)](../guide/single-file-component.html) 구문을 사용합니다.

## `setup`

컴포넌트가 생성되기 <strong>전에</strong> <code>props</code>를 반환(resolved)되면 실행되는 컴포넌트 옵션으로 composition API의 진입점 역할을 합니다.

- **전달인자:**

    - `{Data} props`
    - `{SetupContext} context`

- <strong>작성법</strong>:

```ts
interface Data {
  [key: string]: unknown
}

interface SetupContext {
  attrs: Data
  slots: Slots
  emit: (event: string, ...args: unknown[]) => void
}

function setup(props: Data, context: SetupContext): Data
```

::: tip 
`setup()`에 전달된 인자에 대한 타입을 추론하려면, [defineComponent](global-api.html#definecomponent) 를 사용해야합니다. 
:::

- **예시**

    템플릿 사용:

    ```vue-html
    <!-- MyBook.vue -->
    <template>
      <div>{{ readersNumber }} {{ book.title }}</div>
    </template>

    <script>
      import { ref, reactive } from 'vue'

      export default {
        setup() {
          const readersNumber = ref(0)
          const book = reactive({ title: 'Vue 3 Guide' })

          // 템플릿에 노출
          return {
            readersNumber,
            book
          }
        }
      }
    </script>
    ```

    렌더 함수 사용:

    ```js
    // MyBook.vue

    import { h, ref, reactive } from 'vue'

    export default {
      setup() {
        const readersNumber = ref(0)
        const book = reactive({ title: 'Vue 3 Guide' })
        // 참조값(ref value)을 명시적으로 노출해야합니다.
        return () => h('div', [readersNumber.value, book.title])
      }
    }
    ```

- **참고**: [Composition API `setup`](../guide/composition-api-setup.html)

## 라이프사이클 훅(Lifecycle Hooks)

라이프사이클 훅은 import를 사용하여 직접적으로 `onX` 함수에 등록할 수 있습니다:

```js
import { onMounted, onUpdated, onUnmounted } from 'vue'

const MyComponent = {
  setup() {
    onMounted(() => {
      console.log('mounted!')
    })
    onUpdated(() => {
      console.log('updated!')
    })
    onUnmounted(() => {
      console.log('unmounted!')
    })
  }
}
```

이러한 라이프사이클 훅 등록 함수는 내부 전역 상태에 의존하여 현재 활성 인스턴스(`setup()`이 지금 호출되는 컴포넌트 인스턴스)를 찾기 때문에 <a data-md-type="raw_html" href="#setup">`setup()`</a>중에 동기식으로만 사용할 수 있습니다. 현재 활성 인스턴스없이 호출하면 오류가 발생합니다.

컴포넌트 인스턴스 컨텍스트는 라이프사이클 훅의 동기 실행 중에도 설정됩니다. 결론적으로 라이프사이클 훅 내에서 동기적으로 생성된 감시자(watchers)와 계산된 속성(computed properties)도 컴포넌트가 마운트 해제될 때 자동으로 해제됩니다.

- **옵션 API 라이프사이클 옵션과 Composition API 간의 매핑**

    - ~~`beforeCreate`~~ -> `setup()` 사용
    - ~~`created`~~ -> `setup()`  사용
    - `beforeMount` -> `onBeforeMount`
    - `mounted` -> `onMounted`
    - `beforeUpdate` -> `onBeforeUpdate`
    - `updated` -> `onUpdated`
    - `beforeUnmount` -> `onBeforeUnmount`
    - `unmounted` -> `onUnmounted`
    - `errorCaptured` -> `onErrorCaptured`
    - `renderTracked` -> `onRenderTracked`
    - `renderTriggered` -> `onRenderTriggered`

- **참고**: [Composition API lifecycle hooks](../guide/composition-api-lifecycle-hooks.html)

## Provide / Inject

`provide` 및 <code>inject</code> 는 종속성 주입을 활성화합니다. 둘 다 현재 활성 인스턴스로 <a><code>setup()</code></a>동안만 호출 할 수 있습니다.

- **작성법**:

```ts
interface InjectionKey<T> extends Symbol {}

function provide<T>(key: InjectionKey<T> | string, value: T): void

// 기본값 없는 경우
function inject<T>(key: InjectionKey<T> | string): T | undefined
// 기본값 있는 경우
function inject<T>(key: InjectionKey<T> | string, defaultValue: T): T
```

Vue는 `Symbol`을 확장하는 일반유형인 `InjectionKey` 인터페이스를 제공합니다. 공급자(provider)와 소비자(consumer)간에 삽입된 값의 유형을 동기화하는데 사용할 수 있습니다.

```ts
import { InjectionKey, provide, inject } from 'vue'

const key: InjectionKey<string> = Symbol()

provide(key, 'foo') // 문자열이 아닌 값을 제공하면 오류가 발생합니다

const foo = inject(key) // foo의 타입: string | undefined
```

문자열 키(string keys) 또는 형식화되지 않은 심볼(non-typed symbols)을 사용하는 경우 삽입된 값의 타입을 명시적으로 선언해야합니다:

```ts
const foo = inject<string>('foo') // string | undefined
```

- **참고**:
    - [Provide / Inject](../guide/component-provide-inject.html)
    - [Composition API Provide / Inject](../guide/composition-api-provide-inject.html)

## `getCurrentInstance`

`getCurrentInstance`를 사용하면 고급 사용이나 라이브러리를 생성하는 이에게 유용한 내부 컴포넌트 인스턴스에 접근할 수 있습니다.

```ts
import { getCurrentInstance } from 'vue'

const MyComponent = {
  setup() {
    const internalInstance = getCurrentInstance()

    internalInstance.appContext.config.globalProperties // globalProperties에 접근
  }
}
```

`getCurrentInstance`는 **오직 ** [setup](#setup) 이나 [Lifecycle Hooks](#lifecycle-hooks)에서만 작동합니다.

> [setup](#setup)이나 [Lifecycle Hooks](#lifecycle-hooks) 외부에서 사용하는 경우, `setup`에서 `getCurrentInstance()`를 호출하고 대신 인스턴스를 사용하십시오.

```ts
const MyComponent = {
  setup() {
    const internalInstance = getCurrentInstance() // 작동함

    const id = useComponentId() // 작동함

    const handleClick = () => {
      getCurrentInstance() // 작동안함
      useComponentId() // 작동안함

      internalInstance // 작동함
    }

    onMounted(() => {
      getCurrentInstance() // 작동함
    })

    return () =>
      h(
        'button',
        {
          onClick: handleClick
        },
        `uid: ${id}`
      )
  }
}

// 컴포저블(composable)에서 호출되는 경우에도 작동합니다
function useComponentId() {
  return getCurrentInstance().uid
}
```
