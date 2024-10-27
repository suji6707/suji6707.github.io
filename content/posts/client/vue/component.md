+++
title = 'Component'
date = 2024-09-25T00:20:22+09:00
draft = true
+++
## Vue 컴포넌트


---
### Pass-through: 커스텀 CSS 적용

### :style CSS로 한번에 됨
```js
const dialogStyle = {
    position: 'fixed',
    top: '20%',
    left: '50%',
    transform: 'translateX(-50%)',
    width: '25rem'
};
</script>

<template>
  <div class="create-view">
    <Dialog
      v-model:visible="localVisible"
      modal
      header="Create View"
      :style="dialogStyle"
      @update:visible="updateVisible"
    >
```

---
💎 참고로 PrimeVue 컴포넌트에 클래스를 덧붙인 경우 
해당 클래스명이 scoped CSS 적용이 안되는듯.
이 경우 scope 밖에 정의하면 됨

또는 아래처럼 pt로.

```js
<Tab
  v-for="(tab, index) in tabs"
  :key="tab.title"
  :value="tab.value"
  pt:
  style="padding: 5px; margin: 0 10px; border: none;"
  @click="activeTabIndex = index"
>
```

💎 클래스가 덧붙여진 경우: 클래스를 붙여줘야함. (여러 클래스가 동시에 적용된 경우) 
`.p-tab.custom-tab {` 
💎 부모 자식 관계인 경우 띄어쓰기


---
### Tabs

좋은 질문입니다! `key`와 `value`는 서로 다른 역할을 수행하기 때문에 둘 다 필요합니다. 

**1. `key` prop:**

* **Vue.js 내부 동작:** Vue.js가 `v-for` 렌더링 시 각 요소를 식별하기 위해 사용하는 **고유 식별자**입니다. 
* **DOM 업데이트:** `key` 값을 기반으로 변경된 요소만 업데이트하여 DOM 조작을 최소화하고 성능을 향상시킵니다.
* **값 유형:** 숫자, 문자열, 심볼 등 고유한 값이면 가능합니다.

**2. `value` prop:**

* **컴포넌트 상태:** `Tabs` 컴포넌트처럼 **현재 상태를 나타내는 값**을 전달하는 데 사용됩니다.
* **동작 제어:** `value` 값에 따라 탭의 활성화/비활성화 상태를 동적으로 변경할 수 있습니다.
* **값 유형:** 컴포넌트에 따라 다르지만, 일반적으로 숫자, 문자열, 불리언 값을 사용합니다.

**탭 컴포넌트 예시:**

* `key`: 각 탭 요소를 구분하기 위한 고유 ID (예: `tab.id`)
* `value`: 해당 탭의 인덱스 또는 고유 값 (예: `0`, `1`, `'tab-a'`)

**요약:**

| 속성 | 역할 | 값 유형 |
|---|---|---|
| `key` | Vue.js 내부에서 요소를 식별하고 DOM 업데이트 성능을 향상 | 숫자, 문자열, 심볼 (고유해야 함) |
| `value` | 컴포넌트의 현재 상태를 나타내고 동작을 제어 | 컴포넌트에 따라 다름 (일반적으로 숫자, 문자열, 불리언) |

**결론적으로, `key`는 Vue.js 내부 동작에 필수적인 값이고, `value`는 컴포넌트의 상태 및 동작을 제어하기 위한 값입니다. 탭 컴포넌트의 경우, 두 속성 모두 필요하며 서로 다른 역할을 수행합니다.** 


---

네, 말씀하신대로 `value="0"` 부분에서 props를 통해 탭 컴포넌트에 초기값을 전달하고 있습니다. 다만, 현재 코드에서 `value` prop에 문자열 `"0"`을 전달하는 것은 몇 가지 문제를 일으킬 수 있습니다. 

**1. 숫자 0으로 설정:**

* 대부분의 탭 컴포넌트 라이브러리는 탭의 인덱스를 숫자 형태로 관리합니다. 따라서 `value` prop에 숫자 `0`을 전달하는 것이 더 적절합니다. 

```js
<Tabs
  :value="activeTabIndex"
>
  <div class="tab-header">
    <TabList>
      <Tab
        v-for="(tab, index) in tabs"
        :key="tab.title"
        :value="tab.value"
        @click="activeTabIndex = index"
      >
        {{ tab.title }}
```



---
### 활성 탭

라우팅이 되더라도 탭이 유지되도록 하려면 현재 활성화된 탭 정보를 저장하고, 이를 기반으로 `TabMenu`의 활성 탭을 설정해야 합니다. Vue Router의 `currentRoute` 객체를 활용하여 현재 라우트를 감지하고, 이에 맞는 탭을 활성화하는 방법을 알아보겠습니다.

