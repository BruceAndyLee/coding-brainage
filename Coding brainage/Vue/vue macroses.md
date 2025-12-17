
`defineModel` - макрос для создания двунаправленной реактивной связи родителя и потомка БЕЗ явной декларации пропов и эвентов (кринж).

Пример: дочерний компонент с инпутом + родительский компонент, монтирующий несколько таких дочерних компонентов-редакторов. 

```html
<script setup>
import { defineModel } from 'vue';

const tagModel = defineModel();
</script>

<template>
  <div>
    <input v-model="tagModel"></input>
    <strong>{{tagModel}}</strong>
  </div>
</template>
```

Родительский компонент объединяет несколько инпутов для указания тега сущности:
```html
<script setup>
import { ref, watch } from "vue";
import Tagger from "./Tagger.vue";

const entity_tags = ref(["", "", ""]);
</script>

<template>
	<div v-for="tag, i in entity_tags">
		<Tagger v-model="entity_tags[i]" />
		<div>Entity #{{i}} tag: <strong>{{tag}}</strong> </div>
	</div>
</template>

```

Результат работы: значения внутри массива `entity_tags` обновляются реактивно и отображаются в РОДИТЕЛЬСКОМ компоненте, когда пользователь что-то вводит в инпут в дочернем компоненте.

---

`defineProps`


---

`defineEvents`


