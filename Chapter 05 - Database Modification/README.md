# Chapter 05: Django 數據表操作

- [Django 更改數據表](#django-更改數據表)
- [Django 數據導入](#django-數據導入)
  - [使用 Django Shell 導入數據](#使用-django-shell-導入數據)
  - [使用腳本導入數據](#使用腳本導入數據)
    - [`create()`](#create)
    - [`bulk_create()`](#bulk_create)
    - [`update_or_create()`](#update_or_create)
    - [`get_or_create()`](#get_or_create)
    - [`add()`](#add)
  - [使用 Fixtures 導入數據](#使用-fixtures-導入數據)
- [Django 數據導出](#django-數據導出)
  - [使用 Fixtures 導出數據](#使用-fixtures-導出數據)
  - [使用 PyCharm 自帶功能](#使用-pycharm-自帶功能)

## Django 更改數據表

在 Django 應用程式開發時，每當有更動模型類別之後，都會運行兩條命令：

- `python manage.py makemigrations`：會在應用的 `migrations` 文件夾下生成資料遷移腳本，此時並不會與資料庫進行交互。
- `python manage.py migrate`：根據上一命令生成的資料遷移腳本，在資料庫中生成或更新對應的資料表。除此之外也會在預設的 `django_migrations` 資料表中建立生成紀錄。

倘若要刪除一個模型類別，必須按照以下的步驟操作：

1. 刪除 `models.py` 中，對應的模型類別代碼
2. 刪除 `migrations` 文件夾中對應的資料遷移腳本
3. 刪除資料庫中 `django_migrations` 的生成紀錄
4. 刪除（`DROP`）對應的資料表

## Django 數據導入

### 使用 Django Shell 導入數據

使用 `python manage.py shell` 進入 Django Shell 後，導入資料模型，透過創建物件並保存的方式將資料存入資料庫中：

```python
In [1]: from courses.models import Teacher
In [2]: teacher = Teacher(nickname="Jack")
In [3]: teacher.save()
```

### 使用腳本導入數據

#### `create()`

不需要執行 `save()` 來創建物件，每次保存就會執行一條 SQL 語句，不建議大量增添資料。

```python
Teacher.objects.create(nickname="Terry", introduction="Python 工程師", fans=666)
Teacher.objects.create(nickname="Allen", introduction="Java 工程師", fans=123)
Teacher.objects.create(nickname="Henry", introduction="Golang 工程師", fans=818)
```

#### `bulk_create()`

執行一條 SQL 語句來存入多筆資料，通常搭配列表生成式使用。必須要注意的是：

- 不會調用模型的 `save()` 方法，並且不會發送 `pre_save` 與 `post_save` 訊號。
- 若主鍵設置為 `AutoField`，除非資料庫本身支援，否則不會像 `save()` 那樣檢索並設置主鍵屬性。
- 不適用於多表繼承中的子模型與多對多關係。

```python
Course.objects.bulk_create([Course(title=f"Python 系列課程 {i}", teacher=Teacher.objects.get(nickname="Terry"),
                                    type=random.choice((0, 1, 2)), price=random.randint(200, 300),
                                    volume=random.randint(100, 10000), online=date(2018, 10, 1))
                            for i in range(1, 5)])

Course.objects.bulk_create([Course(title=f"Java 系列課程 {i}", teacher=Teacher.objects.get(nickname="Allen"),
                                    type=random.choice((0, 1, 2)), price=random.randint(200, 300),
                                    volume=random.randint(100, 10000), online=date(2018, 6, 4))
                            for i in range(1, 4)])

Course.objects.bulk_create([Course(title=f"Golang 系列課程 {i}", teacher=Teacher.objects.get(nickname="Henry"),
                                    type=random.choice((0, 1, 2)), price=random.randint(200, 300),
                                    volume=random.randint(100, 10000), online=date(2018, 1, 1))
                            for i in range(1, 3)])
```

#### `update_or_create()`

更新物件，如果沒有找到就創建物件。這個方法會透過給定的 `kwargs` 來更新物件，其中 `defaults` 參數是一個由鍵值對所組成的字典，用於更新內容。返回值是一個由 `(object, created)` 所組成的元組，用以標識目標物件以及是否創立了新的物件。

```python
Student.objects.update_or_create(nickname="小明", defaults={"age": random.randint(18, 58), "gender": random.choice((0, 1, 2)), "study_time": random.randint(9, 999)})
Student.objects.update_or_create(nickname="小華", defaults={"age": random.randint(18, 58), "gender": random.choice((0, 1, 2)), "study_time": random.randint(9, 999)})
Student.objects.update_or_create(nickname="小偉", defaults={"age": random.randint(18, 58), "gender": random.choice((0, 1, 2)), "study_time": random.randint(9, 999)})
Student.objects.update_or_create(nickname="小豪", defaults={"age": random.randint(18, 58), "gender": random.choice((0, 1, 2)), "study_time": random.randint(9, 999)})
```

#### `get_or_create()`

查詢物件，如果沒有找到就創建物件。

```python
TeachingAssistant.objects.get_or_create(nickname="Sean", defaults={"hobby": "唱歌", "teacher": Teacher.objects.get(nickname="Terry")})
TeachingAssistant.objects.get_or_create(nickname="Dose", defaults={"hobby": "跑步", "teacher": Teacher.objects.get(nickname="Allen")})
TeachingAssistant.objects.get_or_create(nickname="Jack", defaults={"hobby": "減肥", "teacher": Teacher.objects.get(nickname="Henry")})
```

#### `add()`

透過 `add()` 來添加外鍵欄位，進行資料表的關聯。

```python
# 下例中，學生（Student）是子表而課程（Course）是父表

# 正向添加：子表關聯父表
Student.objects.get(nickname="小明").course.add(*Course.objects.filter(volume__gte=1000))
Student.objects.get(nickname="小華").course.add(*Course.objects.filter(volume__gt=5000))

# 反向添加：父表關聯子表
Course.objects.get(title="Python 系列課程 1").student_set.add(*Student.objects.filter(study_time__gte=500))
Course.objects.get(title="Python 系列課程 2").student_set.add(*Student.objects.filter(study_time__lte=500))
```

### 使用 Fixtures 導入數據

Fixtures 是一種提供初始化數據的方法，並且被 Django 的測試框架用來處理單元測試的數據。提供一個可以被 Django 的序列化（serialization）系統識別的序列化文件，Fixtures 會讀取並自動轉換成對應的模型並存入資料庫中：

```bash
$ python manage.py dumpdata APP_NAME > DATA.json
$ python manage.py loaddata <APP_NAME/fixtures/DATA.json>
```

使用 Fixtures 導入時，使用到了 Django 的序列化功能，因此不須依賴於特定的資料庫。但由於執行時需要創建物件，效率並不如執行 SQL 語句來的高，通常用於少量的資料初始化或進行資料庫平台轉換時。

## Django 數據導出

### 使用 Fixtures 導出數據

```bash
$ python manage.py dumpdata APP_NAME > DATA.json
```

### 使用 PyCharm 自帶功能

詳見 [PyCharm Document | Exporting and importing data](https://www.jetbrains.com/help/pycharm/exporting-and-importing-data.html)。