```js
<template>
  <div class="dashboard-container">
    <TabMenu :model="items" :activeIndex="activeTabIndex" />
    <router-view />
  </div>
</template>

<script>
import { ref, computed, watch, onMounted } from 'vue';
import { useRouter, useRoute } from 'vue-router';
import { TabMenu } from 'primevue/tabmenu';

export default {
  components: {
    TabMenu,
  },
  setup() {
    const router = useRouter();
    const route = useRoute();
    const items = ref([
      { label: 'Editor', icon: 'pi pi-fw pi-pencil', route: '/editor' },
      { label: 'Recent queries', icon: 'pi pi-fw pi-clock', route: '/recent' },
      { label: 'Saved queries', icon: 'pi pi-fw pi-save', route: '/saved' },
    ]);

    // 현재 라우트에 맞는 탭 인덱스 계산
    const activeTabIndex = computed(() => {
      return items.value.findIndex(item => item.route === route.path);
    });

    // 라우팅이 변경될 때 탭 업데이트
    watch(route, () => {
      const index = items.value.findIndex(item => item.route === route.path);
      if (index !== -1) {
        activeTabIndex.value = index;
      }
    });

    return { items, activeTabIndex };
  },
};
</script>
```

**설명:**

1. **`useRouter` 및 `useRoute`:** Vue Router에서 제공하는 `useRouter` 함수와 `useRoute` 함수를 사용하여 라우터 인스턴스와 현재 라우트 정보를 가져옵니다.
2. **`activeTabIndex` 계산:** `computed` 속성을 사용하여 현재 라우트와 일치하는 탭의 인덱스를 계산합니다. `findIndex` 메서드를 사용하여 `items` 배열에서 현재 라우트와 일치하는 항목의 인덱스를 찾습니다.
3. **`watch`를 사용한 동기화:** `watch` 함수를 사용하여 현재 라우트 (`route`)를 감시합니다. 라우트가 변경될 때마다 콜백 함수가 실행되어 `activeTabIndex`를 업데이트합니다.
4. **`activeIndex` 바인딩:** `TabMenu` 컴포넌트의 `activeIndex` 속성에 `activeTabIndex`를 바인딩하여 활성 탭을 동적으로 설정합니다.

이제 라우팅이 변경되어도 현재 라우트에 해당하는 탭이 활성 상태로 유지됩니다. 

**추가 고려 사항:**

* 탭 컴포넌트의 `to` 속성에 `router-link`를 사용하여 라우팅을 처리할 수도 있습니다.
* 탭의 활성 상태를 로컬 스토리지 또는 쿠키에 저장하여 페이지를 새로 고침하더라도 유지되도록 할 수 있습니다.

이 솔루션을 통해 라우팅 기능을 유지하면서도 탭의 활성 상태를 동기화하여 사용자 경험을 개선할 수 있습니다. 

---

```css
.content {
 overflow: auto;
 width: 100%; 
 height: calc(100vh - 80px);
}
```

---
현재 라우트뿐만 아니라 모든 탭이 고정되어 유지되도록 하려면, 각 탭의 내용을 `router-view` 대신 직접 렌더링하고, 탭 전환 시 해당 컴포넌트만 표시하도록 해야 합니다. 이를 위해 각 탭의 컴포넌트를 동적으로 로드하고, 현재 활성화된 탭에 따라 해당 컴포넌트를 표시하는 구조를 구현할 수 있습니다.

다음은 이를 구현한 예시입니다.

```js
<template>
  <div class="dashboard-container">
    <TabMenu :model="items" :activeIndex="activeTabIndex" @tab-change="onTabChange" />
    <div class="tab-content">
      <Editor v-if="activeTabIndex === 0" />
      <RecentQueries v-if="activeTabIndex === 1" />
      <SavedQueries v-if="activeTabIndex === 2" />
    </div>
  </div>
</template>

<script>
import { ref, computed, watch } from 'vue';
import { useRouter, useRoute } from 'vue-router';
import TabMenu from 'primevue/tabmenu';
import Editor from './components/Editor.vue';
import RecentQueries from './components/RecentQueries.vue';
import SavedQueries from './components/SavedQueries.vue';

export default {
  components: {
    TabMenu,
    Editor,
    RecentQueries,
    SavedQueries,
  },
  setup() {
    const router = useRouter();
    const route = useRoute();
    const items = ref([
      { label: 'Editor', icon: 'pi pi-fw pi-pencil', route: '/editor' },
      { label: 'Recent queries', icon: 'pi pi-fw pi-clock', route: '/recent' },
      { label: 'Saved queries', icon: 'pi pi-fw pi-save', route: '/saved' },
    ]);

    const activeTabIndex = ref(items.value.findIndex(item => item.route === route.path));

    const onTabChange = (e) => {
      const route = items.value[e.index].route;
      router.push(route);
    };

    watch(route, () => {
      activeTabIndex.value = items.value.findIndex(item => item.route === route.path);
    });

    return {
      items,
      activeTabIndex,
      onTabChange,
    };
  },
};
</script>

<style lang="scss" scoped>
.dashboard-container {
  display: flex;
  flex-direction: column;
  height: 100vh;
}

.tab-content {
  flex: 1;
  overflow-y: auto;
}
</style>
```

