<template>
  <div class="todo-background">
    <loading-container :isLoading="isLoading">
      <completed-wrapper v-if="completedTodoItems.length > 0"/>

      <draggable
        v-model="todoItems"
        :pull="false"
        handle=".todo-drag"
        @start="drag=true"
        @end="drag=false"
      >
        <todo-item v-for="todoItem in todoItems" v-bind:key="todoItem.id" :todo-item="todoItem"/>
      </draggable>

      <new-todo-item v-on:todo-item-added="newTodoItemAdded" :is-focused="focusNewTodoItemField"/>
    </loading-container>
  </div>
</template>

<script>
import completedWrapper from "@/features/todoitems/completed-wrapper";
import todoItem from "@/features/todoitems/todo-item.vue";
import newTodoItem from "@/features/todoitems/new-todo-item.vue";
import draggable from "vuedraggable";
import { mapGetters } from "vuex";

export default {
  components: {
    completedWrapper,
    todoItem,
    newTodoItem,
    draggable
  },
  data() {
    return {
      isLoading: true,
      focusNewTodoItemField: false
    };
  },
  computed: {
    ...mapGetters("listsModule", ["currentListId"]),
    ...mapGetters("todoModule", ["openTodoItems", "completedTodoItems"]),
    todoItems: {
      get() {
        return this.openTodoItems;
      },
      set(value) {
        this.$store.dispatch("todoModule/setTodoItemsOrder", value);
      }
    }
  },
  watch: {
    currentListId() {
      this.focusNewTodoItemField = false;
      this.loadTodoItems();
    }
  },
  mounted() {
    this.loadTodoItems();
  },
  methods: {
    loadTodoItems() {
      this.isLoading = true;

      this.$store.dispatch("todoModule/getTodoItems").then(() => {
        if (this.todoItems.length === 0) {
          this.focusNewTodoItemField = true;
        }
        this.isLoading = false;
      });
    },
    newTodoItemAdded() {
      this.focusNewTodoItemField = true;
      this.loadTodoItems();
    }
  }
};
</script>

<style lang="scss">
.todo-background {
  background-color: $primary-background-highlight-colour;
  min-height: calc(100vh - 278px);
}
</style>