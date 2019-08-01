---
title: Hello World
date: 2019-08-01 19:47:50
tags: Start
---

# Flask Web开发实战--李辉（基础部分）



### 初始

#### pipenv

```bash
cd /your-project
pipenv install  # 创建pipfile, pipfile.lock
pipenv install flask
pipenv install python-dotenv
  # .env: 敏感信息
  # .flaskenv: 公开环境变量
  # 加载环境变量优先级：手动设置 > .env > .flaskenv
pipenv install watchdog --dev  # 在pipfile被添加到def-packages
pipenv install --dev  # 安装dev-packages
# pipenv shell 显式激活
pipenv run python hello.py
pipenv run flask run  # 默认http://127.0.0.1:5000/, 多线程
pipenv run flask run --host=0.0.0.0 --port=8000
pipenv run flask shell
pipenv graph  # 查看依赖
```

#### app.config

- 名称必须全大写，小写的变量不会被读取

- app.config.update()

#### url_for()

- url_for(‘index’)  获取 ‘/’

### http

#### 查询字符串

```python
# /hello?name=Grey&age=102

name = request.args.get('name', 'Flask')
```

#### flask routes

#### `@app.route('/colors/<any(blue, white, green): year>')`

- any之外的值返回404错误

#### 重定向

1. 302

   ```python
   @app.route('/hello')
   def hello():
       return '', 302, {'location': 'https://www.baidu.com'}
   ```

2. redirect()

   - `return redirect('https://www.baidu.com')`
   - `return redirect(url_for('hello'))`

#### 错误响应

- abort()

  ```python
  @app.route('/404')
  def not_found():
      abort(404)
  ```

#### jsonify

- 默认生成200响应

- `return jsonify(name='Li', gender='M')`
- `return jsonify({'name': 'Li', 'gender': 'M'})`

#### Flask上下文
- application context
  - current_app
  - g
- request context
  - request
  - session

#### http进阶
- 重定向回访问来源
  ```python
  return redirect(request.referrer)
  
  # return url_for('blah blah', next=request.full_path)
  return redirect(request.args.get('next'))
  ```
  
- AJAX

  ```javascript
  $(function(){
      // 选择器$('#load')对应“加载更多”按钮的id值
      $('#load').click(function(){
          $.ajax({
              url: '/more',  // 目标URL
              type: 'get',  // 请求方法
              // 返回2**响应后触发的回调函数
              success: function(data){
                  // 返回的响应值插入到页面
                  $('.body').append(data);
              }
          })
      })
  })
  ```

  
### 模板

#### 定界符

- 语句 `{%...%}`
  - 必须添加结束标签`{% endif %}`、`{% endfor %}`
- 表达式 `{{...}}`
- 注释 `{#...#}`
#### 变量

- `render_template('yours.html', data=data)`

- 模板中定义

  ```jinja2
  {% set navigation = [('/', 'Home'), ('/about', 'About')] %}
  
  {% set navigation %}
  	<li><a href="/">Home</a>
  	<li><a href="/about">About</a>
  {% endset %}
  ```

#### 辅助工具

- app.context_processor  注册模板上下文处理函数

- app.template_global  注册模板全局函数

- 过滤器

  - `{{ list|length }}`

  - `{% filter length %}` 

  - app.template_filter  注册自定义过滤器

- 文本标记安全

  ```reStructuredText
  {{ text|safe }}
  or
  text = Markup('...')
  ```
  
- 测试器

  - `{% if 变量 is 测试器 %}`
  - app.template_test  注册自定义测试器
- app.jinja_env
  
  - 键值对添加  globals、filters、tests
#### 局部模板

- 命名通常以一个下划线开始
- `{% include'_partial.html' %}`  插入全部内容

#### 宏

- 默认include会传递当前上下文，import需显式使用with context声明传入当前上下文

#### 模板继承

![jinja2_inherited](images\jinja2_inherited.png)
- 子模版 
  - `{% extends 'base.html' %}`
  - 创建同名的块，覆盖父模板的块
  - 追加 用 `{{ super() }}` 声明
#### 模板进阶

- 空白控制
  - 定界符内侧添加减号  `{%- endfor %}`会移除前空白
  - app.jinja_env.trim_blocks  删除jinja2语句后的第一个空行
  - app.jinja_env.lstrip_blocks  删除jinja2语句所在行之前的空格和制表符
  - 宏内的空白不受trim_blocks 、lstrip_blocks 的控制
  
- 静态文件

  ```jinja2
  <img src="{{ url_for('static', filename='avatar.jpg') }}" width="50">
  
  <link rel="icon" type="image/x-icon" href="{{ url_for('static', filename='favicon.ico') }}"
  ```

  - 引入顺序  jQuery -> Popper.js -> Bootstrap

- flash()

  - flash()发送的消息会存储在session，需要get_flashed_messages()获取
  - 需要配置SECRET_KEY
  - get_flashed_messages()被调用时，session存储的所有消息都会被移除
  
- 处理错误

  - @app.errorhandler(404)
  - @app.errorhandler(NameError)  # Werkzeug定义的HTTP异常类
  
- Jinja2

  - Jinja2 只有写入render_template()传入的模板才会被渲染
  - HTML自定义数据属性“data-*”, 例，data-username
    - element.dataset.username
    - element.getAttribute(‘data-username’)
    - $element.data(‘username’)
### 表单

#### flask-wtf

- 默认为每个表单启用CSRF保护，自动生成和验证CSRF令牌，需要设置SECRET_KEY

