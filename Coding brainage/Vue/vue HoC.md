Пример задачи, которой этот паттерн был бы удобен:

В компоненте для редактирования некоторой сущности создана форма
```html
<template>
	<v-form @submit="on_submit">
		<!-- some intputs here -->
		<VBtnPrimary type="submit" >
			{{ locale.save_button_text }}
		</VBtnPrimary>
	</v-form>
</template>

<script setup>
const form_data = ref({});
function on_submit() {
	somePinia
		.dispatch_effect_like_save_or_something(form_data.value)
		.then(on_success)
		.catch(on_error)
}
</script>
```

Изменение в кодовой базе:
- доступность этой формы нужно ограничить: сохранить изменение можно только в том случае, если статус некоторого внешнего сервиса `"on"`.
- это же ограничение нужно навесить на несколько других кнопок на других страницах из тех же соображений. Внешний сервис должен иметь статус `"on"`, чтобы включить сразу несколько как бы несвязанных друг с другом фичей
- помимо ограничения доступа к кнопке необходимо поддерживать два способа расширить логику кнопки:
	- перехватить нажатие и передать управление родительскому коду - например, чтобы показать запрещающий диалог
	- перехватить нажатие и на уровне родительского кода запустить подготовительные эффекты

Как это можно сделать без дополнительного HoC компонента:
- лезем в код каждого компонента с кнопкой, логику которой надо расширить и прописывать в `onClick` функцию условие, которое проверяет внешнюю зависимость и запускает эффект

Как будет выглядеть обновленный код из примера выше:
```html
<template>
	<v-form @submit="on_submit">
		<!-- some intputs here -->
		<VBtnPrimary type="submit" >
			{{ locale.save_button_text }}
		</VBtnPrimary>
	</v-form>
</template>

<script setup>
const form_data = ref({});
const dangerous_submit_confirm_dialog = ref(false);

function submit() {
	somePinia
		.dispatch_effect_like_save_or_something(form_data.value)
		.then(on_success)
		.catch(on_error)
}

function on_submit() {
	if (submit_prohibited_by_external_service_state) {
		// submit откладывается до того момента,
		// пока пользователь не подтвердит сво подозрительное действие
		dangerous_submit_confirm_dialog.value = true;
		return;
	}
	submit();
}

function on_dangerous_submit_user_response(confirmed) {
	if (!confirmed) {
		dangerous_submit_confirm_dialog.value = false;
		return;
	}
	submit();
	dangerous_submit_confirm_dialog.value = false;
}

</script>
```

Предлагаемый вариант с HoC:

```js
export default {
	name: "ClickGuarded",
	// functional component would rerender every time,
	// but having a separate function helps 
	setup(props, { slots }) {
		return () => h(
			
		)
	},
}
```