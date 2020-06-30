# flysystem-google-cloud-storage

A Google Cloud Storage adapter for [flysystem](https://github.com/thephpleague/flysystem) - a PHP filesystem abstraction.

[![Author](http://img.shields.io/badge/author-@superbalist-blue.svg?style=flat-square)](https://twitter.com/superbalist)
[![Build Status](https://img.shields.io/travis/Superbalist/flysystem-google-cloud-storage/master.svg?style=flat-square)](https://travis-ci.org/Superbalist/flysystem-google-cloud-storage)
[![StyleCI](https://styleci.io/repos/44370843/shield?branch=style-ci)](https://styleci.io/repos/44370843)
[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE)
[![Packagist Version](https://img.shields.io/packagist/v/superbalist/flysystem-google-storage.svg?style=flat-square)](https://packagist.org/packages/superbalist/flysystem-google-storage)
[![Total Downloads](https://img.shields.io/packagist/dt/superbalist/flysystem-google-storage.svg?style=flat-square)](https://packagist.org/packages/superbalist/flysystem-google-storage)


## Installation

```bash
composer require gobrave/flysystem-google-storage
```

## Integrations

Want to get started quickly? Check out some of these integrations:

* Laravel - https://github.com/Superbalist/laravel-google-cloud-storage

## Config
Add a new disk to your filesystems.php config

```
'gcs' => [
    'driver' => 'gcs',
    'project_id' => env('GOOGLE_CLOUD_PROJECT_ID', 'your-project-id'),
    'key_file' => env('GOOGLE_CLOUD_KEY_FILE', null), // optional: /path/to/service-account.json
    'bucket' => env('GOOGLE_CLOUD_STORAGE_BUCKET', 'your-bucket'),
    'path_prefix' => env('GOOGLE_CLOUD_STORAGE_PATH_PREFIX', null), // optional: /default/path/to/apply/in/bucket
    'storage_api_uri' => env('GOOGLE_CLOUD_STORAGE_API_URI', null), // see: Public URLs below
    'visibility' => 'public', // optional: public|private
    'acl' => 'uniform', // uniform|fine-graded
],
```

## Usage

```php
use Google\Cloud\Storage\StorageClient;
use League\Flysystem\Filesystem;
use Superbalist\Flysystem\GoogleStorage\GoogleStorageAdapter;


/**
 * The credentials will be auto-loaded by the Google Cloud Client.
 *
 * 1. The client will first look at the GOOGLE_APPLICATION_CREDENTIALS env var.
 *    You can use ```putenv('GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json');``` to set the location of your credentials file.
 *
 * 2. The client will look for the credentials file at the following paths:
 * - windows: %APPDATA%/gcloud/application_default_credentials.json
 * - others: $HOME/.config/gcloud/application_default_credentials.json
 *
 * If running in Google App Engine, the built-in service account associated with the application will be used.
 * If running in Google Compute Engine, the built-in service account associated with the virtual machine instance will be used.
 */

$storageClient = new StorageClient([
    'projectId' => 'your-project-id',
]);
$bucket = $storageClient->bucket('your-bucket-name');

$adapter = new GoogleStorageAdapter($storageClient, $bucket);

$filesystem = new Filesystem($adapter);

/**
 * The credentials are manually specified by passing in a keyFilePath.
 */

$storageClient = new StorageClient([
    'projectId' => 'your-project-id',
    'keyFilePath' => '/path/to/service-account.json',
]);
$bucket = $storageClient->bucket('your-bucket-name');

$adapter = new GoogleStorageAdapter($storageClient, $bucket);

$filesystem = new Filesystem($adapter);

// write a file
$filesystem->write('path/to/file.txt', 'contents');

// update a file
$filesystem->update('path/to/file.txt', 'new contents');

// read a file
$contents = $filesystem->read('path/to/file.txt');

// check if a file exists
$exists = $filesystem->has('path/to/file.txt');

// delete a file
$filesystem->delete('path/to/file.txt');

// rename a file
$filesystem->rename('filename.txt', 'newname.txt');

// copy a file
$filesystem->copy('filename.txt', 'duplicate.txt');

// delete a directory
$filesystem->deleteDir('path/to/directory');

// see http://flysystem.thephpleague.com/api/ for full list of available functionality
```

## Google Storage specifics

When using a custom storage uri the bucket name will not prepended to the file path.

```php
$storageClient = new StorageClient([
    'projectId' => 'your-project-id',
]);
$bucket = $storageClient->bucket('your-bucket-name');
$adapter = new GoogleStorageAdapter($storageClient, $bucket);

// uri defaults to "https://storage.googleapis.com"
$filesystem = new Filesystem($adapter);
$filesystem->getUrl('path/to/file.txt');
// "https://storage.googleapis.com/your-bucket-name/path/to/file.txt"

// set custom storage uri
$adapter->setStorageApiUri('http://example.com');
$filesystem = new Filesystem($adapter);
$filesystem->getUrl('path/to/file.txt');
// "http://example.com/path/to/file.txt"

// You can also prefix the file path if needed.
$adapter->setStorageApiUri('http://example.com');
$adapter->setPathPrefix('extra-folder/another-folder/');
$filesystem = new Filesystem($adapter);
$filesystem->getUrl('path/to/file.txt');
// "http://example.com/extra-folder/another-folder/path/to/file.txt"
```
