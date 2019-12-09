+++
title = "Server"
weight = 5
+++

## Introduction

The server API endpoints can be configured with the `server` property. Only after these have been configured can FilePond upload files to a server using `XMLHttpRequest`.

We'll first go over the different end points and what they do before discussing how to configure them.


### Process

Asynchronously uploading files with FilePond is called processing. In short, FilePond sends a file to the server and expects the server to return a unique file id. This unique id is then used to revert uploads or restore earlier uploads.

{{%note%}}
Along with the file object, FilePond also sends the file metadata to the server, both these objects are given the same `name`.
{{%/note%}}

The upload process described over time:

1. **FilePond** uploads file `my-file.jpg` as `multipart/form-data` using a `POST` request
2. **server** saves file to unique location `tmp/12345/my-file.jpg`
3. **server** returns unique location id `12345` in `text/plain` response
4. **FilePond** stores unique id `12345` in a hidden input field
5. **client** submits the FilePond parent form containing the hidden input field with the unique id
6. **server** uses the unique id to move `tmp/12345/my-file.jpg` to its final location and remove the `tmp/12345` folder

{{%note%}}
FilePond uses unique file ids to prevent showing information about the server file structure to the client.
{{%/note%}}

### Process Chunks

To process files in chunks set `chunkUploads` to `true`.

FilePond will then slice up files bigger than the set `chunkSize` into parts.

Custom headers used in requests

| header        | value           |
| ------------- | --------------- |
| Upload-Length | The total size of the file being transferred |
| Upload-Name   | The name of the file being transferred |
| Upload-Offset | The offset of the chunk being transferred | 
| Content-Type  | The content type of a patch request, set to `'application/offset+octet-stream'` |

In short: 

- FilePond will send a `POST` request (without file) to start a chunked transfer, expecting to receive a unique transfer id in the response body, it'll add the `Upload-Length` header to this request.
- FilePond will send a `PATCH` request to push a chunk to the server. Each of these requests is accompanied by a `Content-Type`, `Upload-Offset`, `Upload-Name`, and `Upload-Length` header.
- FilePond will send a `HEAD` request to determine which chunks have already been uploaded, expecting the file offset of the next expected chunk in the `Upload-Offset` response header.


In detail:

1. **FilePond** requests a transfer id from the server, a unique location to identify this transfer with. It does this using a `POST` request. The request is accompanied by the metadata and the total file upload size set to the `Upload-Length` header.
2. **server** create unique location `tmp/12345/`
3. **server** returns unique location id `12345` in `text/plain` response
4. **FilePond** stores unique id `12345` in file item
5. **FilePond** sends first chunk using a `PATCH` request adding the unique id `12345` in the URL, each `PATCH` request is accompanied by a `Upload-Offset`, `Upload-Length`, and `Upload-Name` header. The `Upload-Offset` header contains the byte offset of the chunk, the `Upload-Length` header contains the total file size, the `Upload-Name` header contains the file name.
6. **FilePond** sends chunks until all chunks have been uploaded succesfully.
7. **server** creates the file if all chunks have been received succesfully.
7. **FilePond** stores the unique id `12345` as the server id of this file.
5. **client** submits the FilePond parent form containing the hidden input field with the unique id
6. **server** uses the unique id to move `tmp/12345/my-file.jpg` to its final location and remove the `tmp/12345` folder

If one of the chunks fails to upload after the set amount of retries in `chunkRetryDelays` the user has the option to retry the upload.

1. **FilePond** As FilePond remembers the previous transfer id the process now starts of with a `HEAD` request accompanied by the transfer id (`12345`) in the URL.
2. **server** responds with `Upload-Offset` set to the next expected chunk offset in bytes.
3. **FilePond** marks all chunks with lower offsets as complete and continues with uploading the chunk at the requested offset.

Everything continues like normal.


### Revert

There's one way the **client** can deviate from the previous path and that is by reverting the upload. Let's go back to step five and switch to this alternate reality.

