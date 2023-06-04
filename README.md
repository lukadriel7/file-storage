```
 ____|_) |       ___|  |
 |     | |  _ \\___ \  __|  _ \   __| _` |  _` |  _ \
 __|   | |  __/      | |   (   | |   (   | (   |  __/
_|    _|_|\___|_____/ \__|\___/ _|  \__,_|\__, |\___|
A file system abstraction for Node.js     |___/
```

[![Test](https://github.com/googlicius/file-storage/actions/workflows/ci.yml/badge.svg)](https://github.com/googlicius/file-storage/actions/workflows/ci.yml) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A simple abstraction to interact with file system inspired by [Laravel File System](https://laravel.com/docs/8.x/filesystem), provide one interface for many kind of drivers: `local`, `ftp`, `sftp`, `Amazon S3`, and `Google Cloud Storage`, even your custom driver.

## Installation

```bash
$ yarn add @file-storage/core @file-storage/common

# Or npm
$ npm install @file-storage/core @file-storage/common
```

And upload a file to local, it will be stored in `storage` folder in your root project directory by default:

```javascript
import Storage from '@file-storage/core';

// Upload from file path.
Storage.put('/my-image.png', '/path/of/destination/my-image.png');

// Or from a read-stream/buffer:
Storage.put(stream, '/path/of/destination/my-image.png');
```

## Configuration

By default only local driver is supported. To use another driver, you need to install corresponding package:

- Amazon S3: `yarn add @file-storage/s3`
- FTP: `yarn add @file-storage/ftp`
- SFTP: `yarn add @file-storage/sftp`
- Google Cloud Storage: `yarn add @file-storage/gcs`

If there is no configuration, it will uploads to local disk. You can specific yours by using `config` method:

```typescript
import Storage from '@file-storage/core';
import S3Driver, { S3DiskConfig } from '@file-storage/s3';
import LocalDriver, { LocalDiskConfig } from '@file-storage/local';

const localDisk: LocalDiskConfig = {
  driver: LocalDriver,
  name: 'local',
  root: 'public',
};

const s3Disk: S3DiskConfig = {
  driver: S3Driver,
  name: 'mys3',
  bucketName: 'mybucket',
  // Uncomment if you want specify credentials manually.
  // region: 'ap-southeast-1',
  // credentials: {
  //   accessKeyId: '123abc',
  //   secretAccessKey: '123abc',
  // },
};

Storage.config({
  // Default disk that you can access directly via Storage facade.
  defaultDiskName: 'mys3',
  diskConfigs: [localDisk, s3Disk],
});

// Somewhere in your code...
// Get file from s3:
Storage.get('/path/to/s3-bucket/my-image.png');
```

## Unique file name

Enable unique file name to prevent a file get replaced when uploading same file (or same name).
The unique name generated by `uuid` to secure your file path.

```javascript
Storage.config({
  ...
  uniqueFileName: true,
});

// The uploaded path could be like this: /path/to/e8a3e633-fc7f-4dde-b7f0-d2686bcd6836.jpeg
```

## Obtain specific disk:

To interact with a specific disk instead of the default, use `disk` method:

```typescript
Storage.disk('local').get('/path/to/local/my-image.png');

// To adjust the configuration on the fly, you can specify the settings in the second argument:
Storage.disk('local', { uniqueFileName: false }).put(...);
```

## Create your custom driver

If built-in drivers doesn't match your need, just defines a custom driver by extends `Driver` abstract class:

```typescript
import Storage from '@file-storage/core';
import { Driver, DiskConfig } from '@file-storage/common';

interface MyCustomDiskConfig extends DiskConfig {
  driver: typeof MyCustomDriver;
  ...
}

class MyCustomDriver extends Driver {
  constructor(config: MyCustomDiskConfig) {
    super(config);
    ...
  }

  // Implement all Driver's methods here.
}

```

And provide it to Storage.diskConfigs:

```typescript
Storage.config<MyCustomDiskConfig>({
  diskConfigs: [
    {
      driver: MyCustomDriver,
      name: 'myCustomDisk',
      ...
    }
  ],
});
```

## Image manipulation

To upload image and also creates many diferent sizes for web resonsive, install this package, it is acting as a plugin, will generates those images automatically. Images will be generated if the size reach given breakpoints. We provide 3 breakpoints by default: large: 1000, medium: 750, small: 500. And the thumbnail is also generaged by default.

```bash
$ yarn add @file-storage/image-manipulation
```

And provide it to Storage config:

```typescript
import ImageManipulation from '@file-storage/image-manipulation';

Storage.config({
  ...
  plugins: [ImageManipulation],
});
```

#### Image manipulation customize

You can customize responsive formats and thumbnail size:

```typescript
import ImageManipulation from '@file-storage/image-manipulation';

ImageManipulation.config({
  breakpoints: {
    size1: 500,
    size2: 800,
  },
  thumbnailResizeOptions: {
    width: 333,
    height: 222,
    fit: 'contain',
  },
});
```

## TODO

- [x] Create interface for all result (Need same result format for all drivers).
- [x] Refactor `customDrivers` option: provides disk defination is enough.
- [x] Implement GCS disk.
- [ ] Put file from a local path.
- [ ] API section: detailed of each driver.
- [x] Remove `customDrivers` option, pass custom driver class directly to `diskConfigs.driver`.
- [x] Unique file name.
- [x] Update `aws-sdk` to v3.
- [x] Replace `request` module with another module as it was deprecated.
- [ ] Remove deprecated: BuiltInDiskConfig, DriverName.

## License

MIT
