# winston-daily-rotate-file

[![NPM version][npm-image]][npm-url]

[![NPM](https://nodei.co/npm/winston-daily-rotate-file.png)](https://nodei.co/npm/winston-daily-rotate-file/)

A transport for [winston](https://github.com/winstonjs/winston) which logs to a rotating file. Logs can be rotated based on a date, size limit, and old logs can be removed based on count or elapsed days.

This fork of original project uses [@zigasebenik/file-stream-rotator](https://github.com/sebenik/file-stream-rotator) instead of original [file-stream-rotator](https://github.com/rogerc/file-stream-rotator/) module, which fixes [this bug](https://github.com/winstonjs/winston/issues/705)
_Some of the options in the 1.x versions of the transport have changed._ Please review the options below to identify any changes needed.

## Compatibility
Please note that if you are using `winston@2`, you will need to use `winston-daily-rotate-file@3`. `winston-daily-rotate-file@4` removed support for `winston@2`.

Starting with version 5.0.0 this module also emits an "error" event for all low level filesystem error cases. Make sure to listen for this event to prevent crashes in your application.

This library should work starting with Node.js 8.x, but tests are only executed for Node.js 14+. Use on your own risk in lower Node.js versions.

## Install
```
npm install @zigasebenik/winston-daily-rotate-file
```

## Options
The DailyRotateFile transport can rotate files by minute, hour, day, month, year or weekday. In addition to the options accepted by the logger, `winston-daily-rotate-file` also accepts the following options:

* **frequency:** A string representing the frequency of rotation. This is useful if you want to have timed rotations, as opposed to rotations that happen at specific moments in time. Valid values are '#m' or '#h' (e.g., '5m' or '3h'). Leaving this null relies on `datePattern` for the rotation times. (default: null)
* **datePattern:** A string representing the [moment.js date format](http://momentjs.com/docs/#/displaying/format/) to be used for rotating. The meta characters used in this string will dictate the frequency of the file rotation. For example, if your datePattern is simply 'HH' you will end up with 24 log files that are picked up and appended to every day. (default: 'YYYY-MM-DD')
* **zippedArchive:** A boolean to define whether or not to gzip archived log files. (default: 'false')
* **filename:** Filename to be used to log to. This filename can include the `%DATE%` placeholder which will include the formatted datePattern at that point in the filename. (default: 'winston.log.%DATE%')
* **dirname:** The directory name to save log files to. (default: '.')
* **stream:** Write directly to a custom stream and bypass the rotation capabilities. (default: null)
* **maxSize:** Maximum size of the file after which it will rotate. This can be a number of bytes, or units of kb, mb, and gb. If using the units, add 'k', 'm', or 'g' as the suffix. The units need to directly follow the number. (default: null)
* **maxFiles:** Maximum number of logs to keep. If not set, no logs will be removed. This can be a number of files or number of days. If using days, add 'd' as the suffix. It uses auditFile to keep track of the log files in a json format. It won't delete any file not contained in it. It can be a number of files or number of days (default: null)
* **options:** An object resembling https://nodejs.org/api/fs.html#fs_fs_createwritestream_path_options indicating additional options that should be passed to the file stream. (default: `{ flags: 'a' }`)
* **auditFile**: A string representing the path of the audit file, passed directly to [file-stream-rotator](https://github.com/rogerc/file-stream-rotator/) as `audit_file`.  If not specified, a file name is generated that includes a hash computed from the options object, and uses the `dirname` option value as the directory. (default: `<dirname>/.<optionsHash>-audit.json`)
* **utc**: Use UTC time for date in filename. (default: false)
* **extension**: File extension to be appended to the filename. (default: '')
* **createSymlink**: Create a tailable symlink to the current active log file. (default: false)
* **symlinkName**: The name of the tailable symlink. (default: 'current.log')
* **auditHashType**: Use specified hashing algorithm for audit. (default: 'sha256')
* **level**: Name of the logging level that will be used for the transport, if not specified option from `createLogger` method will be used

## Usage
``` js
  var winston = require('winston');
  require('@zigasebenik/winston-daily-rotate-file');

  var transport = new winston.transports.DailyRotateFile({
    level: 'info',
    filename: 'application-%DATE%.log',
    datePattern: 'YYYY-MM-DD-HH',
    zippedArchive: true,
    maxSize: '20m',
    maxFiles: '14d'
  });
  
  transport.on('error', error => {
    // log or handle errors here
  });

  transport.on('rotate', (oldFilename, newFilename) => {
    // do something fun
  });

  var logger = winston.createLogger({
    transports: [
      transport
    ]
  });

  logger.info('Hello World!');

```
using multiple transports
``` js
  var winston = require('winston');
  require('@zigasebenik/winston-daily-rotate-file');

  var transport1 = new winston.transports.DailyRotateFile({
    filename: 'application-%DATE%.log',
    datePattern: 'YYYY-MM-DD-HH',
    zippedArchive: true,
    maxSize: '20m',
    maxFiles: '14d'
  });

  var transport2 = new winston.transports.DailyRotateFile({
    level: 'error',
    filename: 'application-error-%DATE%.log',
    datePattern: 'YYYY-MM-DD-HH',
    zippedArchive: true,
    maxSize: '20m',
    maxFiles: '14d'
  });

  transport1.on('error', error => {
    // log or handle errors here
  });

  transport2.on('error', error => {
    // log or handle errors here
  });

  transport1.on('rotate', function(oldFilename, newFilename) {
    // do something fun
  });

  transport2.on('rotate', function(oldFilename, newFilename) {
    // do something fun
  });

  var logger = winston.createLogger({
    level: 'info'
    transports: [
      transport1, // will be used on info level
      transport2  // will be used on error level
    ]
  });

  logger.info('Hello World!');
  logger.error('Hello Error!');

```

### ES6

``` js
import  *  as  winston  from  'winston';
import  '@zigasebenik/winston-daily-rotate-file';


const transport = new winston.transports.DailyRotateFile({
  filename: 'application-%DATE%.log',
  datePattern: 'YYYY-MM-DD-HH',
  zippedArchive: true,
  maxSize: '20m',
  maxFiles: '14d'
});

transport.on('error', error => {
  // log or handle errors here
});

transport.on('rotate', (oldFilename, newFilename) => {
  // do something fun
});

const logger = winston.createLogger({
  transports: [
    transport
  ]
});

logger.info('Hello World!');
```

### Typescript

``` typescript

import * as winston from 'winston';
import DailyRotateFile from '@zigasebenik/winston-daily-rotate-file';

const transport: DailyRotateFile = new DailyRotateFile({
    filename: 'application-%DATE%.log',
    datePattern: 'YYYY-MM-DD-HH',
    zippedArchive: true,
    maxSize: '20m',
    maxFiles: '14d'
});

transport.on('error', error => {
    // log or handle errors here
});


transport.on('rotate', (oldFilename, newFilename) => {
    // do something fun
});

const logger = winston.createLogger({
    transports: [
        transport
    ]
});

logger.info('Hello World!');
```


This transport emits the following custom events:

* **new**: fired when a new log file is created. This event will pass one parameter to the callback (*newFilename*).
* **rotate**: fired when the log file is rotated. This event will pass two parameters to the callback (*oldFilename*, *newFilename*).
* **archive**: fired when the log file is archived. This event will pass one parameter to the callback (*zipFilename*).
* **logRemoved**: fired when a log file is removed from the file system. This event will pass one parameter to the callback (*removedFilename*).
* * **error**: fired when a low level filesystem error happens (e.g. EACCESS)

## LICENSE
MIT

##### AUTHOR: [Charlie Robbins](https://github.com/indexzero)
##### MAINTAINER: [Matt Berther](https://github.com/mattberther)
##### FORK_AUTHOR: [Žiga Šebenik](https://github.com/sebenik)

[npm-image]: https://badge.fury.io/js/winston-daily-rotate-file.svg
[npm-url]: https://npmjs.org/package/winston-daily-rotate-file