**설명:**

1. **탭 모델 정의:** `items` 배열에 각 탭의 라벨, 아이콘, 라우트 정보를 정의합니다.
2. **`activeTabIndex` 상태 관리:** `activeTabIndex`는 현재 활성화된 탭의 인덱스를 저장합니다. 초기값은 현재 라우트에 해당하는 탭의 인덱스입니다.
3. **탭 변경 처리:** `onTabChange` 함수는 탭이 변경될 때 호출되며, 새로운 탭의 라우트로 이동합니다.
4. **라우트 변경 감시:** `watch` 함수를 사용하여 라우트가 변경될 때마다 `activeTabIndex`를 업데이트합니다.
5. **탭 내용 렌더링:** `v-if` 디렉티브를 사용하여 현재 활성화된 탭에 해당하는 컴포넌트만 렌더링합니다.

이 구조에서는 각 탭의 컴포넌트가 고정되어 있으며, 탭 전환 시 해당 컴포넌트만 표시됩니다. 이를 통해 모든 탭의 상태가 유지됩니다.

---
## Primevue TabMenu

`<TabMenu>` 컴포넌트는 PrimeVue 라이브러리의 일부로, Vue.js 애플리케이션에서 탭 메뉴를 쉽게 구현할 수 있도록 해줍니다. `active-index`와 같은 prop은 PrimeVue 컴포넌트에서 제공하는 커스텀 prop이며, HTML 표준 요소에는 원래 존재하지 않습니다. PrimeVue는 이러한 커스텀 prop을 사용하여 컴포넌트의 동작을 제어합니다.

PrimeVue의 `TabMenu` 컴포넌트는 내부적으로 다음과 같은 방식으로 구현되어 있을 가능성이 높습니다:

### 1. Prop 정의
`active-index`와 `model` 같은 prop을 정의하여 부모 컴포넌트로부터 데이터를 받습니다.

```javascript
props: {
  model: {
    type: Array,
    required: true,
  },
  activeIndex: {
    type: Number,
    default: 0,
  },
}
```

### 2. 내부 상태 관리
`activeIndex` prop을 기반으로 내부 상태를 관리합니다. `activeIndex`가 변경될 때마다 컴포넌트의 상태를 업데이트합니다.

```javascript
data() {
  return {
    currentIndex: this.activeIndex,
  };
},
watch: {
  activeIndex(newIndex) {
    this.currentIndex = newIndex;
  },
}
```

### 3. 탭 클릭 이벤트 처리
탭을 클릭할 때 `@tab-change` 이벤트를 발생시켜 부모 컴포넌트에 알립니다.

```javascript
methods: {
  onTabClick(index) {
    this.currentIndex = index;
    this.$emit('tab-change', { index });
  },
}
```

### 4. 템플릿 렌더링
`model` prop을 사용하여 탭 메뉴 항목을 렌더링하고, `currentIndex`를 사용하여 활성화된 탭을 표시합니다.

```html
<template>
  <div class="p-tabmenu">
    <ul class="p-tabmenu-nav">
      <li v-for="(item, index) in model" :key="index" :class="{'p-highlight': currentIndex === index}">
        <a @click="onTabClick(index)">
          <i :class="item.icon"></i>
          <span>{{ item.label }}</span>
        </a>
      </li>
    </ul>
  </div>
</template>
```

### 전체적인 구조

```javascript
export default {
  name: 'TabMenu',
  props: {
    model: {
      type: Array,
      required: true,
    },
    activeIndex: {
      type: Number,
      default: 0,
    },
  },
  data() {
    return {
      currentIndex: this.activeIndex,
    };
  },
  watch: {
    activeIndex(newIndex) {
      this.currentIndex = newIndex;
    },
  },
  methods: {
    onTabClick(index) {
      this.currentIndex = index;
      this.$emit('tab-change', { index });
    },
  },
  template: `
    <div class="p-tabmenu">
      <ul class="p-tabmenu-nav">
        <li v-for="(item, index) in model" :key="index" :class="{'p-highlight': currentIndex === index}">
          <a @click="onTabClick(index)">
            <i :class="item.icon"></i>
            <span>{{ item.label }}</span>
          </a>
        </li>
      </ul>
    </div>
  `,
};
```