<ol start="5">
<li><strong>FilePond</strong> sends <code>DELETE</code> request with <code>12345</code> as body by tapping the undo button
<li><strong>server</strong> removes temporary folder matching the supplied id <code>tmp/12345</code> and returns an empty response
</ol>

This is another reason why FilePond uses unique ids. If we're going to give the client the power to influence the server file system that power should be very minimal.

### Restore

FilePond uses the `restore` end point to restore temporary server files. This might be useful in a situation where the user closes the browser window but hadn't finished completing the form. Temporary files can be set with the `files` property.

Step one and two now look like this.

1. **FilePond** requests restore of file with id `12345` using a `GET` request
2. **server** returns a file object with header `Content-Disposition: inline; filename="my-file.jpg"`

### Load

The `restore` end point is used to restore a temporary file, the `load` end point is used to restore already uploaded server files. These files might be located in a database or somewhere on the server file system. Either way they might not be directly accessible from the web.

For situations where a user might want to edit an existing file selection we can use the `load` end point to restore those files.

1. **FilePond** requests restore of file with id `12345` or a file name using a `GET` request
2. **server** returns a file object with header `Content-Disposition: inline; filename="my-file.jpg"`

### Fetch

The `fetch` end point is used to load files located on remote servers. When a user drops a remote link, FilePond asks the server to download it (CORS might otherwise block it). In this situation the server serves as a proxy.

1. **FilePond** requests fetch of file `http://somewhere/their-file.jpg` using a `GET` request
2. **server** returns a file object as if the file is located on the server

An alternative is to configure **FilePond** to only do a `HEAD` request. The server should then save the file to `tmp/12345/my-file.jpg` and return response headers with the required file information.

```
header('Access-Control-Expose-Headers: Content-Disposition, Content-Length, X-Content-Transfer-Id');
header('Content-Type: image/jpeg');
header('Content-Length: 3965123');
header('Content-Disposition: inline; filename="my-file.jpg"');    
header('X-Content-Transfer-Id: 12345');
```


### Remove

The `remove` end point is used to remove `local` files located on the server. This end point is not enabled by default and can only be set to a custom function.


## Configuration

The `server` configuration property expects an object or a URL. If it's not defined, FilePond will not be able to upload file to the server or use fetch functionality.

### URL

Setting a single URL is the most basic form of defining a server configuration.

```js
FilePond.setOptions({
    server: './'
});
```

This tells FilePond the api is located at the same location as the current page. It will then assume it can call all methods on this url. Like shown below.

| method  | type        | path           |
| ------- | ----------- | -------------- |
| process | POST        |                |
| revert  | DELETE      |                |
| load    | GET         | ?load=<source> |
| restore | GET         | ?restore=<id>  |
| fetch   | GET|HEAD    | ?fetch=<url>   |
| patch   | PATCH       | ?patch=<id>    |

We can of course supply a path or URL to another location, FilePond will simply append the above default paths to the supplied value. If we want more fine grained control we can use an object to configure the server end points.

### Object Configuration

To setup asynchronous uploading only, we pass the location of the process end point.

```js
FilePond.setOptions({
    server: {
        process: './process',
        fetch: null,
        revert: null
    }
});
```

This configuration assumes that the `process` end point is located on the same server. Reverting a file upload and fetching a remote file have been disabled. Restoring or loading an earlier uploaded file with this configuration is also not possible.

To unlock these features we have to supply FilePond with some more end points using the `revert`, `fetch`, `load` and `restore` properties.

```js
FilePond.setOptions({
    server: {
        process: './process',
        revert: './revert',
        restore: './restore/',
        load: './load/',
        fetch: './fetch/'
    }
});
```

FilePond will append the dropped URL to the `fetch` method, and the unique file id will automatically be added to the `restore` and `load` end points. `restore`, `load` and `fetch` are `GET` requests while `process` is a `POST` request and `revert` is a `DELETE` request.

If the endpoints are located on a different server we can add a `url` property to tell FilePond its location.

```js
FilePond.setOptions({
server:{
    url: 'http://192.168.0.100',
    process: './process',
    revert: './revert',
    restore: './restore/',
    load: './load/',
    fetch: './fetch/'
});
```

