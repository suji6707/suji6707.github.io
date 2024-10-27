+++
title = 'Vue'
date = 2024-10-01T00:20:22+09:00
draft = true
+++
## Vue 이벤트 핸들링

### Ref
툴팁 다른곳 클릭하면 없어지게 하기

- DOM 요소에 직접 접근해야 할 때 ref
- v-for에서 ref를 쓰면 배열에 대해 알아서 각각의 ref를 생성해줌 -> 인덱스로 비교. 
- tables 구조체에 넣으면 연동이 안되고, 별도의 구조체 ref가 필요했던 것. (Vue 에서 HTML은 생각처럼 작동하지 않음)
```js
const tableRefs = ref<HTMLElement[]>([]);

const handleOutsideClick = (e: MouseEvent) => {
    console.log(e.target);
    tableRefs.value.forEach((tableRef, index) => {
        if (tableRef && !tableRef.contains(e.target as Node)) {
            tables.value[index].showTooltip = false;
        }
    });
};
```

---
### CodeMirror state update

🍎🍎🍎 중요한 건 래핑된 라이브러리의 셋팅부터 잘 봐야한다는 것.
기본적으로 @codemirror를 쓰는거지만
나는 그걸 래핑한 vue-codemirror 라이브러리를 쓰고있음.
래핑하기 전도 중요하지만 래핑한 후의 쓰임새를 잘 볼 것.
어떤 이벤트를 쓰는지 등.

npm 문서보면 `@codemirror/state`를 dependencies에 추가하라고 되어있고
실제로 업데이트시 @ready 라는 이벤트명이 내부적으로 동작하도록 되어있는데
이 `handleReady`가 받는 payload 타입이 view, state 로 구성된 객체.
state가 `EditorState` 인것.

```js
import { EditorView } from '@codemirror/view';
import { EditorState } from '@codemirror/state';

const modelValue = defineModel<string>();
const extensions = [sql({ dialect: MySQL })];

const emit = defineEmits(['update:modelValue']);

const view = shallowRef();
const handleReady = (payload: {
    view: EditorView;
    state: EditorState;
    container: HTMLDivElement;
}) => {
    view.value = payload.view;
};
const codeMirrorInstance = ref<EditorView | null>(null);

const insertTextAtCursor = (text: string) => {
    const cm = view.value;  
    if (cm) {
        const transaction = cm.state.update({
            changes: { from: cm.state.selection.main.head, insert: text }
        });
        cm.dispatch(transaction);
        emit('update:modelValue', cm.state.doc.toString());
    }
};

defineExpose({
    insertTextAtCursor
});

<template>
  <div class="query-window">
    <codemirror 
      ref="codeMirrorInstance"
      v-model="modelValue"
      placeholder="Code goes here..."
      style="width: 100%; height: 100%;text-align: left; font-size: 15px;"
      :autofocus="true"
      :indent-with-tab="true"
      :tab-size="2"
      :extensions="extensions"
      @ready="handleReady"
    />
  </div>
</template>
```

(참고)
### 프로토타입 체인 이해하기

JavaScript 객체는 프로토타입을 통해 다른 객체로부터 상속받을 수 있습니다. `[[Prototype]]`은 객체가 다른 객체로부터 상속받은 프로퍼티와 메서드를 포함하는 객체를 참조합니다.

### 예시

```javascript
const obj = {
    name: 'John',
    greet() {
        console.log('Hello');
    }
};

console.log(obj);
```

위의 예시에서 `obj` 객체를 콘솔에 출력하면 다음과 같은 구조를 볼 수 있습니다:

```javascript
{
    name: 'John',
    greet: function() { ... },
    [[Prototype]]: Object
}
```

### `[[Prototype]]` 내부 내용

`[[Prototype]]` 내부에는 프로퍼티와 메서드가 모두 포함될 수 있습니다. 예를 들어, `Object`의 프로토타입에는 `toString`, `hasOwnProperty`와 같은 메서드가 있지만, 일반적인 프로퍼티도 포함될 수 있습니다.


---
### 🔥부모-자식 양방향 바인딩
v-model만으로 안되는 경우가 있음.

자식 -> 부모로 emit할 때는
defineEmit으로 이벤트이름을 등록해주고,
자식 컴포넌트 이벤트 발생시 함수안에서 emit으로 데이터를 전달한다.

부모에선 그 데이터를 받으려면 
반드시 @이벤트이름="$event~" 로 이벤트에 담긴 데이터($event)로 함수를 정의해야 한다.

```js
// 자식 컴포넌트 ==========================
const emit = defineEmits(['update:modelValue', 'saveView']);

const saveView = () => {
    emit('saveView', viewName.value);
    emit('update:modelValue', false);
};

	<Button
		type="button"
		label="Save"
		@click="saveView"
	/>  

// 부모 컴포넌트 ==========================
const viewName = ref('');

watch(viewName, (newVal) => {
    if (newVal) {
        console.log('viewName', viewName.value);
        viewName.value = '';
    }
});

	<CreateViewModal
		v-model="showViewModal"
		@saveView="viewName = $event"
	/>

```

부모 -> 자식으로 데이터를 넘겨줄 때는 
💎 자식에서 `defineProps`를 통해 값을 받는다.
(부모는 v-model=`data` 로 넘겨주면 끝인데 자식에서 좀 귀찮은듯?)

(참고)
Vue 3의 v-model 디렉티브는 자동으로 modelValue라는 prop을 사용하여 값을 전달하고, update:modelValue 이벤트를 통해 값을 업데이트합니다. 이는 Vue 3에서 v-model의 기본 동작 방식입니다. 이 동작을 이해하고 활용하는 방법을 설명하겠습니다.


```js
// 자식 컴포넌트 ==========================
const props = defineProps<{
  modelValue: boolean;
}>();

const emit = defineEmits(['🍎update:modelValue']);

const localVisible = ref(props.modelValue);
watch(() => props.modelValue, (newVal) => {
    localVisible.value = newVal;
});

    <Dialog
      v-model:visible="localVisible"
      modal
      header="Create View"
      :style="dialogStyle"
      @update:visible="updateVisible"
    >

// 부모 컴포넌트 ==========================
const showViewModal = ref(false);

<button
	class="exec-button"
	@click="showViewModal = true"
>
	<CreateViewModal
		v-model="showViewModal"
	/>
```