**설명:**
1. **Prop 정의:** `model`과 `activeIndex` 프로퍼티를 정의하여 부모 컴포넌트로부터 데이터를 받습니다.
2. **내부 상태 관리:** `activeIndex` prop을 기반으로 `currentIndex` 상태를 관리합니다. `activeIndex`가 변경될 때마다 `currentIndex`를 업데이트합니다.
3. **탭 클릭 이벤트 처리:** 탭을 클릭할 때 `onTabClick` 메서드가 실행되어 `currentIndex`를 업데이트하고 `tab-change` 이벤트를 발생시킵니다.
4. **템플릿 렌더링:** `model` prop을 사용하여 탭 메뉴 항목을 렌더링하고, `currentIndex`를 사용하여 활성화된 탭을 표시합니다.

이 구조는 PrimeVue의 `TabMenu` 컴포넌트가 내부적으로 어떻게 동작하는지 설명하는 예시입니다. 실제 구현은 다를 수 있지만, 주요 개념은 위와 비슷할 것입니다. PrimeVue의 소스 코드를 직접 확인하려면 [PrimeVue GitHub 리포지토리](https://github.com/primefaces/primevue)를 참조할 수 있습니다.


---
## 모달

`v-model:visible`은 Vue 3에서 도입된 새로운 `v-model` API의 일부로, 단순히 `v-model`과 `:visible`을 합친 것이 아니라, `v-model` 디렉티브를 사용하여 특정 prop과 이벤트를 바인딩하는 방식입니다. 이는 기본적으로 `v-model`의 확장 형태로, 특정 prop을 양방향 바인딩하고 해당 prop의 변경 사항을 이벤트로 처리합니다.

기본적으로 `v-model`은 다음과 같은 방식으로 동작합니다:

1. `v-model`은 기본적으로 `modelValue` prop과 `update:modelValue` 이벤트를 사용합니다.
2. `v-model:visible`처럼 사용자 정의 키를 사용할 때는, prop 이름과 이벤트 이름을 사용자 정의 키에 맞게 변경합니다.

예를 들어, `v-model:visible`은 내부적으로 다음과 같이 동작합니다:

1. `visible`이라는 prop을 자식 컴포넌트로 전달하고,
2. 자식 컴포넌트가 `visible` prop의 변경 사항을 부모 컴포넌트로 전달하는 이벤트(`update:visible`)를 발생시킵니다.

이를 이해하기 위해 아래와 같은 형태로 변환할 수 있습니다:

```vue
<!-- 부모 컴포넌트 -->
<template>
  <CreateViewDialog :visible="showViewModal" @update:visible="showViewModal = $event" />
</template>

<script setup>
import { ref } from 'vue';
import CreateViewDialog from './CreateViewDialog.vue';

const showViewModal = ref(false);
</script>
```

자식 컴포넌트에서는 `visible`이라는 prop을 받고, `update:visible` 이벤트를 발생시켜 부모 컴포넌트로 변경 사항을 전달합니다:

```vue
<!-- 자식 컴포넌트 (CreateViewDialog.vue) -->
<template>
  <Dialog :visible="visible" @update:visible="$emit('update:visible', $event)" modal header="Create View" :style="{ width: '25rem' }">
    <!-- Dialog content -->
  </Dialog>
</template>

<script setup>
import { defineProps, defineEmits } from 'vue';
import Dialog from 'primevue/dialog';
import InputText from 'primevue/inputtext';
import Button from 'primevue/button';

const props = defineProps({
  visible: Boolean
});

const emit = defineEmits(['update:visible', 'saveView']);

const viewName = ref('');

const closeModal = () => {
  emit('update:visible', false);
};

const saveView = () => {
  emit('saveView', viewName.value);
  closeModal();
};
</script>
```

위 코드에서 `v-model:visible`은 실제로 다음과 같이 동작합니다:

1. 부모 컴포넌트는 `showViewModal` 값을 `visible` prop으로 자식 컴포넌트에 전달합니다.
2. 자식 컴포넌트는 `visible` prop을 받아서 `Dialog` 컴포넌트의 `visible` prop에 바인딩합니다.
3. 자식 컴포넌트에서 `Dialog`의 `visible` 상태가 변경되면, `update:visible` 이벤트를 발생시켜 부모 컴포넌트로 변경 사항을 전달합니다.
4. 부모 컴포넌트는 `update:visible` 이벤트를 통해 `showViewModal` 값을 업데이트합니다.

이렇게 `v-model:visible`을 사용하면 prop과 이벤트의 양방향 바인딩을 간편하게 구현할 수 있습니다.

---