Depending on our project we might have to pass additional information to each request. We can accomplish this by turning the end point into an object which allows for more fine grain control over how FilePond handles each request.

For brevity we'll only look at the `process` property. All other properties can be configured with the same configuration object.

```js
FilePond.setOptions({
    server: {
        url: 'http://192.168.0.100',
        process: {
            url: './process',
            method: 'POST',
            withCredentials: false,
            headers: {},
            timeout: 7000,
            onload: null,
            onerror: null,
            ondata: null
        }
    }
});
```

| key             | description                                          | required |
| --------------- | ---------------------------------------------------- | -------- |
| url             | Path to the end point                                | yes      |
| method          | Request method to use                                | no       |
| withCredentials | Toggles the XMLHttpRequest withCredentials on or off | no       |
| headers         | An object containing additional headers to send, or when uploading chunks this can be a function that is expected to return a header object      | no       |
| timeout         | Timeout for this action                              | no       |
| onload          | Called when server response is received, useful for getting the unique file id from the server response | no |
| onerror         | Called when server error is received, receives the response body, useful to select the relevant error data | no |
| ondata          | Called with the formdata object right before it is sent, return extended formdata object to make changes | no |

A more elaborate server configuration is shown below. This configuration reveals the `timeout` property as assigned to the server object. This sets it for all end points, it can also be configured per end point.

```js
FilePond.setOptions({
    server: {
        url: 'http://192.168.0.100',
        timeout: 7000,
        process: {
            url: './process',
            method: 'POST',
            headers: {
                'x-customheader': 'Hello World'
            },
            withCredentials: false,
            onload: (response) => response.key,
            onerror: (response) => response.data,
            ondata: (formData) => {
                formData.append('Hello', 'World');
                return formData;
            }
        },
        revert: './revert',
        restore: './restore/',
        load: './load/',
        fetch: './fetch/'
    }
});
```

