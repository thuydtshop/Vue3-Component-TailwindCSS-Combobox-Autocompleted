# Vue3-Component-TailwindCSS-Combobox-Autocompleted
Created Input Combobox Autocompleted with Vue3 + TailwindCSS


### Packages Requirements
Vue 3

TailwindCSS

Headlessui

Heroicons


## How to use
```import MyCombobox from '../../../commons/MyCombobox.vue';```

```
<MyCombobox 
  :items="items" 
  :selecteds="selecteds"
  :autocomplete="true"
  @remove="remove"
  @add="add"
/>
```
