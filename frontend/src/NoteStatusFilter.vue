<script>
export default {
  name: "NoteStatusFilter",
  emits: ["filter-selected"], // Emit event for filter selection
  methods: {
    onFilterChange(event) {
      this.$emit("filter-selected", event.target.value); // Emit the selected value
    },
  },
};
</script>

<template>
  <div class="radio-input">
    <label>
      <input type="radio" id="value-0" name="value-radio" value="all" @change="onFilterChange">
      <span>All</span>
    </label>
    <label>
      <input type="radio" id="value-1" name="value-radio" value="unimportant" @change="onFilterChange">
      <span>Unimportant</span>
    </label>
    <label>
      <input type="radio" id="value-2" name="value-radio" value="serious" @change="onFilterChange">
      <span>Serious</span>
    </label>
    <label>
      <input type="radio" id="value-3" name="value-radio" value="urgent" @change="onFilterChange">
      <span>Urgent</span>
    </label>
    <span class="selection"></span>
  </div>
</template>

<style scoped>
/* From Uiverse.io by Yaya12085 */
.radio-input input {
  display: none;
}

.radio-input {
  --container_width: 600px; /* Adjusted width to accommodate "All" option */
  position: relative;
  display: flex;
  align-items: center;
  border-radius: 10px;
  background-color: #fff;
  color: #000000;
  width: var(--container_width);
  overflow: hidden;
  border: 1px solid rgba(53, 52, 52, 0.226);
}

.radio-input label {
  width: 25%;
  padding: 10px;
  cursor: pointer;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1;
  font-weight: 600;
  letter-spacing: -1px;
  font-size: 14px;
}

.selection {
  display: none;
  position: absolute;
  height: 100%;
  width: calc(var(--container_width) / 4); /* Adjusted for the "All" option */
  z-index: 0;
  left: 0;
  top: 0;
  transition: .15s ease;
}

.radio-input label:has(input:checked) {
  color: #fff;
}

.radio-input label:has(input:checked) ~ .selection {
  display: inline-block;
}

.radio-input label:nth-child(1):has(input:checked) ~ .selection {
  background-color: #bdc3c7; /* All */
  transform: translateX(calc(var(--container_width) * 0/4));
}

.radio-input label:nth-child(2):has(input:checked) ~ .selection {
  background-color: #00b894; /* Unimportant */
  transform: translateX(calc(var(--container_width) * 1/4));
}

.radio-input label:nth-child(3):has(input:checked) ~ .selection {
  background-color: #fdcb6e; /* Serious */
  transform: translateX(calc(var(--container_width) * 2/4));
}

.radio-input label:nth-child(4):has(input:checked) ~ .selection {
  background-color: #e17055; /* Urgent */
  transform: translateX(calc(var(--container_width) * 3/4));
}

label {
  text-transform: uppercase;
}
</style>
 