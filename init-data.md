# 测试数据

```shell
mongo --port 27017

use mydb

db.auth('user', 'user')

var cnt = 0;
for(var i=0; i<1000; i++){
    var dl = [];
    for(var j=0; j<100; j++){
        dl.push({
                "bookId" : "BBK-" + i + "-" + j,
                "type" : "Revision",
                "version" : "IricSoneVB0001",
                "title" : "Jackson's Life",
                "subCount" : 10,
                "location" : "China CN Shenzhen Futian District",
                "author" : {
                      "name" : 50,
                      "email" : "RichardFoo@yahoo.com",
                      "gender" : "female"
                },
                "createTime" : new Date()
            });
      }
      cnt += dl.length;
      db.books.insertMany(dl);
      print("insert ", cnt);
}
```