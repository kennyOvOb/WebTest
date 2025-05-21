# django-admin startproject <项目名称> <目标目录>
創建project

# python manage.py startapp <应用名称>

創建app
当你执行 `python manage.py startapp polls` 时，Django 会：
1. 在你的项目中创建一个新的目录叫做 `polls`
2. 在这个目录中自动生成以下文件结构：

```
polls/
├── __init__.py
├── admin.py      # 用于管理后台配置
├── apps.py       # 应用的配置文件
├── models.py     # 定义数据模型
├── tests.py      # 用于编写测试
├── views.py      # 处理视图逻辑
└── migrations/   # 数据库迁移文件目录
    └── __init__.py
```
每个文件的作用：
- `models.py`: 定义数据库模型（表结构）
- `views.py`: 包含处理用户请求的逻辑
- `admin.py`: 配置 Django 管理后台的显示
- `apps.py`: 存放应用的配置
- `tests.py`: 编写单元测试
- `migrations/`: 存放数据库迁移文件

创建应用后，你需要：
1. 在 `mysite/settings.py` 的 `INSTALLED_APPS` 列表中注册这个应用
2. 开始在各个文件中编写你的应用逻辑

这种结构的好处是：
1. 模块化：每个应用都是独立的功能模块
2. 可重用：应用可以在不同的 Django 项目中重复使用
3. 维护性：代码组织清晰，易于维护

例如，如果你正在制作一个博客网站，你可能会有：
- `polls`: 投票功能
- `blog`: 博客文章功能
- `users`: 用户管理功能

每个功能都可以作为独立的应用来开发和管理。

# urls配置
```python
from django.urls import path  # 从django.urls导入path函数，这是用来定义URL模式的主要工具
from . import views   # noqa       # 从当前目录导入views模块，其中包含了处理请求的视图函数

urlpatterns = [  # urlpatterns是Django查找URL匹配时会用到的列表
    path("", views.index, name="index"),  # 定义一个URL模式
]
```
让我们详细分析 `path("", views.index, name="index")` 这一行：
1. 第一个参数 `""`：
    - 这是URL模式字符串
    - 空字符串表示根路径
    - 如果这个 urls.py 是在 polls 应用中，并且在主 urls.py 中配置为 `path('polls/', include('polls.urls'))`
    - 那么这个视图将响应 `/polls/` 的访问

2. 第二个参数 `views.index`：
    - 指向处理这个URL的视图函数
    - 对应你的 views.py 中的 index 函数
    - 当用户访问匹配的URL时，Django 会调用这个函数
    - 在你的代码中，这个函数返回 "Hello, world. You're at the polls index."

3. 第三个参数 `name="index"`：
    - 给这个URL模式一个名字
    - 这个名字可以在 Django 模板或视图中用来生成URL
    - 使用命名URL是Django的最佳实践，因为：
        - 如果将来需要修改URL结构，只需要修改urls.py，而不需要修改所有使用这个URL的地方
        - 可以使用 `{% url 'index' %}` 在模板中引用这个URL
        - 可以使用 `reverse('index')` 在Python代码中引用这个URL

实际例子：
1. 当用户访问 `http://你的域名/polls/` 时：
    - Django 会匹配到这个URL模式
    - 调用 `views.index` 函数
    - 返回 "Hello, world. You're at the polls index."

2. 在模板中使用这个URL：
```
<a href="{% url 'index' %}">访问首页</a>
```

1. 在Python程式中使用這個URL：
```python
from django.urls import reverse
url = reverse('index')  # 獲取URL路徑
```
這種URL配置方式體現了Django的幾個設計理念：
1. URLs和視圖的解耦
2. 清晰的URL結構
3. 可維護性和靈活性
4. 程式碼的可重用性

當你在開發過程中，這種結構可以讓你：
- 更容易管理和組織URL
- 方便地修改URL結構而不影響其他程式碼
- 在不同的地方重複使用相同的URL
- 更好地維護和擴展你的專案

# python manage.py migrate

1. **套用資料庫遷移**：
    - 執行尚未執行的資料庫遷移（migrations）
    - 根據 models.py 中定義的模型建立或修改資料庫表格
    - 更新資料庫結構以匹配你的模型變更

2. **追蹤已執行的遷移**：
    - Django 會記錄哪些遷移已經執行過
    - 確保不會重複執行相同的遷移
    - 在 db.sqlite3 資料庫中有一個 `django_migrations` 表格來追蹤這些訊息

