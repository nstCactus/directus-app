<template>
  <draggable
    v-model="sortableList"
    element="div"
    class="interface-checkboxes"
    :class="{ draggable: options.draggable, single: options.single }"
    animation="200"
    ghost-class="ghost"
    :draggable="options.draggable ? '.sortable-box.sortable' : false"
    @end="saveSort"
  >
    <v-checkbox
      v-for="item in sortableList"
      :id="item.id + '-' + item.val"
      :key="item.id"
      name="list-sorting"
      class="sortable-box"
      :class="{ sortable: options.draggable }"
      :value="item.val"
      :disabled="readonly"
      :label="item.label"
      :checked="selection.includes(item.val)"
      @change="updateValue(item.val, $event)"
    />
  </draggable>
</template>

<script>
import mixin from "@directus/extension-toolkit/mixins/interface";
import shortid from "shortid";

export default {
  name: "InterfaceCheckboxes",
  mixins: [mixin],

  data() {
    return {
      sortableList: [],
      customValue: "",
      customChecked: false
    };
  },

  computed: {
    // The currently saved selection. It's this.value converted to an array, and
    // adjusted for the wrapped `,` characters if applicable
    selection() {
      if (!this.value) return [];

      let selection;

      // Convert the value to an array
      if (typeof this.value === "string") {
        selection = this.value.split(",");
      } else {
        selection = this.value;
      }

      // If the wrapping option is activated, the first and last item will be an
      // empty string ( ",rijk," will split to ["", "rijk", ""]. This will remove those empty strings
      if (this.options.wrap && selection.length > 2) {
        selection.pop();
        selection.shift();
      }

      return selection;
    }
  },

  created() {
    const options = Object.keys(this.options.choices).map(key => ({
      val: key,
      label: this.options.choices[key]
    }));

    let sortableOptions;

    if (this.options.draggable) {
      // Convert the selected items and the choices into an array sorted by the
      // manual sort of the user.
      const selected = this.selection.map(val => ({
        val: val,
        label: this.options.choices[val]
      }));

      const optionsWithoutSelection = options.filter(
        option => this.selection.includes(option.val) === false
      );

      sortableOptions = [...selected, ...optionsWithoutSelection];
    } else {
      sortableOptions = options;
    }

    // Add a unique ID to each sortable option so we can use that to key the items in the template
    sortableOptions = sortableOptions.map(item => ({
      ...item,
      id: shortid.generate()
    }));

    this.sortableList = sortableOptions;
  },

  methods: {
    updateValue(val) {
      let selection = [...this.selection];
      if (selection.includes(val)) {
        selection.splice(selection.indexOf(val), 1);
      } else {
        selection.push(val);
      }

      selection = selection.join(",");

      if (this.options.wrap && selection.length > 0) {
        selection = `,${selection},`;
      }

      if (this.type === "array") {
        selection = selection.split(",");
      }

      this.$emit("input", selection);
    },

    saveSort() {
      const selection = this.selection;

      const staged = this.sortableList
        // Get all the values of the sorted available checkboxes
        .map(s => s.val)
        // Only leave the ones that are selected
        .filter(s => selection.includes(s));

      return this.$emit("input", staged);
    }
  }
};
</script>

<style lang="scss" scoped>
.interface-checkboxes {
  width: var(--width-x-large);
  max-width: 100%;
  display: grid;
  grid-gap: 20px;
  grid-template-columns: repeat(1, 1fr);

  @media only screen and (min-width: 330px) {
    grid-template-columns: repeat(2, 1fr);
  }

  @media only screen and (min-width: 480px) {
    grid-template-columns: repeat(3, 1fr);
  }
  @media only screen and (min-width: 800px) {
    grid-template-columns: repeat(4, 1fr);
  }
}

.single {
  grid-template-columns: repeat(1, 1fr);
}

.sortable-box {
  position: relative;
  transition: opacity var(--medium) var(--transition),
    background-color var(--slow) var(--transition);
}

.sortable {
  margin-left: 12px; // To make space to show the drag handle

  &:after {
    position: absolute;
    font-family: "Material Icons", sans-serif;
    display: inline-block;
    line-height: 1;
    letter-spacing: normal;
    vertical-align: middle;
    content: "drag_indicator";
    height: 100%;
    width: 24px;
    font-size: 24px;
    left: -20px;
    color: var(--lighter-gray);
    cursor: grab;
    top: 0;
  }
}
</style>

<style lang="scss">
// The styles for the 'drop-preview' eg the ghost item that shifts around in the list to show
// the user where the item is going to be dropped
// NOTE: this class is added dynamically and can't be scoped in the style block above
.ghost {
  opacity: 0.4;
}
</style>
