---
title: "Uploading Files to AWS S3"
slug: "uploading-files"
---

```js
app.post('/pets', upload.single('coverImg'), function (req, res, next) {    
  var pet = new Pet(req.body);
  pet.save(function (err) {      
    if (req.file) {
      client.upload(req.file.path, {}, function (err, versions, meta) {
        if (err) { return res.status(400).send({ err: err }) };

        versions.forEach(function(image) {
          // console.log(image.width, image.height, image.url);
          // 1024 760 https://my-bucket.s3.amazonaws.com/path/110ec58a-a0f2-4ac4-8393-c866d813b8d1.jpg

          var urlArray = image.url.split('-');
          urlArray.pop();
          var url = urlArray.join('-');
          pet.coverImgUrl = url;
          pet.save();
        });

        res.send({ pet: pet });
      });
    } else {
      res.send({ pet: pet });
    }
  }
}
```
