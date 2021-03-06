<template>
  <div class="vddl-draggable">
    <slot></slot>
  </div>
</template>

<script>
export default {
  name: 'vddl-draggable',
  // css: vddl-dragging, vddl-dragging-source
  props: {
    draggable: [ Object, Array ],
    wrapper: Array,
    index: Number,

    effectAllowed: String,
    type: String,

    // diable
    disableIf: Boolean,

    // callback fn
    dragstart: Function,
    selected: Function,
    dragend: Function,
    moved: Function,
    copied: Function,
    canceled: Function,
  },
  data() {
    return {};
  },
  computed: {},
  methods: {
    handleDragstart(event) {
      event = event.originalEvent || event;

      var draggable = JSON.stringify(this.draggable);
      // Check whether the element is draggable, since dragstart might be triggered on a child.
      if (draggable == 'false' || this.disableIf) return true;

      // Serialize the data associated with this element. IE only supports the Text drag type
      event.dataTransfer.setData("Text", draggable);

      // Only allow actions specified in dnd-effect-allowed attribute
      event.dataTransfer.effectAllowed = this.effectAllowed || "move";

      // Add CSS classes. IE9 not support 'classList'
      this.$el.className = this.$el.className.trim() + " vddl-dragging";
      setTimeout(() => {
        this.$el.className = this.$el.className.trim() + " vddl-dragging-source";
      }, 0);

      // Workarounds for stupid browsers, see description below
      this.dndDropEffectWorkaround.dropEffect = "none";
      this.dndDragTypeWorkaround.isDragging = true;

      // Save type of item in global state. Usually, this would go into the dataTransfer
      // typename, but we have to use "Text" there to support IE
      this.dndDragTypeWorkaround.dragType = this.type || undefined;

      // Try setting a proper drag image if triggered on a dnd-handle (won't work in IE).
      if (event._dndHandle && event.dataTransfer.setDragImage) {
        event.dataTransfer.setDragImage(this.$el, event._dndHandleLeft, event._dndHandleTop);
      }

      // Invoke callback
      if (typeof(this.dragstart) === 'function') {
        this.dragstart.call(this, event.target);
      }

      event.stopPropagation();
    },

    handleDragend(event) {
      event = event.originalEvent || event;

      var dropEffect = this.dndDropEffectWorkaround.dropEffect;
      switch (dropEffect) {
        case "move":
          if (typeof(this.moved) === 'function') {
            this.moved(this.index, event.target);
          } else {
            this.wrapper.splice(this.index, 1);
          }
          break;
        case "copy":
          if (typeof(this.copied) === 'function') {
            this.copied(this.draggable, event.target);
          }
          break;
        case "none":
          if (typeof(this.canceled) === 'function') {
            this.canceled(event.target);
          }
          break;
      }
      if (typeof(this.dragend) === 'function') {
        this.dragend(dropEffect, event.target);
      }

      // Clean up
      this.$el.className = this.$el.className.replace("vddl-dragging", "").trim();
      setTimeout(() => {
        this.$el.className = this.$el.className.replace("vddl-dragging-source", "").trim();
      }, 0);
      this.dndDragTypeWorkaround.isDragging = false;
      event.stopPropagation();
    },

    handleClick(event) {
      if (!this.selected) return;

      event = event.originalEvent || event;
      if (typeof(this.selected) === 'function') {
        this.selected(this.wrapper[this.index], event.target);
      }
      event.stopPropagation();
    },

    /**
     * Workaround to make element draggable in IE9
     * http://stackoverflow.com/questions/5500615/internet-explorer-9-drag-and-drop-dnd
     */
    handleSelected() {
      if (this.dragDrop) this.dragDrop();
      return false;
    },
  },
  watch: {
    disableIf(val) {
      this.$el.setAttribute('draggable', !val);
    },
  },
  mounted() {
    let status = true;
    if (this.disableIf) status = false;
    this.$el.setAttribute('draggable', status);

    this.$el.addEventListener('dragstart', this.handleDragstart, false);
    this.$el.addEventListener('dragend', this.handleDragend, false);
    this.$el.addEventListener('click', this.handleClick, false);
    this.$el.addEventListener('selectstart', this.handleSelected, false);
  },
  beforeDestroy() {
    this.$el.removeEventListener('dragstart', this.handleDragstart, false);
    this.$el.removeEventListener('dragend', this.handleDragend, false);
    this.$el.removeEventListener('click', this.handleClick, false);
    this.$el.removeEventListener('selectstart', this.handleSelected, false);
  },
};
</script>

<style lang="less">
.ddl-draggable {
  color: #222;
}
</style>
