+++
title = "Svelte"
+++

The FilePond Svelte Component functions as a tiny adapter for the FilePond object so it's easier to use with Svelte.

Installation instructions for npm.

```bash
npm install svelte-filepond filepond --save
```

Now you can import the `<FilePond>` Component in your Svelte project. The adapter automatically references FilePond methods to the Component instance so you can use the Component just like you would use FilePond itself.

We can configure our pond by using the [FilePond instance properties](../../api/filepond-instance/#properties) as attributes on the `<FilePond>` Component. See the example below.

```html
<FilePond allowMultiple={true} max-files={3} server="/api"/>
```

Callbacks can be used exactly like they're used on FilePond itself.

```html
<!-- normal -->
<FilePond oninit={handleFilePondInit}/>
```

A more elaborate example, see the GitHub repositor example directory for a live demo.

```html
<script>
import FilePond, { registerPlugin, supported } from 'svelte-filepond';

// Import the Image EXIF Orientation and Image Preview plugins
// Note: These need to be installed separately
// `npm i filepond-plugin-image-preview filepond-plugin-image-exif-orientation --save`
import FilePondPluginImageExifOrientation from 'filepond-plugin-image-exif-orientation'
import FilePondPluginImagePreview from 'filepond-plugin-image-preview'

// Register the plugins
registerPlugin(FilePondPluginImageExifOrientation, FilePondPluginImagePreview);

// a reference to the component, used to call FilePond methods
let pond;

// pond.getFiles() will return the active files

// the name to use for the internal file input
let name = 'filepond';

// handle filepond events
function handleInit() {
	console.log('FilePond has initialised');
}

function handleAddFile(err, fileItem) {
	console.log('A file has been added', fileItem);
}
</script>

<div class="app">

	<FilePond bind:this={pond} {name}
		server="/api"
		allowMultiple={true}
		oninit={handleInit}
		onaddfile={handleAddFile}/>
	
</div>

<style global>
@import 'filepond-plugin-image-preview/dist/filepond-plugin-image-preview.css';
</style>
```