# Chapter 04: 元數據 Meta

- [元數據介紹](#元數據介紹)
- [模型類開發示例](#模型類開發示例)
  - [教師資料表 `Teacher`](#教師資料表-teacher)
  - [課程資料表 `Course`](#課程資料表-course)
  - [學生資料表 `Student`](#學生資料表-student)
  - [助教資料表 `TeachingAssistant`](#助教資料表-teachingassistant)

## 元數據介紹

所謂的元數據（Meta）指的是 ORM 模型類別中不屬於資料欄的訊息，比如：資料排序方式、資料表名稱以及 Admin 選項…等。在 Django 的模型類別中添加元數據的方式，是在該模型類別下增添一個子類別 `Meta`，並在此類別中進行設定：

```python
class AddressInfo(models.Model):
    address = models.CharField(max_length=200, null=True, blank=True, verbose_name="地址")
    pid = models.ForeignKey('self', null=True, blank=True, verbose_name="自關聯")
    note = models.CharField(max_length=200, null=True, blank=True, verbose_name="說明")

    def __str__(self):
        return self.address

    class Meta:
        db_table = 'address'                    # 設置自定義資料表名稱
        ordering = ['pid']                      # 設置排序資料欄方式，須以列表方式傳入
        verbose_name = 'address'                # 設置後台顯示名稱
        verbose_name_plural = 'addresses'       # 設置後台顯示名稱（複數）
        abstract = True                         # 設置為抽象基礎類別（Abstract Base Classes）
        # permissions = [('權限', '說明')]
        # managed = False
        unique_together = ('address', 'note')   # 設置共同唯一鍵
        app_label = 'courses'                   # 設置應用標籤
        # db_tablespace                         # 設置資料表空間名稱
```

更多關於元數據的設置與說明，可以參考官方文檔 [Django Documentation | Model Meta options](https://docs.djangoproject.com/en/3.0/ref/models/options/)。

## 模型類開發示例

以下範例將會以慕課網課程資訊為範例，將會創建以下資料表：

- 教師資料表 [`Teacher`](#教師資料表-teacher)
- 課程資料表 [`Course`](#課程資料表-course)
- 學生資料表 [`Student`](#學生資料表-student)
- 助教資料表 [`TeachingAssistant`](#助教資料表-teachingassistant)

### 教師資料表 `Teacher`

```python
class Teacher(models.Model):
    """教師資料表"""
    nickname = models.CharField(max_length=30, primary_key=True, db_index=True, verbose_name="暱稱")
    introduction = models.TextField(default="無", verbose_name="簡介")
    fans = models.PositiveIntegerField(default="0", verbose_name="粉絲數")
    created_at = models.DateTimeField(auto_now_add=True, verbose_name="創建時間")
    updated_at = models.DateTimeField(auto_now=True, verbose_name="更新時間")

    class Meta:
        verbose_name = "教師資料表"
        verbose_name_plural = verbose_name

    def __str__(self):
        return self.nickname
```

### 課程資料表 `Course`

一位教師可以開授許多課程，因此與教師資料表之間為多對一關係，設置 `ForeignKey` 以 `Teacher` 資料表為外鍵。

```python
class Course(models.Model):
    """課程資料表"""
    title = models.CharField(max_length=100, primary_key=True, db_index=True, verbose_name="課程名稱")
    teacher = models.ForeignKey(Teacher, null=True, blank=True, on_delete=models.CASCADE, verbose_name="課程教師")  # 刪除級聯
    type = models.CharField(choices=((1, "實戰課程"), (2, "免費課程"), (0, "其它")), max_length=1, default=0, verbose_name="課程類型")
    price = models.PositiveSmallIntegerField(verbose_name="價格")
    volume = models.BigIntegerField(verbose_name="銷量")
    online = models.DateField(verbose_name="上線時間")
    created_at = models.DateTimeField(auto_now_add=True, verbose_name="創建時間")
    updated_at = models.DateTimeField(auto_now=True, verbose_name="更新時間")

    class Meta:
        verbose_name = "課程資料表"
        get_latest_by = "created_at"
        verbose_name_plural = verbose_name

    def __str__(self):
        return f"{self.get_type_display()}-{self.title}"
```

### 學生資料表 `Student`

不同學生可以選擇多門課程學習，因此與課程資料表之間屬於多對多關係。

```python
class Student(models.Model):
    """學生資料表"""
    nickname = models.CharField(max_length=30, primary_key=True, db_index=True, verbose_name="暱稱")
    course = models.ManyToManyField(Course, verbose_name="課程名稱")
    age = models.PositiveSmallIntegerField(verbose_name="年齡")
    gender = models.CharField(choices=((1, "男"), (2, "女"), (0, "保密")), max_length=1, default=0, verbose_name="性别")
    study_time = models.PositiveIntegerField(default="0", verbose_name="學習時長（hour）")
    created_at = models.DateTimeField(auto_now_add=True, verbose_name="創建時間")
    updated_at = models.DateTimeField(auto_now=True, verbose_name="更新時間")

    class Meta:
        verbose_name = "學生資料表"
        ordering = ['age']
        verbose_name_plural = verbose_name

    def __str__(self):
        return self.nickname
```

### 助教資料表 `TeachingAssistant`

一名教師配置一名助教，與教師彼此之間為一對一關係。

```python
class TeachingAssistant(models.Model):
    """助教資料表"""
    nickname = models.CharField(max_length=30, primary_key=True, db_index=True, verbose_name="暱稱")
    teacher = models.OneToOneField(Teacher, null=True, blank=True, on_delete=models.SET_NULL, verbose_name="教師")  # 删除置空
    hobby = models.CharField(max_length=100, null=True, blank=True, verbose_name="愛好")
    created_at = models.DateTimeField(auto_now_add=True, verbose_name="創建時間")
    updated_at = models.DateTimeField(auto_now=True, verbose_name="更新時間")

    class Meta:
        verbose_name = "助教資料表"
        db_table = "courses_assistant"
        verbose_name_plural = verbose_name

    def __str__(self):
        return self.nickname
```