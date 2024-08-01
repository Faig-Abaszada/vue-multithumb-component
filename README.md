# Multi-Thumb Slider Component

This is a Vue.js component that creates a multi-thumb slider. It's a flexible and customizable component that can be used in a variety of scenarios where a multi-thumb slider is needed.

## Features

- Multiple thumbs for different ranges.
- Customizable color for each thumb.
- Reset to default values.
- Error handling.
- Responsive design.

## Usage

First, import the component:

```javascript
import MultiThumbSlider from "@/components/MultiThumbSlider.vue";
```

Then, use it in your template:

```vue
<multi-thumb-slider
    :options="multithumbOptions"
    v-model:display="data.display"
    v-model:video="data.video"
    v-model:social="data.social"
    v-model:search="data.search"
    :totalBudget="data.totalBudget"
    :error="{ has: false, messages: 'sliderErrorMessages'}"
/>
```

The `options` prop is an array of objects, each representing a thumb on the slider. Each object should have the following properties:

- `label`: The label for the thumb.
- `key`: The key for v-model.
- `default`: The default value for the thumb.
- `color`: The color of the thumb.
- `disabled`: Whether the thumb is disabled or not.
- `error`: Whether there is an error associated with the thumb.

The `v-model` directives are used to bind the slider values to your data.

The `totalBudget` prop is used to calculate the percentage of each thumb.

The `error` prop is used to handle errors.

## Styles

The component uses scoped SCSS for styling. You can customize the styles according to your needs.

## Dependencies

This component is built with Vue.js and requires Vue 2.x or Vue 3.x.

## License

This component is open-source and available under the MIT License.