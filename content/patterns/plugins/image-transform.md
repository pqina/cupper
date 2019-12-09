+++
title = "Image transform"
+++

The Image transform plugin applies the image modifications supplied by the [Image crop](../image-crop) and [Image resize](../image-resize) plugins before the image is uploaded. It can also change the file format to either JPEG or PNG.

You can combine the image transform plugin with the [Doka Image Editor](https://pqina.nl/doka/?ref=filepond) to offer image manipulation capabilities to your users.

## Installation

### Using npm

{{<cmd>}}npm i filepond-plugin-image-transform --save{{</cmd>}}

Now we can add the Image Transform plugin to our project like this.

```js
// Import FilePond
import * as FilePond from 'filepond';

// Import the plugin code
import FilePondPluginImageTransform from 'filepond-plugin-image-transform';

// Register the plugin
FilePond.registerPlugin(FilePondPluginImageTransform);
```


### Using a CDN

```html
<!-- add before </body> -->
<script src="https://unpkg.com/filepond-plugin-image-transform/dist/filepond-plugin-image-transform.js"></script>
<script src="https://unpkg.com/filepond/dist/filepond.js"></script>

<script>
// Register the plugin
FilePond.registerPlugin(FilePondPluginImageTransform);

// ... FilePond initialisation code here
</script>
```

### Manual installation

```html
<!-- add before </body> -->
<script src="filepond-plugin-image-transform.js"></script>
<script src="filepond.js"></script>

<script>
// Register the plugin
FilePond.registerPlugin(FilePondPluginImageTransform);

// ... FilePond initialisation code here
</script>
```

## Using Transform Variants

The following settings add two additional files to the default transform output. A medium and small thumbnail version of the input file.

```js
FilePond.create({
    imageResizeTargetWidth: 600,
    imageCropAspectRatio: 1,
    imageTransformVariants: {
        'thumb_medium_': transforms => {
            transforms.resize.size.width = 384;
            return transforms;
        },
        'thumb_small_': transforms => {
            transforms.resize.size.width = 128;
            return transforms;
        }
    }
});
```

Instead of a single file, the transform plugin will now pass an array of files to the FilePond file processing method. The name of each file will be prefixed with the name of the transform variant.


## Using Before and After Create Blob

The `imageTransformBeforeCreateBlob` and `imageTransformAfterCreateBlob` hooks are useful to make changes to the data without having to exit FilePond.

```js
FilePond.create({
    imageResizeTargetWidth: 600,
    imageCropAspectRatio: 1,
    imageTransformBeforeCreateBlob: (canvas) => new Promise(resolve => {

        // Do something with the canvas, like drawing some text on it
        const ctx = canvas.getContext('2d');
        ctx.font = '48px serif';
        ctx.fillText('Hello world', 10, 50);

        // return canvas to the plugin for further processing
        resolve(canvas);
    }),
    imageTransformAfterCreateBlob: (blob) => new Promise(resolve => {
        // do something with the blob, for instance send it to a custom compression alogrithm
        

        // return the blob to the plugin for further processing
        resolve(blob);
    })
});
```


## Adding Markup to Images

The image transform plugin can render markup on top of images. Here we use the [metadata plugin](../file-metadata) to define a watermark to be placed in the bottom right of dropped images.

[Live demo](https://pqina.github.io/filepond-plugin-image-transform/)

```js
FilePond.create({
    fileMetadataObject: {
        'markup': [
            [
                'rect', {
                    left: 0,
                    right: 0,
                    bottom: 0,
                    height: '60px',
                    backgroundColor: 'rgba(0,0,0,.5)'
                },
            ],
            [
                'image', {
                    right: '10px',
                    bottom: '10px',
                    width: '128px',
                    height: '34px',
                    src: './filepond-logo.svg',
                    fit: 'contain'
                }
            ]
        ]
    },
    onpreparefile: (file, output) => {
        const img = new Image();
        img.src = URL.createObjectURL(output);
        document.body.appendChild(img);
    }
});
```


## Properties

| Property                     | Default | Description                                                                                                                                                                                                                                                                                                         |
| ---------------------------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| allowImageTransform          | `true`  | Enable or disable client-side image transforms                                                                                                                                                                                                                                                                      |
| imageTransformOutputMimeType | `null`  | The file type of the output image. Can be either `'image/jpeg'` or `'image/png'` as those are the formats the [HTML5 Canvas element can output to](https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement/toBlob). If not defined, will default to the input file type, and fallback to `'image/jpeg'`. |
| imageTransformOutputQuality  | `null`  | The quality of the output image supplied as a value between `0` and `100`. Where `100` is best quality and `0` is worst. When not supplied it will use the browser default quality which averages around `94`.                                                                                                      |
| imageTransformOutputQualityMode | `'always'` | Should output quality be enforced, set the `'optional'` to only apply when a transform is required due to other requirements (e.g. resize or crop). |
| imageTransformOutputStripImageHead | `true` | Should JPEG EXIF data be stripped from the output image, defaults to `true` (as that is what the browser does), set to `false` to copy over the EXIF data from the original image to the output image. This will automatically remove the EXIF `orientation` tag to prevent orientation problems. |
| imageTransformClientTransforms | `null` | An array of transforms to apply on the client, useful if we, for instance, want to do resizing on the client but cropping on the server. `null` means apply all transforms (`'resize'`, `'crop'`). |
| imageTransformVariants | `null` | An object that can be used to output multiple different files based on different transfom instructions. |
| imageTransformVariantsIncludeDefault | `true` | Should the transform plugin output the default transformed file. |
| imageTransformVariantsDefaultName | `null` | The name to use in front of the file name. |
| imageTransformVariantsIncludeOriginal | `false` | Should the transform plugin output the original file. |
| imageTransformVariantsOriginalName | `null` | The name to use in front of the original file name. |
| imageTransformBeforeCreateBlob | `null` | A hook to make changes to the canvas before the file is created. |
| imageTransformAfterCreateBlob | `null` | A hook to make changes to the file after the file has been created. |
| imageTransformCanvasMemoryLimit | `isBrowser && isIOS ? 4096 * 4096 : null` | A memory limit to make sure the canvas can be used correctly when rendering the image. By default this is only active on iOS. |