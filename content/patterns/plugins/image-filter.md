+++
title = "Image filter"
+++

The Image filter plugin adds filter information to the file in the form of a Color Matrix.

The [Image preview](../image-preview) plugin uses this information to show the correct preview. The [Image transform](../image-transform) plugin uses this information to transform the image before uploading it to the server.

## Installation

### Using npm

{{<cmd>}}npm i filepond-plugin-image-filter --save{{</cmd>}}

Now we can add the Image Filter plugin to our project like this.

```js
// Import FilePond
import * as FilePond from 'filepond';

// Import the plugin code
import FilePondPluginImageFilter from 'filepond-plugin-image-filter';

// Register the plugin
FilePond.registerPlugin(FilePondPluginImageFilter);
```


### Using a CDN

```html
<!-- add before </body> -->
<script src="https://unpkg.com/filepond-plugin-image-filter/dist/filepond-plugin-image-filter.js"></script>
<script src="https://unpkg.com/filepond/dist/filepond.js"></script>

<script>
// Register the plugin
FilePond.registerPlugin(FilePondPluginImageFilter);

// ... FilePond initialisation code here
</script>
```

### Manual installation

```html
<!-- add before </body> -->
<script src="filepond-plugin-image-filter.js"></script>
<script src="filepond.js"></script>

<script>
// Register the plugin
FilePond.registerPlugin(FilePondPluginImageFilter);

// ... FilePond initialisation code here
</script>
```

## About ColorMatrices

A Color Matrix is an array of 20 numbers, it represents a transformation to apply to each color in an image.

```
[ R, G, B, A, T,
  R, G, B, A, T,
  R, G, B, A, T,
  R, G, B, A, T ]
```

An example grayscale Color Matrix.
```js
const pond = FilePond.create(inputElement, {
    imageFilterColorMatrix: [
        0.299, 0.587, 0.114, 0, 0,
        0.299, 0.587, 0.114, 0, 0,
        0.299, 0.587, 0.114, 0, 0,
        0.000, 0.000, 0.000, 1, 0
    ]
});
```

## Properties

| Property             | Default | Description                                                                             |
| -------------------- | ------- | --------------------------------------------------------------------------------------- |
| allowImageFilter       | `true`  | Enable or disable image filtering                                                        |
| imageFilterColorMatrix | `null`  | The Color Matrix to apply to the image in the preview and transform phase.  |
