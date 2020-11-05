# Chapter 03: 字段類型和參數

- [常用的字段](#常用的字段)
- [關係型字段](#關係型字段)
- [字段參數](#字段參數)
  - [通用字段參數](#通用字段參數)
  - [關聯字段參數](#關聯字段參數)
- [自關聯](#自關聯)

## 常用的字段

- [`AutoField`](https://docs.djangoproject.com/en/3.0/ref/models/fields/#autofield)：自增長字段，當資料表中紀錄每增加一條時，資料欄的數值便會加一，默認數值是整數型。
- [`BigAutoField`](https://docs.djangoproject.com/en/3.0/ref/models/fields/#bigautofield)：與自增長字段類似，但允許更大的數值（64 位整數）。
- [`BinaryField`](https://docs.djangoproject.com/en/3.0/ref/models/fields/#binaryfield)：用於儲存原始二進制數據的特殊字段，預設狀況下會將 `editable` 設置為 `False`，亦不能被包含在 `ModelForm`中，由於存儲檔案或 Python 物件，因此無法手動進行輸入，必須透過視圖層或 Django Shell 來賦值。
- [`BooleanField`](https://docs.djangoproject.com/en/3.0/ref/models/fields/#booleanfield)：用於儲存布林值 `true` 或 `false`，預設狀況下的初始值為 `None`。
- [`NullBooleanField`](https://docs.djangoproject.com/en/3.0/ref/models/fields/#nullbooleanfield)：等價於使用 [`BooleanField`](https://docs.djangoproject.com/en/3.0/ref/models/fields/#booleanfield) 並設定 `null=True`。此字段不建議使用，有可能會在未來的 Django 版本中被摒棄。
- [`PositiveSmallIntegerField`](https://docs.djangoproject.com/en/3.0/ref/models/fields/#positivesmallintegerfield)、[`SmallIntegerField`](https://docs.djangoproject.com/en/3.0/ref/models/fields/#smallintegerfield)、[`PositiveIntegerField`](https://docs.djangoproject.com/en/3.0/ref/models/fields/#positiveintegerfield)、[`IntegerField`](https://docs.djangoproject.com/en/3.0/ref/models/fields/#django.db.models.IntegerField)、[`BigIntegerField`](https://docs.djangoproject.com/en/3.0/ref/models/fields/#bigintegerfield)：整數字段，分別可以存放 5, 6, 10, 11, 20 個位元組大小的整數。
- [`FloatField`](https://docs.djangoproject.com/en/3.0/ref/models/fields/#floatfield)、[`DecimalField`](https://docs.djangoproject.com/en/3.0/ref/models/fields/#decimalfield)：浮點數字段，兩者分別對應到 Python 中的 `float` 類別和 `Decimal` 類別。在 Python 中的 `float` 最大值取決於運行的平台，如果跟金錢相關的數值，建議使用 `DecimalField`。
- [`CharField`](https://docs.djangoproject.com/en/3.0/ref/models/fields/#charfield)、[`TextField`](https://docs.djangoproject.com/en/3.0/ref/models/fields/#textfield)：字串字段，分別用以儲存長度較小與長度較長的字串，並對應到 MySQL 中的 `varchar` 和 `longtext` 字段。
- [`DateField`](https://docs.djangoproject.com/en/3.0/ref/models/fields/#datefield)、[`DateTimeField`](https://docs.djangoproject.com/en/3.0/ref/models/fields/#datetimefield)、[`DurationField`](https://docs.djangoproject.com/en/3.0/ref/models/fields/#durationfield)：用以儲存時間日期，其中 `DurationField` 會儲存整數，透過 Python 中的 `timedelta` 函數庫實現。
- [`EmailField`](https://docs.djangoproject.com/en/3.0/ref/models/fields/#emailfield)、[`ImageField`](https://docs.djangoproject.com/en/3.0/ref/models/fields/#imagefield)、[`FileField`](https://docs.djangoproject.com/en/3.0/ref/models/fields/#filefield)、[`FilePathField`](https://docs.djangoproject.com/en/3.0/ref/models/fields/#filepathfield)、[`URLField`](https://docs.djangoproject.com/en/3.0/ref/models/fields/#urlfield)、[`UUIDField`](https://docs.djangoproject.com/en/3.0/ref/models/fields/#uuidfield)、[`GenericIPAddressField`](https://docs.djangoproject.com/en/3.0/ref/models/fields/#genericipaddressfield)

## 關係型字段

- [`OneToOneField`](https://docs.djangoproject.com/en/3.0/ref/models/fields/#onetoonefield)：一對一關係字段。
- [`ForeignKey`](https://docs.djangoproject.com/en/3.0/ref/models/fields/#foreignkey)：多對一關係字段，透過設置外鍵（Foreign Key）來實現。
- [`ManyToManyField`](https://docs.djangoproject.com/en/3.0/ref/models/fields/#manytomanyfield)：多對多關係字段，默認使用中間表（intermediary join table）來實做，有可能需要自行定義中間表。

## 字段參數

### 通用字段參數

- `db_column`：預設的資料表欄位名稱會對應到 ORM 類別的屬性名稱，如果要自定義欄位名稱時需要代入 `db_column` 參數。
- `primary_key`：在一張關係型資料庫的資料表中，會設置一個唯一並具有識別性的欄位作為主鍵（Primary Key），需要在 ORM 中使用 `primary_key=True` 來標識。
- `verbose_name`：設置資料欄位的別名，通常作為備註使用。
- `unique`：當設置為 `True` 時，欄位中的資料不會重複。
- `help_text`：表單中的說明。
- `null` 和 `blank`：用來標識欄位值是否可以為空，其中 `null` 是資料庫層面是否為空，而 `blank` 為前端表單提交時是否為空。因此千萬不能設置成 `null=False, blank=True`。
- `db_index`：給字段建立索引。
- `editable`：是否可以更改。

### 關聯字段參數

- `related_name`：用於反向查詢，透過父表查詢到子表的訊息。
- `on_delete`：當外鍵所關聯的物件被刪除時，需要進行什麼操作。可以選用的操作有：
  - `CASCADE`：為當前 Django 的預設操作。將模擬 SQL 中的 `ON DELETE CASCADE` 約束，將定義有外鍵的模型物件同時刪除！
  - `PROTECT`：阻止上述的 `CASCADE` 刪除操作，但拋出 `ProtectedError` 異常。
  - `SET_NULL`：將外鍵字段設置為 `null`，只有當字段設置了 `null=True` 時才能使用。
  - `SET_DEFAULT`：將外鍵字段設置為預設值，只有當字段設置了 `default` 參數時才能使用。
  - `DO_NOTHING`：什麼也不做。
  - `SET()`：設置為一個傳遞給 `SET()` 的值或者一個回調函數的返回值，注意大小寫。

## 自關聯

**自關聯（Self-referential Relationships）** 又稱作 **自連接（Self Joins）**，是指資料表中互相關聯的紀錄存在相同的欄位中，這樣我們就可以直接用表內關聯將外來鍵關聯設定成自身資料表的欄位。比如：使用者評論，每條評論都可能有子評論，但每條評的欄位內容應該都是相同的，並且每條評論都只有一個父評論，在這個例子中，父評論為關聯欄位，可以對應多個子評論，這就是一對多的自關聯。

在 Django 中使用自關聯的方式是將 `ForeignKey` 字段的 `to` 欄位設置為 `self` 或本身的類別名稱：

```python
class AddressInfo(models.Model):
    address = models.CharField(max_length=200, null=True, blank=True, verbose_name="地址")
    pid = models.ForeignKey('self', null=True, blank=True, verbose_name="自關聯")
    # pid = models.ForeignKey('AddressInfo', null=True, blank=True, verbose_name="自關聯")

    def __str__(self):
        return self.address
```

上述的範例是省、市與縣的地址資料，不同的省份和縣市所儲存的內容和欄位都相同，所以應該存放於同一張資料表下，但每一筆紀錄之間又相互關聯或有隸屬關係，比如：唐山市隸屬於河北省，路安區隸屬於唐山市。這樣的關係可以儲存為以下的資料表：

| `id` | `address` | `pid` |
| :--: | :--: | :--: |
| `7` | 河北省 | `null` |
| `8` | 石家莊市 | `7` |
| `9` | 唐山市 | `7` |
| `10` | 路安區 | `9` |
| `11` | 安徽省 | `null` |
| `12` | 合肥市 | `11` |
| `13` | 瑤海區 | `12` |
| `14` | 盧陽區 | `12` |