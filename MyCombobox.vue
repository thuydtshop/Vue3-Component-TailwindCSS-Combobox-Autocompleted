<!-- multiple select tags -->
<template>
  <Popover v-slot="{ open }" class="relative w-full">
    <PopoverButton
      :class="open ? '' : 'text-opacity-90'"
      class="w-full group inline-flex items-center rounded-md text-base hover:text-opacity-100 focus:outline-none focus-visible:ring-2 focus-visible:ring-white focus-visible:ring-opacity-75"
      @click="showAndFocus"
    >
      <div class="flex items-start h-auto w-full border border-grey p-1 flex-wrap gap-1 min-h-[32px]">
        <div v-for="selected in selecteds" :key="selected.id"
          class="flex items-center px-2 gap-1 border border-grey2 rounded"
        >
          {{ selected.name }}
          <IconCloseCircle class="w-4 h-4 fill-primary" @click="removeTag(selected.id)" />
        </div>
        <div class="w-fit">
          <input
            v-if="autocomplete"
            v-model="search"
            type="text"
            class="border-none outline-none appearance-none w-auto"
            @keydown.enter="addTag(filteredItems[0])"
            @keydown.down="focusNext"
            @keydown.up="focusPrev"
            @keydown.backspace="removeLastTag"
            @focus="showAndFocus"
            @input="inputHandler"
            autoatocomplete="off"
          />
        </div>
      </div>
    </PopoverButton>

    <transition
      enter-active-class="transition duration-200 ease-out"
      enter-from-class="translate-y-1 opacity-0"
      enter-to-class="translate-y-0 opacity-100"
      leave-active-class="transition duration-150 ease-in"
      leave-from-class="translate-y-0 opacity-100"
      leave-to-class="translate-y-1 opacity-0"
    >
      <PopoverPanel v-slot="{ close }"
        class="absolute z-10 mt-1 transform max-h-64 overflow-y-auto w-full border border-grey rounded shadow-lg ring-1 ring-black ring-opacity-5"
      >
        <div class="overflow-hidden w-full">
          <div class="relative grid gap-4 bg-white p-4 grid-cols-1 w-full">
            <template v-if="filteredItems.length > 0">
              <template v-for="item in filteredItems">
                <div v-if="!hideSelected || (hideSelected && !isSelected(item.id))"
                  class="-m-3 flex items-center rounded p-2 transition duration-150 ease-in-out hover:bg-stone1 focus:outline-none focus-visible:ring focus-visible:ring-orange-500 focus-visible:ring-opacity-50 cursor-pointer"
                  :class="focusing && focusing.id === item.id ? 'bg-stone1' : ''"
                  @click="addTag(item)"
                >
                  <p class="text-sm font-medium text-gray-900">
                    {{ item.name }}
                  </p>
                </div>
              </template>
            </template>
            <p v-else>データが見つかりません</p>
          </div>
        </div>
      </PopoverPanel>
    </transition>
  </Popover>
</template>
<script lang="ts">
import { Popover, PopoverButton, PopoverPanel } from '@headlessui/vue';
import IconCloseCircle from '../icons/IconCloseCircle.vue';

export default {
  components: {
    Popover,
    PopoverButton,
    PopoverPanel,
    IconCloseCircle,
  },
  props: {
    items: { type: Array, required: true },
    selecteds: { type: Array, required: true },
    hideSelected: { type: Boolean, default: true },
    autocomplete: { type: Boolean, default: false },
  },
  data() {
    return {
      search: '',
      searchingItems: [],
      focusing: {}
    };
  },
  mounted() {
    document.addEventListener('keydown', this.keydown);

    document.addEventListener('click', (e: any) => {
      if (!this.$el.contains(e.target)) {
        this.search = '';
        this.focusing = {};
        this.$emit('blur');
      }
    });
  },
  computed: {
    filteredItems() {
      if (this.search.length > 0) {
        if (!this.autocomplete) {
          return this.items.filter((item: { name: string; }) => item.name.toLowerCase().includes(this.search.toLowerCase()));
        }

        // get item not in selecteds and have name match with search
        return this.items.filter((item: { id: any; name: string; }) => {
          if (this.selecteds.find((selected: { id: any; }) => selected.id === item.id)) {
            return false;
          }

          return item.name.toLowerCase().includes(this.search.toLowerCase());
        });
      }

      return this.items;
    },
  },
  methods: {
    removeTag(id: number) {
      if (!id) {
        return;
      }

      this.$emit('remove', id);
    },
    addTag(item: any) {
      if (!item) {
        return;
      }

      if (this.selecteds.length > 0 && this.selecteds.find((selected: { id: any; }) => selected.id === item.id)) {
        return;
      }
      
      this.$emit('add', item);
    },
    removeLastTag() {
      if (this.search.length === 0 && this.selecteds.length > 0) {
        this.$emit('remove', this.selecteds[this.selecteds.length - 1].id);
      }
    },
    isSelected(id: number) {
      return this.selecteds.find((selected: { id: any; }) => selected.id === id);
    },

    showAndFocus() {
      this.$nextTick(() => {
        const input = this.$el.querySelector('input');
        if (input) {
          input.focus();
        }
      });
    },
    focusNext() {
      const input = this.$el.querySelector('input');
      if (input && document.activeElement === input) {
        if (this.filteredItems.length > 0) {
          this.focusing = this.filteredItems[0];
        }
        input.blur();

        return;
      }

      if (this.filteredItems.length === 0) {
        return;
      }

      if (!this.focusing || !this.focusing.id) {
        this.focusing = this.filteredItems[0];
        return;
      }

      const index = this.filteredItems.findIndex((item: { id: any; }) => item.id === this.focusing.id);
      this.focusing = index === -1 
        ? this.filteredItems[0] : (this.filteredItems[index + 1] 
        ? this.filteredItems[index + 1] : this.filteredItems[index]);
    },
    focusPrev() {
      const input = this.$el.querySelector('input');
      if (input && document.activeElement === input) {
        if (this.filteredItems.length > 0) {
          this.focusing = this.filteredItems[this.filteredItems.length - 1];
        }
        input.blur();

        return;
      }

      if (this.filteredItems.length === 0) {
        return;
      }

      if (!this.focusing || !this.focusing.id) {
        this.focusing = this.filteredItems[this.filteredItems.length - 1];
        return;
      }

      const index = this.filteredItems.findIndex((item: { id: any; }) => item.id === this.focusing.id);
      this.focusing = index === -1 
        ? this.filteredItems[this.filteredItems.length - 1] 
        : (this.filteredItems[index - 1] ? this.filteredItems[index - 1] : this.filteredItems[index]);
    },
    keydown(e: KeyboardEvent) {
      if (e.key === 'Enter' && this.focusing && this.focusing.id) {
        this.addTag(this.focusing);

        // focus to input
        this.$nextTick(() => {
          const input = this.$el.querySelector('input');
          if (input) {
            input.focus();
          }
        });

        return;
      }

      if (e.key === 'ArrowDown') {
        e.preventDefault();
        this.focusNext();
      } else if (e.key === 'ArrowUp') {
        e.preventDefault();
        this.focusPrev();
      }
    },
    inputHandler(e: any) {
      this.search = e.target.value;
      this.focusing = {};
    },
  },
}
</script>