常見使用時機：
1. **首次設定專案時**：
    - 建立 Django 預設的資料表（如用戶認證相關的表格）
    - 初始化資料庫結構

2. **新增應用程式後**：
    - 建立新應用程式定義的資料表
    - 套用新應用程式的初始遷移

3. **修改模型後**：
    - 當你修改了 models.py 中的模型定義
    - 執行 `python manage.py makemigrations` 產生遷移檔案後
    - 需要用 `migrate` 來實際套用這些變更

實際運作流程：
1. 檢查所有已安裝的應用程式
2. 讀取每個應用程式的 migrations 資料夾
3. 比對資料庫中已執行的遷移記錄
4. 依序執行尚未執行的遷移

例如，當你：
```python
from django.db import models

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')
```
執行順序應該是：
1. 先執行 `python manage.py makemigrations`（建立遷移檔案）
2. 再執行 `python manage.py migrate`（套用遷移到資料庫）

重要提示：
1. 在開發過程中要經常執行 migrate
2. 修改模型後一定要記得執行
3. 部署到生產環境前也要確保所有遷移都已執行
4. 最好在修改資料庫結構前先備份資料

這個命令是 Django 專案開發中非常重要的一部分，它確保了你的資料庫結構始終與你的程式碼保持同步。

# `python manage.py makemigrations polls`

1. **指令組成**：
    - `python manage.py`：Django 項目管理工具
    - ：創建遷移檔案的命令 `makemigrations`
    - ：指定要創建遷移的應用名稱 `polls`

2. **工作流程**：
``` 
   模型修改 -> makemigrations -> 生成遷移檔案 -> migrate -> 更新資料庫
```
1. **實際操作過程**：
    - 檢查 polls/models.py 中的更改
    - 在 polls/migrations/ 目錄下創建新的遷移檔案
    - 記錄模型的變化

### 使用時機
1. **首次創建模型後**：
``` python
   # polls/models.py
   class Question(models.Model):
       question_text = models.CharField(max_length=200)
       pub_date = models.DateTimeField('發布日期')
```
1. **修改現有模型時**：
    - 添加新欄位
    - 修改欄位屬性
    - 刪除欄位

### 執行結果
當你執行這個命令時，可能會看到類似這樣的輸出：
``` 
Migrations for 'polls':
  polls/migrations/0001_initial.py
    - Create model Question
    - Create model Choice
```
### 注意事項
1. **執行順序**：
    - 先修改 models.py
    - 執行 makemigrations
    - 最後執行 migrate

2. **重要提示**：
    - 遷移檔案要納入版本控制
    - 每次修改模型後都需要執行
    - 遷移檔案會按順序執行

3. **常見問題**：
    - 如果遇到錯誤，檢查模型定義是否正確
    - 確保應用已在 settings.py 中註冊

### 最佳實踐
1. **經常性遷移**：
    - 每次修改模型後立即創建遷移
    - 不要積累太多未遷移的更改

2. **遷移檔案管理**：
    - 給遷移檔案做好註釋
    - 保持遷移歷史的清晰

3. **測試**：
    - 在應用到生產環境前先在開發環境測試
    - 確保遷移可以正確執行

### 實際開發流程示例：
1. 修改模型：
``` python
   # polls/models.py
   class Question(models.Model):
       question_text = models.CharField(max_length=200)
       pub_date = models.DateTimeField('發布日期')
       author = models.CharField(max_length=100)  # 新增作者欄位
```
1. 創建遷移：
``` bash
   python manage.py makemigrations polls
```
1. 查看將要執行的 SQL：
``` bash
   python manage.py sqlmigrate polls 0001
```
1. 應用遷移：
``` bash
   python manage.py migrate
```
這個命令是 Django 開發中非常重要的一部分，它幫助我們管理資料庫結構的變化，使得資料庫修改變得更加安全和可控。


# 
编辑 models.py 文件，改变模型。

运行 python manage.py makemigrations 为模型的改变生成迁移文件。

运行 python manage.py migrate 来应用数据库迁移。

# 查詢

### 常見查詢語法速查表
1. 
``` python
__exact      # 精確匹配
__iexact     # 不區分大小寫的精確匹配
__contains   # 包含
__icontains  # 不區分大小寫的包含
__in         # 在列表中
__gt        # 大於
__gte       # 大於等於
__lt        # 小於
__lte       # 小於等於
__startswith # 以...開始
__endswith   # 以...結束
__range      # 範圍查詢
__year      # 年份
__month     # 月份
__day       # 日期
__isnull    # 是否為空

```
