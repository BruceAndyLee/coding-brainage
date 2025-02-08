---
Status: In progress
tags:
  - bread
  - frontend
  - vue
---
## Читать, смотреть примеры

[vue plugin in TypeScript](https://q-now.de/2021/10/lets-write-a-vue-js-3-plugin-with-typescript-from-scratch-part-1/)

[vue async created() hook](https://stackoverflow.com/questions/59310803/vue-async-call-in-created-lifecycle-hook)

---

## Полезные плагины

### плагин для отслеживания изменений в форме:

```JavaScript
import { setupDevtoolsPlugin } from "@vue/devtools-api";
import Vue from "vue";

const md5 = require("md5");

const STATE_TYPE = "changerito";
const INSPECTOR_ID = "changerito";

const $changed = Vue.observable({});
const $state = Vue.observable({
  hash: {
    current: "",
    init: ""
  }
});

const $triggers = [];

export const mapChanged = function () {
  let res = {};

  res[`$ch_values`] = function changeMapped() {
    return $changed;
  };
  res[`$ch_values`].ch = true;

  return res;
};

export default {
  install(app, options) {
    Vue.mixin({
      beforeCreate() {
        if (this.$options.changerito) {
          initDevTool(this, {
            triggers: $changed,
            state: $state
          });
        }
      }
    });

    /**
     * @param {string} key
     */
    app.prototype.$ch_get = function (key) {
      if (!key) return $changed["_default"];
      return $changed[key];
    };

    const clearPluginData = function () {
      $state.hash.init = "";
      $state.hash.current = "";

      Object.keys($changed).forEach(key => {
        delete $changed[key];
      });

      $triggers.splice(0, $triggers.length);
    };

    /**
		 * Called by the host-component.
		 * Previous state of the plugin is cleared.
		 * The initial states are remembered
     * @param {Vue Component} component
     * @param {string | array} fields
     * @param {object} options
     */
    app.prototype.$ch_init = function (component, fields, options) {
      
      clearPluginData();

      let tracked = typeof fields === "object" ? fields : [fields];

      let stringified = JSON.stringify(tracked.map((key) => component[key]));

      // Hash the initial value to compare against later
      $state.hash.init = md5(stringified);
      $state.hash.current = md5(stringified);

      // Set computed triggers;
      let keys = Object.keys(options.triggers);

      // Set Default trigger;
      $triggers.push({
        key: "_default",
        boolean: true,
        value: true,
      });

      app.set($changed, "_default", $state.hash.current != $state.hash.init);

      keys.forEach(key => {
        $triggers.push({
          key,
          boolean: typeof options.triggers[key] === "boolean",
          value: options.triggers[key],
        });

        app.set(
          $changed,
          key,
          !!(typeof options.triggers[key] === "boolean"
            ? options.triggers[key]
            : component[options.triggers[key]]) &&
            $state.hash.current != $state.hash.init,
        );
        // Set watcher for linked data
        if (typeof options.triggers[key] !== "boolean") {
          component.$watch(options.triggers[key], value => {
            $changed[key] = !!value && $state.hash.current != $state.hash.init;
          });
        }
      });

      // Set Data Watchers
      data.forEach(field => {
        component.$watch(
          field,
          () => {
            let stringified = JSON.stringify(data.map(x => component[x]));
            $state.hash.current = md5(stringified);

            recalcTriggersState(component);
          },
          { deep: true }
        );
      });
    };
  }
};

/**
 * Called each time a component hosting the changerito plugin
 * gets one of the watched fields updated.
 * 
 * Corresponding trigger in the $changed map is compared against
 * the initial state.
 * 
 * @param {Vue Component} component - the vue component,
 * that uses the changerito plugin
 */
const recalcTriggersState = (component) => {
  $triggers.forEach((trigger) => {
    $changed[trigger.key] =
      (trigger.boolean ? trigger.value : component[trigger.value]) &&
      $state.hash.current != $state.hash.init;
  });
};

const initDevTool = function (app, data) {
	// only initialize the whole thing in local development build
  if (process.env.VUE_APP_DEPLOYMENT_LOCAL) {
    setupDevtoolsPlugin(
      {
        id: "check-changes-in-data",
        label: "changerito",
        app,
        componentStateTypes: [STATE_TYPE]
      },
      (api) => {
				// setting up the components inspector?
        api.addInspector({
          id: INSPECTOR_ID,
          label: "Changerito",
          icon: "multiple_stop",
          treeFilterPlaceholder: "Filter stores..."
        });

				// every time a component within the app's treeview is clicked
				// specify which values should be displayed for the dev
				// to inspect
        api.on.inspectComponent((payload, context) => {
          payload.instanceData.state.push({
            type: STATE_TYPE,
            key: "$triggers",
            value: data.triggers
          });
          payload.instanceData.state.push({
            type: STATE_TYPE,
            key: "$state",
            value: data.state
          });
        });

				// Called when the dev edits the changerito values via the
				// devtools plugin UI.
				// updates fields inside the 'changerito' plugin
        api.on.editInspectorState((payload) => {
					// there may be multiple apps on one page I guess
					if (payload.app !== app || payload.inspectorId !== INSPECTOR_ID) return;
          
          const modulePath = payload.nodeId;
          let path = payload.path;
					
					// include paths stops of the module path
					// if the edited node is not in the root (whatever the hecc that means)
          if (modulePath !== "root") {
            path = [...modulePath.split("/").filter(Boolean), ...path];
          }

					// where the hecc did the store come from?
					// from somewhere in the closure?
          store._withCommit(() => {
            payload.set(store._state.data, path, payload.state.value);
          });
          
        });
      }
    );
  }
};
```

## Vue 3 moments & overview

Notable changes, moments and the general vibe of the new major

- **html-ed vue**
    
    вот так можно запустить вьюшное приложение без webpack-а:
    
    ```JavaScript
    <script type="importmap">
    {
    	"imports": {
    		"vue": "https://unpkg.com/vue@3/dist/vue.esm-browser.js"
    	}
    }
    </script>
    
    <div id="app">{{ message }}</div>
    
    <script type="module">
    // the browser will understand this import
    // statetement thanks to the script above
    import { createApp, ref } from "vue";
    
    createApp({
    	data() {
    		return {
    			message: "heh, wow",
    		}
    	}).mount("\#app")
    </script>
    ```
    
    полифилл для `type="importmap"` [https://github.com/guybedford/es-module-shims](https://github.com/guybedford/es-module-shims)
    
- **app instance config**
    
    В масштабах одного приложения (а их можно больше одного на странице создать) можно настраивать компонент config и записывать глобально доступные компоненты
    
    ```JavaScript
    app.config.errorHandler = (err) => {
      /* handle error */
    }
    
    // in any child component you can use <badge>
    import Badge from "./route/to/badge-component.vue"
    app.component("badge", Badge);
    ```
    
    Можно прятать эти действия внутрь более сложных структур. Например, в логику установки плагина:
    
    ```JavaScript
    // main.js
    import { createApp } from "vue";
    import GlobalComponents from "./plugins/global-components.js"
    
    const app = createApp(App).mount("\#app");
    
    // app.use() calls supplied object's install function
    // passing it the instance as the argument
    app.use(GlobalComponents);
    
    // ./plugins/global-components.js
    import Badge from "./route/to/badge-component.vue"
    const GlobalComponents = {
      install(app) {
    		app.component("badge", Badge);
    	}
    }
    ```
    
- **attributes** ==**binding**== **and** ==**event**== **listeners in template** `**(SIC!)**`
    
    ```JavaScript
    <template>
    	<!-- This is not specific to vue 3 -->
    	<div v-bind="attrsObject"> </div>
    </template>
    
    <script>
    export default {
    	data() {
    		return {
    			attrsObject: {
    				id: "lol",
    				key: "keyek",
    			}
    		}
    	}
    }
    </script>
    
    <!-- Dynamic binding components -->
    <a v-bind:[attributeName]="url"> ... </a>
    <!-- or -->
    <a :[attributeName]="url"> ... </a>
    
    <script>
    export default {
    	data() {
    		return {
    			attributeName: "href",
    		}
    	}
    }
    </script>
    
    // EVENTS listeners can be assigned dynamically too
    <a v-on:[eventName]="doSomething"> ... </a>
    <!-- shorthand -->
    <a @[eventName]="doSomething">
    ```
    
- **correct way to** ==**debounce**== **methods**
    
    ```JavaScript
    export default {
    	methods: {
    		click() { /* some state-reliant code */},
    	},
    	created() {
    		// to grant every reused instance of component its independent debounced function
    		this.debouncedClick = _.debounce(this.click, 300);
    	},
    	unmounted() {
    		// clear up the mess
    		this.debouncedClick.cancel();
    	},
    }
    ```
    
- **mouse event modifiers**
    
    ```HTML
    <!-- only trigger if the div clicked (not child) -->
    <div @click.self></div>
    
    <!-- event targetting an inner element will first go through this handler -->
    <div @click.capture="processClick"></div>
    
    <!-- won't be triggered more than once -->
    <a @click.once="processClick">
    
    <!-- scroll - default on-scroll behaviour - will be triggered -->
    <!-- instantly, instead of waiting for the onScroll to complete -->
    <div @scroll.passive="onScroll"></div>
    
    <!-- mouse button specifiers !!!! -->
    <div @click.left/middle/right="doMouseButtonSpecificStuff"></div>
    ```
    
- **keyboard event modifiers (key aliases)**
    
    ```HTML
    <!-- Actions and navigation -->
    <div @keyup.enter="onSubmit"></div>
    <div @keyup.page-down="onPageDown"></div>
    <div @keyup.tab="onTab"></div>
    <div @keyup.delete="onDeleteOrEscape"></div>
    <div @keyup.space="onSpace"></div>
    <div @keyup.up="onArrowUp"></div>
    <div @keyup.down="onArrowDown"></div>
    <div @keyup.left="onArrowLeft"></div>
    <div @keyup.right="onArrowRight"></div>
    
    <!-- System key modifiers -->
    <div @keyup.alt.enter="onAltEnter"></div>
    <div @click.ctrl="onCtrlClick"></div>
    <div @keyup.shift.enter="onShiftEnter"></div>
    <div @click.meta.enter="onMetaClick"></div>
    
    <!-- .exact allows control of the exact combination -->
    <!-- the below event won't get to the event listener -->
    <!-- if at the time of clicking there are other keys than ctrl -->
    <div @click.ctrl.exact="onCtrlClick"></div>
    ```
    
- **form** ==**sync**==**-ing modifiers and stuff**
    
    ```HTML
    <!-- calls the handler function on CHANGE event instead of INPUT -->
    <input v-model.lazy="boundVariable" />
    
    <!-- to typecast value to integer if parsable -->
    <input v-model.number="boundVariable" />
    
    <!-- to automatically trim the string input -->
    <input v-model.trim="boundVariable" />
    ```
    
- **parent-child tight coupling**
    
    ```JavaScript
    <Child ref="child"/>
    
    <script>
    export default {
    	mounted() {
    		// is the same as 'this' whithin the Child component itself
    		// so, all the methods and data are fully accessible
    		this.$refs.child;
    	},
    }
    </script>
    
    // in Child.vue using the 'expose' option to limit parent's access
    export default {
    	expose: ["publicData", "publicMethod"],
    	data() {
    		return {
    			publicData: "Musical instruments",
    			privateData: "Some hentai shish",
    		},
    	},
    	methods: {
    		publicMethod() { /* playing the guitar */ },
    		privateMethod() { /* treating oneself to some deplorable footage */}
    	}
    }
    ```
    
- ==**component**==**’s events declaration**
    
    ```JavaScript
    export default {
    	emits: {
    
    		// no validation.
    		// might as well just dont mention click event here
    		click: null,
    
    		// called after the event was emitted
    		// but before it is passed up to the parent
    		submit(payload) {
    			// return true or false to indicate
    			// if validation passed or failed respecctively
    		}
    	},
    	methods: {
    		this.$emit("submit", someQuestionableData);
    	}
    }
    ```
    
- **components two-way data flow** (==**v-model**== ==**mark III**==)
    
    ```JavaScript
    // when used on a custom component, v-model acts differently
    <CustomInput v-model="searchText"/>
    // the above is equivalent to
    <CustomInput
    	:modelValue="searchText"
    	@update:modelValue="newVal => searchText = newVal"
    />
    ```
    
    ```JavaScript
    // using a v-model modificator: child should expect a specific prop
    // or many props even
    <ChildComponent
    	v-model:title="parentTitleVariable"
    	v-model:author="parentAuthorVariable"
    />
    
    // in Child.vue:
    export default {
    	props: ["title"],
    	emits: {
    		"update:title"(payload) {
    			// event payload validation here
    		},
    		"update:author"(payload) {
    			// author payload validation
    		}
    	},
    	methods: {
    		onTitleUpdate() {
    			this.$emit("update:title", localTitleVariable);
    		},
    		onAuthorUpdate() {
    			this.$emit("update:author", localAuthorVariable)
    		}
    	}
    }
    ```
    
- ==**Inject / Provide**==
    
      
    
- ==**Asynchronous**== **components**
    
    Компоненты, которые не встроятся в наше приложение (или в компонент, в котором они зарегистрированы локально), пока не они не понадобятся явно
    
    ```JavaScript
    import { defineAsyncComponent } from "vue";
    
    const AsyncComp = defineAsyncComponent(
    	() => import("./components/AsyncComponent.vue")
    )
    
    app.component("async-component", AsyncComp)
    ```
    
    Пример локальной регистрации асинхронного компонента
    
    ```JavaScript
    import { defineAsyncComponent } from "vue"'
    
    createApp({
    	components: {
    		AsyncComponent: defineAsyncComponent(
    			() => import("./components/AsyncComponent.vue")
    		)
    	}
    })
    ```
    
- ==**suspence**== **component**
    
    Уницифированная семантическая альтернатива для рендеринга частей страницы в момент загрузки:
    
    ```JavaScript
    <template>
    	<suspense>
    		<template \#default>
    			<todo-list />
    		</template>
    		<template \#fallback>
    			<div>Loading...</>
    		</template>
    	</suspense>
    </template>
    
    <script>
    export default {
    	components: defineAsyncComponent(() => import("./TodoList.vue"))
    }
    </script>
    ```
    
    Асинхронный потомок может находится на любом уровне вложенности под компонентом ==**suspense**==.
    
    Асинхронность потомка необязана задаваться ленивой-загрузкой. Она так же может быть описана менее явно, если `setup` функция этого потомка асинхронная.
    
    Дочерний компонент может отказаться от управляемости компонентом ==**suspense**==, для этого его опция suspensible переводится в false
    
- **ref**
    
    ```JavaScript
    <template>
    	<input ref="input">
    </template>
    
    <script>
    export default {
    	mounted() {
    		this.focusInput();
    	},
    	methods: {
    		focusInput() {
    			this.$refs.input.focus();
    		}
    	}
    }
    </script>
    ```
    
- ==**v-once**==
    
    Можно установить на корневом элементе компонента директиву `v-once`
    
    Чтобы вычислить его один раз и закешировать и больше НИКОГДА не вычислять.
    
    ```JavaScript
    app.component('terms-of-service', {
    	template: `
    		<div v-once>
    			<h1>Условия пользования</h1>
    			<p>...Очень много статического содержимого...</p>
    		</div>
    	`
    })
    ```
    

остановился вот тут [https://v3.ru.vuejs.org/ru/guide/migration/mount-changes.html#обзор](https://v3.ru.vuejs.org/ru/guide/migration/mount-changes.html#%D0%BE%D0%B1%D0%B7%D0%BE%D1%80)

portal [https://vueschool.io/articles/vuejs-tutorials/portal-a-new-feature-in-vue-3/](https://vueschool.io/articles/vuejs-tutorials/portal-a-new-feature-in-vue-3/)

## Rendering (!!!!)

[https://vuejs.org/guide/extras/rendering-mechanism.html](https://vuejs.org/guide/extras/rendering-mechanism.html)

## Composable

### Example + basic logic and reasoning

Общее понятие, обозначающее импортируемая модульную логику, которая использует состояние.

Пример: mouse-tracker

```JavaScript
<template>Mouse position is at: {{ x }}, {{ y }}</template>

<script setup>
import { ref, onMounted, onUnmounted } from "vue";

const x = ref(0);
const y = ref(0);

function update(event) {
	x.value = event.pageX;
	y.value = event.pageY;
}

onMounted(() => window.addEventListener("mousemove", update))
onUnmounted(() => window.removeEventListener("mousemove", update))
</script>
```

Теперь этот код можно сделать переиспользуемым, завернув его в функцию, которую достаточно вызвать где-то еще, чтобы она в рамках другого кода инициализировала точно такой же стейт с такой же логикой поведения и реагирования на действия юзера:

```JavaScript
import { ref, onMounted, onUnmounted } from "vue";

export function useMouse() {
	const x = ref(0);
	const y = ref(0);
	
	function update(event) {
		x.value = event.pageX;
		y.value = event.pageY;
	}
	
	onMounted(() => window.addEventListener("mousemove", update))
	onUnmounted(() => window.removeEventListener("mousemove", update))

	// the function returns reactively updated state (two variables in this case)
	return { x, y }
}
```

Использование этого функционала в компоненте:

```JavaScript
<template>
	Mouse is at: {{ x }}, {{ y }}
</template>

<script setup>
import { useMouse } from "./use-mouse.js";

// we use const here since the variables themselves are
// objects with hidden properties for value access and control
const { x, y } = useMouse();

</script>
```

### Nesting

Теперь можно на основе **composable** единиц собирать более сложные **composable**-модули, руководствуясь той же логикой, что и про построении многокомпонентных приложений: можно настраивать сложную модульную логику используя мало кода.

1. Напишем composable для более лаконичного подключения слушателей:
    
    ```JavaScript
    import { onMounted, onUnmounted } from "vue";
    
    /**
     * @param {String} event - name of the event
     * @param {VNode} target - most likely window or some other DOM node
     * @param {Function} callback - user's callback per event emitted
     */
    export function useEventListener(event, target, callback) {
    		onMounted(() => target.addEventListener(event, callback))
    		onUnmounted(() => target.removeEventListener(event, callback))
    }
    ```
    
2. Напишем ==**composable**==, добавляющий вычисление позиции курсора на странице с использованием `useEventListener`
    
    ```JavaScript
    import { ref } from "vue"
    import { useEventListener } from "./use/event-listener.js"
    
    export function useMouse() {
    	const x = ref(0)
    	const y = ref(0)
    
    	function updateMousePos(event) {
    		x.value = event.pageX
    		y.value = event.pageY
    	}
    
    	useEventListener("mouseover", window, updateMousePos)
    
    	return { x, y }
    }
    ```
    

### Asynchronous state

Вынесем логику с асинхронным получением данных и обновлением состояния запроса (`data/loading/error`) в ==**composable.**==

```JavaScript
import { ref } from "vue";

export function useFetch(url, headers) {
	const data = ref(null);
	const error = ref(null);

	fetch(url, headers)
		.then((res) => res.json())
		.then((res) => data.vaule = res)
		.catch((err) => error.value = err)

	return { data, error }
}
```

Теперь можно его использовать в коде компонента:

```JavaScript
<srcipt setup>
import { useFetch } from "./use/fetch.js"

const { data, error } = useFetch();
</script>

<template>
	<JSONComponent  v-if="data" :value="data">
	<ErrorComponent v-else-if="error" :value="error" />
	<div v-else>
		Loading...
	</div>
</template>
```

Добавим **re-fetch** в наш ==**composable**==**:**

```JavaScript
import { ref, isRef, watchEffect } from "vue";

export function useFetch(url, headers) {
	const data = ref(null);
	const error = ref(null);

	function doFetch() {

		data.value = null;
		error.value = null;
		
		// calling unref returns the value of a ref
		// or the thing itself if it's not a ref
		fetch((unref(url), headers)
			.then((res) => res.json())
			.then((res) => data.vaule = res)
			.catch((err) => error.value = err)
	}

	// url can be a static string or a Ref<string>
	// tracked bu the state itself
	if (isRef(url)) {
		// telling vue to track the URL ref
    // and execute our method on update
		watchEffect(doFetch)
	} else {
		doFetch()
	}

	return { data, error }
}
```

### Reactivity and ref unwrapping

Стандартный подход: возвращать из инициализатора ==**composable**== простой объект, состоящий из ref-ов, чтобы можно было использовать деструктуризацию в вызывающем коде и не терять реактивность получаемых значений

```JavaScript
export function useMouse() {
	const x = ref(null)
	const y = ref(null)
	// some logic skipped for the sake of example

	return { x, y }
}

// x and y are refs
const { x, y } = useMouse()
```

Чтобы из таким же образом написанного ==**composable**== получить реактивный объект без деструктуризации, необходимо использовать `reactive()`

```JavaScript
export function useMouse() {
	const x = ref(null)
	const y = ref(null)
	// some logic skipped for the sake of example

	return { x, y }
}

const mouseComposable = reactive(useMouse())
```

библиотека ==**composable**==-ов [https://vueuse.org/](https://vueuse.org/)

## Composition API

### setup() ← вернись сюда, как следующие три раздела закончишь

Скрипт, который лежит под **Options API** и который можно написать самостоятельно, создал реактивные свойства компонента.

```JavaScript
<script>
import { ref } from "vue"

export default {
	props: {
		title: String,
	},
	// does NOT have access to 'this'
	// props - object with reactively supported fields
  // context - safely desctructible object of (un-reactive) component config fields
	setup(props, /** context */ { attrs, slots, emits, expose }) {
	
		const counter = ref(0)
		const title_ref = ref(0)
		
		// to retain reactivity of the each of the props
		// access it via dot-notaion
		title_res.value = props.title

		// exposing a select subset of local refs to the parent
		// so that they can access it via
		//  this.$ref.child_ref_name.publicCoutner
		expose({ publicCounter: counter }) 
		
		// returns refs visible for template injection.
		// 
		// Refs are shallow unwrapped (.value) when used
		// in template (and in component's methods too)
		return {
			counter,
		}
	}
}
</script>
```

## Reactivity

- **Basics**
    - `reactive()` - работает с объектами (в т.ч. с массивами), поддерживает актуальное состояние объекта внутри, пересчитывает зависимости при обновлении внутренностей объекта.
        
        При передаче содержимого в функцию или при сохранении его в переменную - реактивность теряется.
        
        ```JavaScript
        const reactive_state = reactive({ count: 0 });
        
        const { count: not_reactive } = reactive_state;
        
        functionOnCounter(reactive_state) // receives a plain number. Nothing to track the changes with
        ```
        
    - `ref()` - создает реактивные значения любых типов, не только объекты. Реактивность обеспечивается тем, что значение хранится внутри объекта, то есть, можно запомнить ссылку на объект и перехватить операции чтения и перезаписи значения с помощью `Proxy`
        
        Логично, что реактивность не потеряется при деструктурировании объектов, содержащих поля-рефы или при передаче полей-рефов объекта в функции (Ведь каждый реф - это объект с перехваченными операциями чтения и перезаписи).
        
        **СУПЕР ВАЖНО:**
        
        При использовании в шаблоне документа рефы разворачиваются самостоятельно, но только в том случае, если они являются переменными верхнего уровня:
        
        ```JavaScript
        const lol = ref(0);
        lol.value += 1;
        
        // при использовании lol в шаблоне достаточно просто написать
        <div>{{ lol }}</div> // и всё будет работать реактивно
        ```
        
        НО если реф лежит в объекте под каким-то ключом, то в шаблоне надо явно обращаться к его `.value`
        
        ```JavaScript
        const map_of_lol = {
        	lol1: ref(0),
        };
        map_of_lol.lol1.value += 1;
        
        // если не обратиться к value то vue вызовет toString (или valueOf?) обычного объекта
        // и результат будет кринжовым
        <div>{{ map_of_lol.lol1 + 1 }}</div> // отобразится [object Object]1
        
        // при использовании в шаблоне надо обращаться внутрь рефа явным образом
        <div>{{ map_of_lol.lol1.value + 1 }}</div> // отобразится 2
        // объект map_of_lol можно деструктурировать, сделав lol1 переменной верхнего уровня, тогда всё заработает
        ```
        
- **Ref unwrapping in reactive objects**
    
    Чтение и запись в реф, который находится внутри объекта созданного функцией `reactive()` не требует обращения к `.value` этого рефа, потому что реактивная обертка сама справляется с этой задачей. При этом глубина не имеет значения:
    
    ```JavaScript
    const count = ref(0);
    const counterState = reactive({
      numbers: { count },
    });
    
    // вызов функции increment будет работать корректно 
    function increment() {
      counterState.numbers.count++;
    }
    // взаимозаменяемый вариант
    function increment() {
    	count.value++;
    }
    ```
    
    При замене одного рефа другим внутри реактивного объекта, первый реф потеряет связь с объектом, но останется работоспособен
    
    ```JavaScript
    const otherCount = ref(2)
    
    counterState.numbers.count = otherCount
    console.log(counterState.numbers.count) // 2
    // original ref is now disconnected from state.count
    console.log(count.value) // 0
    ```
    
- [No] **Ref unwrapping in reactive Arrays and Collections**
    
    При обращении к рефам внутри массивов и коллекций разворачивания не происходит, надо явно обращаться к `.value`
    
    ```JavaScript
    const gtaGames = reactive([ref("San Andreas")]);
    gtaGames[0].value // не будет работать unless мы укажем .value
    
    const nfsGames = reactive(new Map([["Hot Pursuit", ref("So effing stoopid")]]));
    nfsGames.get("Hot Pursuit").value // та ж тема
    ```
    

### Reactivity: Core

[https://vuejs.org/api/reactivity-core.html](https://vuejs.org/api/reactivity-core.html)

### Reactivity: Utilities

[https://vuejs.org/api/reactivity-utilities.html](https://vuejs.org/api/reactivity-utilities.html)

### Reactivity: Advanced

## Inject

---

## Community utils

- [vue awesome](https://github.com/vuejs/awesome-vue)

## Bugs and solutions

node sass

```Bash
yarn remove node-sass
yarn add -D sass
```

```Bash
vue 2 работает с sass-loader 10 версии, но не выше
```

[vuetify] Multiple instances of vue detected

```Bash
yarn install
yarn upgrade
```

## Vue apollo

[TESTING VUE + APOLLO](https://dev.to/n_tepluhina/testing-vue-apollo-2020-edition-2l2p)

### Apollo Client

Здесь главное написать функцию для генерации Apollo-провайдера в приложение:

```JavaScript
// пример настроек для создания Apollo-клиента
const defaultOptions = {
  httpEndpoint: process.env.VUE_APP_GRAPHQL_HTTP,
  tokenName: AUTH_TOKEN,
  persisting: false,
  websocketsOnly: false,
  ssr: false,
  link: from([hasura_link, error_link, split_link]),

  cache: new InMemoryCache(),
};

export function createProvider(options = {}) {
  // Создание Apollo-клиента с обозначенными выше defaultOptions
  const apolloClient = new ApolloClient({
    ...defaultOptions,
    ...options,
  });

  // Create vue apollo provider
  const apolloProvider = new VueApollo({
    defaultClient: apolloClient,
  });
  return apolloProvider;
}
```

Далее в main.js при создании приложения можно использовать провайдер:

```JavaScript
import { createProvider } from "./vue-apollo-setup.js"

export const vm = new Vue({
	...,
	router,
	apolloProvider: createProvider(),
	...
})
```

---

### Apollo link

Используется для добавления дополнительной логики в запросы к GQL серверу.

![[Screenshot_2022-03-28_at_13.50.25 1.png|Screenshot_2022-03-28_at_13.50.25 1.png]]

На примере выше компонент программиста посылает запрос к graphql серверу через apollo. Каждый Link - это некий узел в цепочке, который либо:

- меняет запрос: добавляет заголовок, обрабатывает данные запроса
- производит дополнительные действия в системе: логгирует что-то на хосте.

Пример из жизни: добавление заголовка **x-hasura-role** в запрос в зависимости от того, с какой ролью текущий пользователь может получить доступ к целевому ресурсу запроса.

По умолчанию используется `HttpLink`. Он же всегда расположен в конце определяемой юзером цепочки.

Для определения кастомной линки надо использовать конструктор `ApolloClient`

Пример с использованием одного дополнительного объекта `Link` в цепи:

```JavaScript
import { ApolloClient, InMemoryCache, HttpLink, from } from "@apollo/client";
import { onError } from "@apollo/client/link/error";

const httpLink = new HttpLink({
  uri: "http://localhost:4000/graphql"
});

const errorLink = onError(({ graphQLErrors, networkError }) => {
  if (graphQLErrors)
    graphQLErrors.forEach(({ message, locations, path }) =>
      console.log(
        `[GraphQL error]: Message: ${message}, Location: ${locations}, Path: ${path}`,
      ),
    );

  if (networkError) console.log(`[Network error]: ${networkError}`);
});

/** 
 * вместо указания uri в настройках ApolloClient множно передать
 * цепочку линков, uri в таком случае должен быть указан в последнем
 * линке, который обязательно является объектом HttpLink
 */ 
const client = new ApolloClient({
  /** 
	 * функция `from` формирует цепочку объектов Link (link chain)
	 * из переданного массива в ручную созданных линков
	 */ 
  link: from([errorLink, httpLink]),
  cache: new InMemoryCache()
});
```

Это был пример использования встроенного подвида объекта класса ApolloLink

Но можно создавать и полностью собственный вид линки. При создании своей линки надо указать метод-обработчик запроса.

```JavaScript
import { ApolloLink } from '@apollo/client';

const timeStartLink = new ApolloLink((operation, forward) => {
  operation.setContext({ start: new Date() });
  // вызов метода forward отправляет запрос в следующию линку в цепи
  return forward(operation);
});
```

объект `operation: Operation`, передаваемый в кастомный request-handler:

```JavaScript
{
	// объект класса DocumentNode (parsed GraphQL operation)
  // описывающий gql операцию
	query,
	// json-объект переменных gql запроса
	variables,
	// Строчное имя запроса или null, если имя не указано программистом
	operationName,
	// хэш мап расширений, отправляемых на сервер
  extensions,
  // Функция возвращает контекст, закрепленный за запросом.
  // Контекст может использоваться линками,
	// чтобы выполнять нужные по логике действия.
	getContext,
	// Функция принимающая либо новый json-объект контекста,
	// или функцию, которая обрабатывает предыдущее состояние контекста
	// и возвращает новое. Результат коллбека будет использован для обновления
	// полей в предыдущем состоянии контекста 
	setContext,
}
```

Функция forward должна быть вызвана в конце обработчика кастомной `ApolloLink`.

Ее результат должен быть возвращён или непосредственно, или в обработанном виде, например, можно сделать

```JavaScript
return forward(operation).map(
	data => {
		...
		return result;
	}
)
```

Следующая линка в цепи получит на вход результат выполнения коллбека функции `map`

Внутрь этого коллбека можно запихнуть логику, которая не меняет `data`, но производит вспомогательные действия, например, смотрит в `operation.getContext()` и логгирует данные.

### Аддитивное и направленное объединение

Несколько линков можно объединить в одну, если логика подразумевает, что они должны в любом случае использоваться в одном порядке:

```JavaScript
import { from, HttpLink } from '@apollo/client';
import { RetryLink } from '@apollo/client/link/retry';
import MyAuthLink from '../auth';

const additiveLink = from([
  new RetryLink(),
  new MyAuthLink(),
  new HttpLink({ uri: 'http://localhost:4000/graphql' })
]);
```

В случае, когда доступны разные сценарии, можно использовать бинарное ветвление:  
У объекта  
`ApolloLink` есть метод `split`, возвращающий одну “агрегированную” линку, как и в случае с аддитивным объединением.

Этому методу доступен объект `operation`, следовательно, на основании любых значений внутри этого объекта можно делать вывод, к какому линку переходить дальше:

```JavaScript
import { ApolloLink, HttpLink, InMemoryCache } from '@apollo/client';
import { RetryLink } from '@apollo/client/link/retry';

const directional_link = new RetryLink().split(
	// test: тестовая функция, возвращающая Boolean
  (operation) => operation.getContext().version === 1,
	// left: следующая линка, если test() вернет true
  new HttpLink({ uri: "http://localhost:4000/v1/graphql" }),
	// right: следующая линка, если test() вернет false
  new HttpLink({ uri: "http://localhost:4000/v2/graphql" })
);

// использовать этот линк можно как и в случаях с
// простыми/необъединенными линками
new apolloClient({
	cache: new InMemoryCache(),
	link: from([some_other_link, directional_link])
})
```

---

### In-memory Cache

### Обзор

Последовательность действий при первом запросе объекта `Book`

![[Screenshot_2022-03-29_at_12.24.57 1.png|Screenshot_2022-03-29_at_12.24.57 1.png]]

После того как объект с указанным айдишником был получен, он кешируется, и новый запрос не будет отправлен **GraphQL** серверу

![[Screenshot_2022-03-29_at_12.26.45 1.png|Screenshot_2022-03-29_at_12.26.45 1.png]]

Кэш по форме плоский, но поддерживает вложенность объектов.

Айдишник (`cache-id`) генерится из полей `__typename` и `id` (или `_id`). Вот здесь про то, как можно кастомизировать айдишник: [https://www.apollographql.com/docs/react/caching/cache-configuration/#customizing-cache-ids](https://www.apollographql.com/docs/react/caching/cache-configuration/#customizing-cache-ids)

После добавления всех объектов в кеш, вложенные объекты заменяются на соответствующие айдишники.

```JavaScript
// Вот такой объект
{
	__typename: "guitar",
	id: "prs_santana_signature",
	owner: {
		__typename: "living_legend",
		id: "Carlos Santana",
	},
}

// превращается в две записи, в одной из записей появляется ссылка
{
	__typename: "Guitar",
	id: "prs_santana_signature",
	owner: {
		__ref: "LivingLegend:CarlosSantana",
	},
},
{
	__typename: "LivingLegend",
	id: "CarlosSantana",
}
```

Такой прием нормализации помогает избежать дубликации данных в тех случаях, когда два объекта ссылаются на один и тот же (вложенный по отношению к ним) объект.

(Уплощения не происходит, если в объекте нет поля `id/_id` и если при этом не написана кастомная генерация айдишника для такого типа объектов)

Если в ответе на какой-либо из запросов содержится объект, чей `cache-id` существует в кэше, то запись в кеше обновляется следующим образом:

- пересекающиеся поля обновляются до состояния полученного объекта
- все поля, которые встречаются только в закешированном или только во вновь пришедшем объекте, **сохраняются**.

### Настройка объекта класса `InMemoryCache`

### Чтение и запись данных

- `readQuery` `writeQuery` `updateQuery` - использование gql синтаксиса и переменных для обновления данных в кеше.
    
    ```JavaScript
    const READ_GUITAR_GQL = gql`
      query ReadGuitar($id: ID!) {
        guitar(id: $id) {
          id
          owner_id
        }
      }
    `;
    
    // получить объект типа guitar с полем id === 5
    const { guitar } = client.readQuery({
      query: READ_GUITAR_GQL,
    	// как и в случае с отправлением мутаций или запроса
    	// можно предоставить объект с переменными
      variables: { 
        id: 5,
      },
    });
    ```
    
    изменять полученный объект не имеет смысла с точки зрения хороших программистских практик. Логика такая же, как и при использовании библиотеки ==**vuex**==. Изменение данных, доступных только локально или присылаемых удаленным сервером, должно происходить централизованно. То есть, с помощью `client.writeQuery`.
    
    ```JavaScript
    client.writeQuery({
      query: gql`
        query WriteGuitar($id: Int!) {
          guitar(id: $id) {
            id
            owner_id
          }
        }`,
      data: { // Contains the data to write
        guitar: {
          __typename: 'Guitar',
          id: 5,
          owner_id: 4,
        },
      },
      variables: {
        id: 5
      }
    });
    ```
    
    хорошая тема, надо только помнить, что никто не будет проверять данные на соответствие gql-схеме.
    
- `readFragment` `writeFragment` `updateFragment` - обновление информации в кеше без использования целого запроса с мутацией или запросом и объектом с переменными. Объект в кэше находится по айдишнику (aka `` `${obj.__typename}${obj.id}` ``)
- `cache.modify` - обновление объекта кеша без использования gql синтаксиса

---

### Composable utilities

[https://v4.apollo.vuejs.org/guide-composable/setup.html#_1-install-vue-apollo-composable](https://v4.apollo.vuejs.org/guide-composable/setup.html#_1-install-vue-apollo-composable)

### Zen-observable

rxjs-like thingie

[https://www.npmjs.com/package/zen-observable](https://www.npmjs.com/package/zen-observable)

## Vuelidate

[https://vuelidate-next.netlify.app/#installation](https://vuelidate-next.netlify.app/#installation)

- vue.ts