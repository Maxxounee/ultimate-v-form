# Ultimate-V-Form

Набор компонентов формы. Работает как конструктор.
Реализован из необходимости создавать большие формы.
Позволяет декларативно описывать большие формы с любым дизайном, без
необходимости лезть в код компонента. Содержит в себе только обнуляющие стили.

## Импорт компонентов
```
import { FormWrapper, FormInput, FormText, FormSend, FormAgree } from 'ultimate-v-form';
```

## Примеры использования
`Вариант формы через компоненты `
```html
<FormWrapper
    :additional="{
        flatId: '1-2-3-4',
        mortgageType: 'family'
    }"                                 <!-- Дополнительные поля-->
    @before-send="onBeforeSend"        <!--Хук перед отправкой-->
    @send-success="countersHandler"    <!--Хук при успешной отправке-->
    @send-error="countersHandler"      <!--Хук при ошибке-->
    @available="onAvailableChange"     <!--Эмит при изменении доступности к отправке-->
>
  <FormInput
      name="phone"                     <!--Имя поля для api-->
      input-type="tel"                 <!--Тип инпута-->
      mask="phone"                     <!--Маска поля-->
      placeholder="Телефон"            <!--Плейсхолдер-->
      initial-value="+7 ("             <!--Начальное значение при фокусе поля-->
      required                         <!--Обязательное поле-->
      @update="onInputUpdate"          <!--Эмит при изменении поля-->
  />

  <FormInput
      name="email"
      input-type="email"
      placeholder="email"
  />

  <FormText
      name="message"                   <!--Имя инпута для api-->
      placeholder="Your text"          <!--Плейсхолдер-->
      minimal-length-required="10"     <!--Минимальная длина текста. Аналог required-->
      @update="onInputUpdate"          <!--Эмит при изменении поля-->
  />

  <FormAgree
    initial-active                     <!--Изначально активно-->
    @update="agreeUpdate"              <!--Эмит при изменении поля  -->
  >
    <template #before>                 <!--Темплейт для чекбокса-->
        <Icon id="checkbox" />
    </template>
    <template #default>                <!--Темплейт для текста-->
      Отправляя форму вы согласны
    </template>
  </FormAgree>

  <FormSend
      @click="sendClick"               <!-- Эмит клика-->
  >
      <StandardButton />               <!--Слот для кнопки-->
  </FormSend>

  <template #preloader>                <!--Темплейт для прелоадера FormWrapper-->
    <img src="/assets/preloader.svg" />
  </template>
</FormWrapper>
```

`Вариант формы через объект в data() `
```html
<template>
  <div class="page">
    <FormWrapper>
      <Component
        v-for="(field, index) in fields"
        :is="field.component"             <!--Название компонента-->
        :key="index"
        v-bind="field.binds"              <!--Пропсы компонента-->
        v-on="field.handlers"             <!--Обработчики компонента-->
      />
      <FormSend>
        <StandardButton />
      </FormSend>
      <template #preloader>
        <img src="/assets/preloader.svg" />
      </template>
    </FormWrapper>
  </div>
</template>

<script>
import { FormWrapper, FormInput, FormText, FormSend, FormAgree } from 'ultimate-v-form';
export default {
  components: {
    FormWrapper,
    FormInput,
    FormText,
    FormSend,
    FormAgree,
  },
  data() {
    return {
      fields: [
        {
          component: 'FormInput',       // Название компонента
          binds: {
            name: 'phone',              // Имя поля для api
            inputType: 'tel',           // Тип инпута
            mask: 'phone',              // Маска поля
            placeholder: 'Телефон',     // Плейсхолдер
            initialValue: '+7 (',       // Начальное значение при фокусе поля
            required: true,             // Обязательное поле
          },
          handlers: {                   // Описание обработчиков
            update: this.onInputUpdate,
          },
        },
        {
          component: 'FormInput',
          binds: {
            name: 'email',
            inputType: 'email',
            placeholder: 'Почта',
          },
          handlers: {                  
            update: this.onInputUpdate,
          },
        },
        {
          component: 'FormText',         // Название компонента
          binds: {
            name: 'message',             // Имя инпута для api 
            minimalLengthRequired: 0,    // Минимальная длина текста. Аналог required  
            placeholder: 'Ваш текст',    // Плейсхолдер 
          },
          handlers: {
            update: this.onInputUpdate,
          },
        },
        {
          component: 'FormAgree',         // Название компонента
          binds: {
            initialActive: true,          // Изначально активно
            text: 'Солгасен на обработку' // Текст
          },
          handlers: {
            update: this.onAgreeUpdate,   // Эмит при изменении поля
          },
        },    
      ],
    };
  }
}
</script>
```
