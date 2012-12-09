jquery-file-upload-middleware
=============================

jQuery-File-Upload Express.js middleware. Based on the server code of [jQuery-File-Upload](https://github.com/blueimp/jQuery-File-Upload)

Installation:

```
    $ npm install jquery-file-upload-middleware
```

Usage:

```javascript
    var express = require("express"),
        upload = require('jquery-file-upload-middleware');

    var app = express();

    // configure upload middleware
    upload.configure({
        uploadDir: __dirname + '/public/uploads',
        uploadUrl: '/uploads',
        imageVersions: {
            thumbnail: {
                width: 80,
                height: 80
            }
        }
    });

    app.configure(function () {
        ...
        app.use('/upload', upload.fileHandler());
        app.use(express.bodyParser());
        ...
    });

```

On the frontend:

```html
   <input id="fileupload" type="file" name="files[]" data-url="/upload" multiple>
   <script>$('#fileupload').fileupload({ dataType: 'json' })</script>
```

Overriding global configuration

```javascript

    app.use('/upload2', upload.fileHandler({
        uploadDir: __dirname + '/public/uploads2',
        uploadUrl: '/uploads2',
        imageVersions: {
            thumbnail: {
                width: 100,
                height: 100
            }
        }
    }));

```

More sophisticated example - Events

```javascript
        app.use('/upload', upload.fileHandler());

        // events
        upload.on('begin', function (fileInfo) { ... });
        upload.on('abort', function (fileInfo) { ... });
        upload.on('end', function (fileInfo) {
            // fileInfo structure is the same as returned to browser
            // { 
            //     name: '3 (3).jpg',
            //     originalName: '3.jpg',
            //     size: 79262,
            //     type: 'image/jpeg',
            //     delete_type: 'DELETE',
            //     delete_url: 'http://yourhost/upload/3%20(3).jpg',
            //     url: 'http://yourhost/uploads/3%20(3).jpg',
            //     thumbnail_url: 'http://youhost/uploads/thumbnail/3%20(3).jpg' 
            // }
        });
        upload.on('error', function (e) {
            console.log(e.message);
        });
```

Dynamic upload directory and url, isolating user files:

```javascript
        upload.configure({
            imageVersions: {
                thumbnail: {
                    width: 80,
                    height: 80
                }
            }
        });

        app.use('/upload', function (req, res, next) {
            // imageVersions are taken from upload.configure()
            upload.fileHandler({
                uploadDir: function () {
                    return __dirname + '/public/uploads/' + req.sessionID
                },
                uploadUrl: function () {
                    return '/uploads/' + req.sessionID
                }
            })(req, res, next);
        });
```

Getting uploaded files mapped to their fs locations:

```javascript
        app.use('/list', function (req, res, next) {
            upload.getFiles({
                uploadDir: function () {
                    return __dirname + '/public/uploads/' + req.sessionID
                },
                uploadUrl: function () {
                    return '/uploads/' + req.sessionID
                }
            }, function (files) {
                //  {
                //      "00001.MTS": {
                //          "path": "/home/.../public/uploads/ekE6k4j9PyrGtcg+SA6a5za3/00001.MTS"
                //      },
                //      "DSC00030.JPG": {
                //          "path": "/home/.../public/uploads/ekE6k4j9PyrGtcg+SA6a5za3/DSC00030.JPG",
                //          "thumbnail": "/home/.../public/uploads/ekE6k4j9PyrGtcg+SA6a5za3/thumbnail/DSC00030.JPG"
                //      }
                //  }
                res.json(files);
            });
        });
```

Passing uploaded files down the request chain:

```javascript
        app.use('/api', function (req, res, next) {
            upload.getFiles({
                uploadDir: function () {
                    return __dirname + '/public/uploads/' + req.sessionID
                },
                uploadUrl: function () {
                    return '/uploads/' + req.sessionID
                }
            }, function (files) {
                res.jquploadfiles = files;
                next();
            });
        });
```

Other options and their default values:

```javascript
{
    tmpDir: '/tmp',
    uploadDir: __dirname + '/public/uploads',
    uploadUrl: '/uploads',
    maxPostSize: 11000000000, // 11 GB
    minFileSize: 1,
    maxFileSize: 10000000000, // 10 GB
    acceptFileTypes: /.+/i,
    imageTypes: /\.(gif|jpe?g|png)$/i,
    imageVersions: {
        thumbnail: {
            width: 80,
            height: 80
        }
    },
    accessControl: {
        allowOrigin: '*',
        allowMethods: 'OPTIONS, HEAD, GET, POST, PUT, DELETE'
    }
}
```

## License
Copyright (c) 2012 [Aleksandr Guidrevitch](http://aguidrevitch.blogspot.com/)
Released under the [MIT license](http://www.opensource.org/licenses/MIT).