- 表单类

  ```python
  from flask_wtf import FlaskForm
  from wtforms import StringField, PasswordField, BooleanField, SubmitField
  from wtforms.validators import DataRequired, Length
  
  
  class LoginForm(FlaskForm):
      username = StringField('Username', validators=[DataRequired()])
      password = PasswordField('Password', validators=[DataRequired(), Length(8, 128)])
      remember = BooleanField('Remember me')
      submit = SubmitField('Log in')
      
  form = LoginForm()
  print(form.username())
  print(form.username.label())
  ```

  - WTFroms默认输出的HTML只包含id、name，添加额外属性：

    - render_kw

      ```python
      username = StringField('Username', render_kw={'placeholder': 'Your Username'})
      ```

      

    - 调用时传入

      ```python
      form.username(style='width: 200px;', class_='bar')
      ```

      

- 表单渲染

  ```python
  from forms import LoginForm
  @app.route('/basic')
  def basic():
      form = LoginForm()
      return render_template('basic.html', form=form)
  ```

  

#### 表单数据

- 表单数据验证

  - 客户端验证

    - H5内置的验证属性（type, required, min, max, accept）

      ```html
      <input type="text" name="username" required>
      ```

      

  - 服务器端验证

    - WTForms验证：在实例化表单时传入表单数据，然后对实例调用validate()，如验证失败，错误消息会存储到errors属性

      ```python
      form = Form()
      form.data
      form.validate()
      form.errors
      ```
    
  - 视图函数验证表单
    
    ![views_validate](images\views_validate.jpg)
    
    - Flask-WTF的validate_on_submit()方法验证POST, PUT, PATCH, DELETE

- 防止重复提交表单， PRG(Post/Redirect/Get)模式

#### 表单进阶

- 验证器

    ```python
    from wtforms.validators import ValidationError

    raise ValidationError(...)
    ```

- 文件上传
  - `<input type="file">`  不安全
    - 需要将表单的enctype属性设为" multipart/form-data"，这会告诉浏览器将上传数据发送到服务器，否则仅会把文件名作为表单数据提交
  - FileField
  - Flask-Uploads
  - 文件大小 `app.config['MAX_CONTENT_LENGTH']`
  - 通过form.[FileStorage.name].data获取FileStorage
  - 处理文件名
    - 原文件名FileStorage.filename
    - 过滤后的文件名
      - `from werkzeug import secure_filename`会过滤掉所有危险字符，但如果文件名完全由非ASCII字符组成，那么会得到一个空文件名
    - 统一重命名
      - `uuid.uuid4().hex + os.path.splitext(filename)[1]`, uuid4()不接收参数生成随机UUID
  - `app.root_path` 相当于 `os.path.abspath(os.path.dirname(__file__))`
  - FileStorage.save()
  - 视图函数通过send_from_directory()获取文件

- 多文件上传
  - `<input type="file" id="file" name="file" multiple>`
  - MultipleFileField
- Flask-CKEditor
  - `ckeditor = CKEditor(app)`
  - `<textarea></textarea>`
  - TextAreaField
- 单个表单多个提交按钮
  - 只有被单击的提交字段才会出现在request.form
  - 调用data属性时，WTForms会将提交字段的值转换为布尔值，被单击为True
- 单个页面多个表单
  - 为多个表单的提交字段设置不同的名称
  - 更简洁的方法是分离表单的渲染和验证

### 数据库

#### ORM三层映射

- 表 &rarr; Python类

- 字段 &rarr; 类属性

- 记录 &rarr; 类实例

#### Flask-SQLAlchemy

- `db = SQLAlchemy(app)`

- `sqlite:///:memory:`

- SQLite的数据库URI在Linux或macOS系统下的斜线数量是4个；在Windows系统下的URI中的斜线数量为3个。内存型数据库的斜线固定为3个。

- SQLite文件名不限定后缀

#### 数据库操作

- create

  ```python
  db.session.add()
  db.session.add_all()
  db.session.commit()
  ```

  

- read

  - `<模型类>.query.<过滤方法>.<查询方法>`

  - | filter | SQL  |
    | :----- | :--- |
    | in_    | IN   |
    | and_   | AND  |
    | or_    | OR   |

    

- update

  - 直接赋值并commit

- delete

  - `db.session.delete()`

#### 定义关系

- 一对多
  - 外键总是在“多”的一侧定义
  
  - 关系属性在“一”的一侧定义  `db.relationship()`（关系属性在关系模式的出发侧定义）
  
  - 建立关系
    - 外键字段赋值
    - 关系属性赋给实际的对象
  
  - 双向关系
    
    - 在关系的另一侧也创建一个relationship()并添加back_populates参数
    
  - backref简化关系定义
  
    - db.relationship的参数
  
    - db.backref()
  
- 多对一

- 一对一

- 多对多

  - 关联表不存储数据，只用来存储关系两侧模型的外键对应关系

  - db.relationship()secondary参数

#### 更新表

- 重新生成表  drop_all(), create_all()

- Flask-Migrate

  - `migrate = Migrate(app, db)`

    ```bash
    # 创建迁移环境
    flask db init
    # 生成迁移脚本
    flask db migrate
    # 生成的脚本中：upgrade()、downgrade()
    # 更新数据库
    flask db upgrade
    # 回滚
    flask db downgrade
    ```

    

#### 数据库进阶

- cascade参数
  - save-update,merge
  - save-update,merge,delete
  - all  (all = save-update,merge,refresh-expire,expunge,delete)
  - all,delete-orphan
    - delete-orphan必须和delete一起使用

- 事件监听
  - `@listens_for(target, identifier)`