If we want to disable certain end points we can pass a `null` instead of string or object. Say we only want to use the fetch functionality and not do asynchronous uploading we can disable processing. In that situation revert and restore do not make a lot of sense (since we're no longer uploading temporary files) so we can remove those.

```js
FilePond.setOptions({
    server: {
        url: 'http://192.168.0.100',
        timeout: 7000,
        process: null,
        load: './load/',
        fetch: './fetch/'
    }
});
```

## Advanced

If we require even more control we can configure each end point as a function instead of an object. FilePond will then run our function and supply callback methods to control the FilePond interface.

Note that in the examples below we make use of [arrow functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions), these can of course also be written as a classic function.

### Process

The custom `process` function receives a `file` object plus a set of FilePond callback methods to return control to FilePond. The `file` parameter contains the native file object (instead of a FilePond file item) access the file item is restricted in the `process` function to prevent setting properties or running functions that would would contradict or interfere with the current processing of the file.

{{%note%}}
The `transfer` and `options` parameters are only to be used when uploading chunks. Use `transfer` to return the transfer id to FilePond, the `options` parameter contains the chunk related options.
{{%/note%}}

```js
FilePond.setOptions({
    server: {
        process:(fieldName, file, metadata, load, error, progress, abort, transfer, options) => {

            // fieldName is the name of the input field
            // file is the actual file object to send
            const formData = new FormData();
            formData.append(fieldName, file, file.name);

            const request = new XMLHttpRequest();
            request.open('POST', 'url-to-api');

            // Should call the progress method to update the progress to 100% before calling load
            // Setting computable to false switches the loading indicator to infinite mode
            request.upload.onprogress = (e) => {
                progress(e.lengthComputable, e.loaded, e.total);
            };

            // Should call the load method when done and pass the returned server file id
            // this server file id is then used later on when reverting or restoring a file
            // so your server knows which file to return without exposing that info to the client
            request.onload = function() {
                if (request.status >= 200 && request.status < 300) {
                    // the load method accepts either a string (id) or an object
                    load(request.responseText);
                }
                else {
                    // Can call the error method if something is wrong, should exit after
                    error('oh no');
                }
            };

            request.send(formData);
            
            // Should expose an abort method so the request can be cancelled
            return {
                abort: () => {
                    // This function is entered if the user has tapped the cancel button
                    request.abort();

                    // Let FilePond know the request has been cancelled
                    abort();
                }
            };
        }
    }
});
```


### Revert

Custom revert methods receive the unique server file id and a load and error callback.

```js
FilePond.setOptions({
    server: {
        revert: (uniqueFileId, load, error) => {
            
            // Should remove the earlier created temp file here
            // ...

            // Can call the error method if something is wrong, should exit after
            error('oh my goodness');

            // Should call the load method when done, no parameters required
            load();
        }
    }
});
```


### Load

Custom load methods receive the local file `source`, and the callback methods: `load`, `error`, `abort`, and `headers`.

```js
FilePond.setOptions({
    server: {
        load: (source, load, error, progress, abort, headers) => {
            // Should request a file object from the server here
            // ...

            // Can call the error method if something is wrong, should exit after
            error('oh my goodness');

            // Can call the header method to supply FilePond with early response header string
            // https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/getAllResponseHeaders
            headers(headersString);

            // Should call the progress method to update the progress to 100% before calling load
            // (endlessMode, loadedSize, totalSize)
            progress(true, 0, 1024);

            // Should call the load method with a file object or blob when done
            load(file);

            // Should expose an abort method so the request can be cancelled
            return {
                abort: () => {
                    // User tapped cancel, abort our ongoing actions here

                    // Let FilePond know the request has been cancelled
                    abort();
                }
            };
        }
    }
});
```


### Fetch

The custom fetch method receives the `url` to fetch and a set of FilePond callback methods to return control to FilePond.

```js
FilePond.setOptions({
    server: {
        fetch: (url, load, error, progress, abort, headers) => {
            // Should get a file object from the URL here
            // ...

            // Can call the error method if something is wrong, should exit after
            error('oh my goodness');

            // Can call the header method to supply FilePond with early response header string
            // https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/getAllResponseHeaders
            headers(headersString);

            // Should call the progress method to update the progress to 100% before calling load
            // (computable, loadedSize, totalSize)
            progress(true, 0, 1024);

            // Should call the load method with a file object when done
            load(file);

            // Should expose an abort method so the request can be cancelled
            return {
                abort: () => {
                    // User tapped abort, cancel our ongoing actions here

                    // Let FilePond know the request has been cancelled
                    abort();
                }
            };
        }
    }
});
```


### Restore

Custom restore methods receive the server file id of the file to restore and a set of FilePond callback methods to return control to FilePond.

```js
FilePond.setOptions({
    server: {
        restore: (uniqueFileId, load, error, progress, abort, headers) => {
            // Should get the temporary file object from the server
            // ...

            // Can call the error method if something is wrong, should exit after
            error('oh my goodness');

            // Can call the header method to supply FilePond with early response header string
            // https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/getAllResponseHeaders
            headers(headersString);

            // Should call the progress method to update the progress to 100% before calling load
            // (computable, loadedSize, totalSize)
            progress(true, 0, 1024);

            // Should call the load method with a file object when done
            load(serverFileObject);

            // Should expose an abort method so the request can be cancelled
            return {
                abort: () => {
                    // User tapped abort, cancel our ongoing actions here

                    // Let FilePond know the request has been cancelled
                    abort();
                }
            };
        };
    }
});
```


### Remove

The custom remove method receives the local file `source` and a `load` and `error` callback.

{{% warning %}}
This method is `null` by default as giving clients the power to remove files from the server in this way might not be very secure. But, because of popular demand the method has been added.
{{% /warning %}}

```js
FilePond.setOptions({
    server: {
        remove: (source, load, error) => {
            
            // Should somehow send `source` to server so server can remove the file with this source

            // Can call the error method if something is wrong, should exit after
            error('oh my goodness');

            // Should call the load method when done, no parameters required
            load();
        }
    }
});
```


## Conclusion

We can set the server location, end point paths, configure end point request parameters or override them with methods to finaly control how data is sent to the server. This gives us all the control we need to finely tune our connection to the server.