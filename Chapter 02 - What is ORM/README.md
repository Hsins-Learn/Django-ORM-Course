# Chapter 02: ORM 介紹

  - [ORM 介紹](#orm-介紹)

## ORM 介紹

在使用物件導向概念進行軟體開發的過程中，基於對現實的需求，通常在物件進行操作之後，會需要對資料庫進行增刪改查的動作。所謂的 **物件關係映射（ORM, Object Relational Mapping）** 就是一種將物件導向概念與關係型資料庫中表的概念進行對應，提供對資料庫的映射而不需要使用 SQL 直接編碼，提昇開發效率。

<p align="center">
  <img src="https://user-images.githubusercontent.com/26391143/72669177-bc19b580-3a69-11ea-8f17-1e53f04c0d21.png">
</p>

如上圖所示，ORM 是業務邏輯與資料庫之間的橋樑。當開發者定義了一個物件，就對應到資料庫中的一張表；而物件的實例，也就對應著資料表中的一筆紀錄。在 Django 中，模型類別會定義在每一個應用的 `model.py` 文件下。舉例來說，下面的代碼將會對應到資料庫中的資料表 `User`，其中有一個欄位 `name` 並且類型為最大長度 `255` 的字元型態：

```python
from django.db import models


class User(models.Model):
    name = models.CharField(max_length=255)
```

在開發時使用 ORM 可以提昇開發效率，但仍舊存在缺點，比如：

- 無法實現較為複雜的查詢
- 無法進行資料表的分表操作
- 無法確定 Detached/Attached 狀態

關於何時該使用 ORM，可以參考這篇 [When Should I Use ORM](http://mikehadlow.blogspot.com/2012/06/when-should-i-use-orm.html)。
