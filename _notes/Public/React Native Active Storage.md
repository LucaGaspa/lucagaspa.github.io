---
title: How to integrate Expo with Ruby on Rails Active Storage
notetype: feed
date: 10-01-2022
---

## Intro

In this article I'll explain how to upload an image with Expo to your favourite bucket thanks to Active Storage, the default image uploading utility in Ruby on Rails.
Moreover, check out [How to Use ActiveStorage Outside of a Rails View](https://cameronbothner.com/activestorage-beyond-rails-views/), the starting point of my article that let me understand Active Storage flow. Feel free to comment and discuss opening an issue in my [GitHub repo](https://github.com/LucaGaspa/lucagaspa.github.io/issues).

## Setup

To complete the later implementation, I end up with two basic utilities, inspired by [btoa](https://github.com/davidchambers/Base64.js/blob/master/base64.js) and [hex2a](https://stackoverflow.com/a/3745677) functions. Any equivalent implementation can be used.

## Implementation

### Create a blob

Active Storage expects the following POST request:

```bash
POST /rails/active_storage/direct_uploads HTTP/1.1
Content-Type: application/json

{
  "blob": {
    "filename": "my_file.jpeg",
    "content_type": "image/jpeg",
    "byte_size": 1020123,
    "checksum": base_64_of_md5_hash(fileObject)
  }
}
```

To create this JSON object in Expo, we need the file URI that usually is provided by the `expo-image-picker` through the OS directly for images. We set the MIME type, the size of the file and the checksum. Ruby on Rails accepts base64 encoded md5 signatures. We can rely on `expo-file-system` package to to the job, as follows:

```javascript
import * as FileSystem from "expo-file-system"

const fileURI = "path-to-the-image"
const info = await FileSystem.getInfoAsync(fileURI, { size: true, md5: true })
let checksum = btoa(hex2a(info.md5)) // convert hex md5 to a plain string and then into base64

const blob = {
  filename: fileURI.split("/").pop(),
  byte_size: info.size,
  content_type: "image/jpeg",
  checksum: checksum,
}
```

### Call Ruby on Rails direct_uploads route

Cal the `direct_uploads` Rails route with your preferred networking library.

```javascript
let response = await fetch(`your_rails_server_url/direct_uploads`, {
  method: "POST",
  headers: {
    Accept: "application/json",
    "Content-Type": "application/json",
    Authorization: `Bearer ${token}`, // I set up JWT as authenticate mechanism
  },
  body: JSON.stringify({ blob: blob }),
})
let json = await response.json()
```

### Upload the file to the generated signed URL and save the signed_id

```javascript
const bucketURL = json.direct_upload?.url
const headers = json.direct_upload?.headers
const imageSignedId = json.signed_id

if (!bucketURL || !headers) {
  throw new Error("active_storage_no_upload_url")
}

let uploadResponse = await FileSystem.uploadAsync(bucketURL, fileURI, {
  httpMethod: "PUT",
  headers: headers,
})
if (uploadResponse.status < 200 || uploadResponse.status >= 300) {
  throw new Error("active_storage_upload_put")
}
```

### Update the Rails model to assign your signed_id to the attachment relation

Call the PUT route of the model you want the image to be assigned to. For example, we want to update the user avatar.

```bash
PUT /users/1.json HTTP/1.1
Content-Type: application/json

{
  "user": {
    "avatar": signed_id
  }
}
```

```ruby
class User < ApplicationRecord
  has_one_attached :avatar
end
```

### More

If you want to upload a list of images, just create N blobs and call direct_uploads N times. Once you have N signed_ids, call the PUT of the model passing the list of signed_ids, using `has_many_attached` on it.

```ruby
class Post < ApplicationRecord
  has_many_attached :images
end
```
