# Flask-RESTful
Flask-RESTful是Flask的扩展，它增加了快速构建REST API的支持。它是一个轻量级抽象，可以与您现有的ORM /库一起工作。Flask-RESTful鼓励使用最少的设置实现最佳实践。如果你对Flask本身很熟悉的话，Flask-RESTful就很容易上手。
## <span id = "anchor0">用户指南</span>
这一部分文档将想你展示如何开始在Flask中使用Flask-RESTful
* [安装](#anchor1)
* [快速上手](#anchor2)
    1. [一个最简单的API示例程序](#anchor2.1)
    2. [资源路由](#anchor2.2)
    3. [端点](#anchor2.3)
    4. [参数解析](#anchor2.4)
    5. [数据格式化](#anchor2.5)
    6. [完整例子](#anchor2.6)
* [请求信息的解析](#anchor3)
    1. [基础参数的解析](#anchor3.1)
    2. [必要参数](#anchor3.2)
    3. [多值和列表](#anchor3.3)
    4. [参数改名](#anchor3.4)
    5. [从不同的对象中解析参数](#anchor3.5)
    6. [从多个对象中解析参数](#anchor3.6)
    7. [解析器的继承](#anchor3.7)
    8. [错误处理](#anchor3.8)
    9. [错误消息](#anchor3.9)
* [输出字段](#anchor4)
    1. [基本用法](#anchor4.1)
    2. [重命名属性](#anchor4.2)
    3. [缺省值](#anchor4.3)
    4. [自定义字段和多值](#anchor4.4)
    5. [URL和其他具体的字段](#anchor4.5)
    6. [复杂的结构体](#anchor4.6)
    7. [列表字段](#anchor4.7)
    8. [进阶：嵌套字段](#anchor4.8)
* [扩展 Flask-RESTful](#anchor5)
    1. [内容协商](#anchor5.1)
    2. [自定义字段与输入](#anchor5.2)
    3. [响应格式](#anchor5.3)
    4. [资源方法装饰器](#anchor5.4)
    5. [自定义错误处理](#anchor5.5)
* [进阶使用方法](#anchor6)
    1. [项目的结构](#anchor6.1)
    2. [使用蓝图](#anchor6.2)
    3. [完整的参数解析示例](#anchor6.3)
    4. [将构造函数作为参数传递给资源](#anchor6.4)

## <span id = "anchor1">安装</span>
使用 pip 安装Flask-RESTful
> pip install flask-restful

开发版的程序可以从GitHub上下载.
> git clone https://github.com/flask-restful/flask-restful.git

> cd flask-restful

> python setup.py develop

下面是Flask-RESTful的依赖组件,使用pip安装的时候会自动安装.

* Flask version 0.8 or greater

Flask-RESTful 支持的Python版本包括 2.6, 2.7, 3.3, 3.4, 3.5 or 3.6.

## <span id = "anchor2">快速上手</span>
现在可以开始编写你的第一个REST API程序了,这个手册假设已经是使用并且了解Flask了，而且你已经安装了Flask和Flask-RESTful.如果没有请根据[安装](#anchor1)这个小节的指引完成必要的步骤。

[返回目录](#anchor0)
## <span id = "anchor2.1">一个最简单的API示例程序</span>
一个最简单的Flask-RESTful API程序:
```
from flask import Flask
from flask_restful import Resource, Api

app = Flask(__name__)
api = Api(app)

class HelloWorld(Resource):
    def get(self):
        return {'hello': 'world'}

api.add_resource(HelloWorld, '/')

if __name__ == '__main__':
    app.run(debug=True)
```
将代码保存为api.py并使用python解释器运行. 注意,我们已经开启了Flask的调试模式,这样可以提供代码的重载和详细的错误信息.
```
$ python api.py
 * Running on http://127.0.0.1:5000/
 * Restarting with reloader
```
> 警告：调试模式不应当被应用于生产环境中!

现在可以开启终端，用命令工具curl测试你的API
```
$ curl http://127.0.0.1:5000/
{"hello": "world"}
```
[返回目录](#anchor0)

## <span id = "anchor2.2">资源路由</span> 
Flask-RESTful提供的主要代码块被称为资源(Resources),资源是构建在Flask顶层的可拔插的视图之上的，你可以通过多种HTTP方法访问你所定义的资源,下面的例子就是一个基础的CRUD资源访问示例.
```
from flask import Flask, request
from flask_restful import Resource, Api

app = Flask(__name__)
api = Api(app)

todos = {}

class TodoSimple(Resource):
    def get(self, todo_id):
        return {todo_id: todos[todo_id]}

    def put(self, todo_id):
        todos[todo_id] = request.form['data']
        return {todo_id: todos[todo_id]}

api.add_resource(TodoSimple, '/<string:todo_id>')

if __name__ == '__main__':
    app.run(debug=True)
```
你可以这样测试:
```
$ curl http://localhost:5000/todo1 -d "data=Remember the milk" -X PUT
{"todo1": "Remember the milk"}
$ curl http://localhost:5000/todo1
{"todo1": "Remember the milk"}
$ curl http://localhost:5000/todo2 -d "data=Change my brakepads" -X PUT
{"todo2": "Change my brakepads"}
$ curl http://localhost:5000/todo2
{"todo2": "Change my brakepads"}
```
或者在安装有requests库的python交互式命令窗口中:
```
>>> from requests import put, get
>>> put('http://localhost:5000/todo1', data={'data': 'Remember the milk'}).json()
{u'todo1': u'Remember the milk'}
>>> get('http://localhost:5000/todo1').json()
{u'todo1': u'Remember the milk'}
>>> put('http://localhost:5000/todo2', data={'data': 'Change my brakepads'}).json()
{u'todo2': u'Change my brakepads'}
>>> get('http://localhost:5000/todo2').json()
{u'todo2': u'Change my brakepads'}
```
Flask-RESTful 可以理解多种从视图方法中返回的数值类型. 和Flask一样, 你可以返回可迭代对象,并且将其转换成响应,包括原始的Flask响应对象. Flask-RESTful同样支持将响应的返回代码，响应的头信息等一次性返回,就像下面的例子一样:
```
class Todo1(Resource):
    def get(self):
        # Default to 200 OK
        return {'task': 'Hello world'}

class Todo2(Resource):
    def get(self):
        # Set the response code to 201
        return {'task': 'Hello world'}, 201

class Todo3(Resource):
    def get(self):
        # Set the response code to 201 and return custom headers
        return {'task': 'Hello world'}, 201, {'Etag': 'some-opaque-string'}
```
[返回目录](#anchor0)

## <span id = "anchor2.3">端点</span>
很多时候，在一个API中, 你的资源需要有多个URLs. 你可以使用add_resource()方法将多个URLs添加到Api对象上.每一个URL将会指向到不同的资源.
```
api.add_resource(HelloWorld,
    '/',
    '/hello')
```
你也可以将部分路径用变量匹配的方式添加资源中.
```
api.add_resource(Todo,
    '/todo/<int:todo_id>', endpoint='todo_ep')
```
> 注意
>
> 如果请求无法匹配你的应用程序定义的端点, Flask-RESTful将会返回404错误代码,并且在错误消息中建议其他相近的匹配端点信息.如果要禁止这样的情况，你可以在程序的配置中将ERROR_404_HELP置为False.

[返回目录](#anchor0)

## <span id = "anchor2.4">参数解析</span>
虽然Flask提供了获取请求数据(比如查询字符串或者POST表单的编码数据)比较方便的方法,但是验证数据的有效性还是比较痛苦的。Flask-RESTful有一个内置的请求数据验证库，类似于argparse。
```
from flask_restful import reqparse

parser = reqparse.RequestParser()
parser.add_argument('rate', type=int, help='Rate to charge for this resource')
args = parser.parse_args()
```
> 注意
> 
> 与argparse模块不同，reqparse.RequestParser.parse_args() 返回Python的字典结构，而不是自定义的数据结构。
> 使用reqparse模块还可以免费提供正常的错误消息。如果一个参数不能通过验证，那么Flask-restful将会响应400号错误和一个突出显示的错误响应。

```
$ curl -d 'rate=foo' http://127.0.0.1:5000/todos
{'status': 400, 'message': 'foo cannot be converted to int'}
```
输入模块提供了一些通用转换函数，比如inputs.date() 和 inputs.url()。
调用parse_args函数时，如果参数strict=True那么一旦请求数据中发现未定义的数据就会抛出错误。
```
args = parser.parse_args(strict=True)
```
[返回目录](#anchor0)

## <span id = "anchor2.5">数据格式化</span>
默认情况下，返回可迭代的所有字段都将按原样呈现。虽然这在处理Python数据结构时非常有用，但在处理对象时可能会非常令人沮丧。为了解决这个问题，Flask-restful提供了字段模块和marshal\_with()装饰器。与Django ORM和WTForm类似，可以使用字段模块来描述响应的结构。
```
from flask_restful import fields, marshal_with

resource_fields = {
    'task':   fields.String,
    'uri':    fields.Url('todo_ep')
}

class TodoDao(object):
    def __init__(self, todo_id, task):
        self.todo_id = todo_id
        self.task = task

        # This field will not be sent in the response
        self.status = 'active'

class Todo(Resource):
    @marshal_with(resource_fields)
    def get(self, **kwargs):
        return TodoDao(todo_id='my_todo', task='Remember the milk')
```
上面的示例使用一个python对象，并准备将其序列化。marshal\_with()装饰器可以完成resource\_fields所描述的转换。程序只需要从对象中提取指定的惟一字段即可。fields.Url字段是一个特殊字段，它接受端点名称，并在响应中为该端点生成Url。您需要的许多字段类型已经包括在模块内，具体的可以通过fields的指南产看完整的清单。
[返回目录](#anchor0)
## <span id = "anchor2.6">完整例子</span>
```
# Save this example in api.py

from flask import Flask
from flask_restful import reqparse, abort, Api, Resource

app = Flask(__name__)
api = Api(app)

TODOS = {
    'todo1': {'task': 'build an API'},
    'todo2': {'task': '?????'},
    'todo3': {'task': 'profit!'},
}


def abort_if_todo_doesnt_exist(todo_id):
    if todo_id not in TODOS:
        abort(404, message="Todo {} doesn't exist".format(todo_id))

parser = reqparse.RequestParser()
parser.add_argument('task')


# Todo
# shows a single todo item and lets you delete a todo item
class Todo(Resource):
    def get(self, todo_id):
        abort_if_todo_doesnt_exist(todo_id)
        return TODOS[todo_id]

    def delete(self, todo_id):
        abort_if_todo_doesnt_exist(todo_id)
        del TODOS[todo_id]
        return '', 204

    def put(self, todo_id):
        args = parser.parse_args()
        task = {'task': args['task']}
        TODOS[todo_id] = task
        return task, 201


# TodoList
# shows a list of all todos, and lets you POST to add new tasks
class TodoList(Resource):
    def get(self):
        return TODOS

    def post(self):
        args = parser.parse_args()
        todo_id = int(max(TODOS.keys()).lstrip('todo')) + 1
        todo_id = 'todo%i' % todo_id
        TODOS[todo_id] = {'task': args['task']}
        return TODOS[todo_id], 201

##
## Actually setup the Api resource routing here
##
api.add_resource(TodoList, '/todos')
api.add_resource(Todo, '/todos/<todo_id>')


if __name__ == '__main__':
    app.run(debug=True)
```
运行示例
```
$ python api.py
 * Running on http://127.0.0.1:5000/
 * Restarting with reloader
```
GET task Lists
```
$ curl http://localhost:5000/todos
{"todo1": {"task": "build an API"}, "todo3": {"task": "profit!"}, "todo2": {"task": "?????"}}
```
GET a task
```
$ curl http://localhost:5000/todos/todo3
{"task": "profit!"}
```
DELETE a task
```
$ curl http://localhost:5000/todos/todo2 -X DELETE -v

> DELETE /todos/todo2 HTTP/1.1
> User-Agent: curl/7.19.7 (universal-apple-darwin10.0) libcurl/7.19.7 OpenSSL/0.9.8l zlib/1.2.3
> Host: localhost:5000
> Accept: */*
>
* HTTP 1.0, assume close after body
< HTTP/1.0 204 NO CONTENT
< Content-Type: application/json
< Content-Length: 0
< Server: Werkzeug/0.8.3 Python/2.7.2
< Date: Mon, 01 Oct 2012 22:10:32 GMT
```
Add a new task
```
$ curl http://localhost:5000/todos -d "task=something new" -X POST -v

> POST /todos HTTP/1.1
> User-Agent: curl/7.19.7 (universal-apple-darwin10.0) libcurl/7.19.7 OpenSSL/0.9.8l zlib/1.2.3
> Host: localhost:5000
> Accept: */*
> Content-Length: 18
> Content-Type: application/x-www-form-urlencoded
>
* HTTP 1.0, assume close after body
< HTTP/1.0 201 CREATED
< Content-Type: application/json
< Content-Length: 25
< Server: Werkzeug/0.8.3 Python/2.7.2
< Date: Mon, 01 Oct 2012 22:12:58 GMT
<
* Closing connection #0
{"task": "something new"}
```
Update a task
```
$ curl http://localhost:5000/todos/todo3 -d "task=something different" -X PUT -v

> PUT /todos/todo3 HTTP/1.1
> Host: localhost:5000
> Accept: */*
> Content-Length: 20
> Content-Type: application/x-www-form-urlencoded
>
* HTTP 1.0, assume close after body
< HTTP/1.0 201 CREATED
< Content-Type: application/json
< Content-Length: 27
< Server: Werkzeug/0.8.3 Python/2.7.3
< Date: Mon, 01 Oct 2012 22:13:00 GMT
<
* Closing connection #0
{"task": "something different"}
```
[返回目录](#anchor0)

## <span id = "anchor3.0">请求信息的解析</span>
>警告
>
>Flask-RESTful的整个请求解析器部分将会被删除，并将被替换为能更好地执行输入/输出功能，并且能与其他软件包整合集成的软件包（如marshmallow）。
>这意味着它将保持到2.0，但认为它已被弃用。不要担心，如果你现在有这样的代码，并希望继续这样做，它不会马上消失。
>Flask-RESTful的请求解析接口reqparse在argparse接口之后建模。它旨在提供对Flask中flask.request对象的任何变量的简单和统一的访问。

[返回目录](#anchor0)
## <span id = "anchor3.1">基础参数的解析</span>
下面是一个简单的请求解析实例.在flask.Request.values的字典中有两个参数，一个是整数类型，一个是字符串类型。
```
from flask_restful import reqparse

parser = reqparse.RequestParser()
parser.add_argument('rate', type=int, help='Rate cannot be converted')
parser.add_argument('name')
args = parser.parse_args()
```
> 注意
>
> 默认参数类型是一个unicode字符串。在python3中是str，在python2中的unicode。如果指定了help的值，则在分析类型错误时会将其作为错误消息。 如果你不指定help消息，则默认行为是将类型错误的原始信息作为错误消息。 可以通过查看错误消息了解更多详情。
>
> 默认情况下，参数不是必需的。此外，如果请求中提供的参数不是RequestParser定义的参数将被忽略。
> 在请求解析器中声明但未在请求中设置的参数将默认为None。

[返回目录](#anchor0)
## <span id = "anchor3.2">必要参数</span>
如果一个参数传递的值是不可或缺的，就在add_argument()函数中添加required=True参数.
```
parser.add_argument('name', required=True, help="Name cannot be blank!")
```
[返回目录](#anchor0)
## <span id = "anchor3.3">多个参数及列表的解析</span>

如果你要接收的参数是由相同key组成的list，你可以增加action='append'参数
```
parser.add_argument('name', action='append')
```
可以用这种方式请求
```
curl http://api.example.com -d "name=bob" -d "name=sue" -d "name=joe"
```

你看到的结果:
```
args = parser.parse_args()
args['name']    # ['bob', 'sue', 'joe']
```
[返回目录](#anchor0)
## <span id = "anchor3.4">参数改名</span>
如果有一些原因，你需要改变一些被解析的参数在字典中的关键字，可以使用dest参数
```
parser.add_argument('name', dest='public_name')

args = parser.parse_args()
args['public_name']
```

[返回目录](#anchor0)
## <span id = "anchor3.5">从不同的对象中解析参数</span>
在缺省状态下，RequestParser会尝试从flask.Request.values和 flask.Request.json对象中解析具体数值。也可以在add_argument()中指定具体参数的来源，例如：
```
# Look only in the POST body
parser.add_argument('name', type=int, location='form')

# Look only in the querystring
parser.add_argument('PageSize', type=int, location='args')

# From the request headers
parser.add_argument('User-Agent', location='headers')

# From http cookies
parser.add_argument('session_id', location='cookies')

# From file uploads
parser.add_argument('picture', type=werkzeug.datastructures.FileStorage, location='files')
```
> 注意
>
> 在使用location='json'参数时，只能type=list

[返回目录](#anchor0)
## <span id = "anchor3.6">从多个对象中解析参数</span>
如果多个参数位于不同的对象中，可以将多个对象放置于一个列表中
```
parser.add_argument('text', location=['headers', 'values'])
```
当这种解析方式被指定时，从不同对象中解析的参数会组合成一个MultiDict，在列表中最后的对象解析结果会出现在整个结果的最前面。
如果这个列表中包含headers，那么参数就不会是大小写敏感，而是首字母大写（str.title()的效果）,如果指定location='headers'，并不是一个列表，那就会返回的参数大小写敏感。

[返回目录](#anchor0)
## <span id = "anchor3.7">解析器的继承</span>
你经常会为不同的资源编写不同的解析器，但是问题在于有些解析器有共同的参数。解决的方法是你可以编写一个父一级的解析器，专门负责解析所有共同的参数，然后使用copy()函数拓展出不同的解析器。你也可以通过在父解析器的replace_argument()函数中覆盖一些参数，或者干脆使用的replace_argument()函数将参数全部移除。
```
from flask_restful import reqparse

parser = reqparse.RequestParser()
parser.add_argument('foo', type=int)

parser_copy = parser.copy()
parser_copy.add_argument('bar', type=int)

# parser_copy has both 'foo' and 'bar'

parser_copy.replace_argument('foo', required=True, location='json')
# 'foo' is now a required str located in json, not an int as defined
#  by original parser

parser_copy.remove_argument('foo')
# parser_copy no longer has 'foo' argument
```
[返回目录](#anchor0)
## <span id = "anchor3.8">错误处理</span>
在缺省状态下，处理RequestParser的错误方式是在第一个错误出现时就放弃。如果处理参数的时间比较长，这种处理方式是有好处的。然而如果能将所有出错信息一起返回给客户端，当然是更好了，只需要在生成RequestParser对象时使用bundle_errors=True选项。
```
from flask_restful import reqparse

parser = reqparse.RequestParser(bundle_errors=True)
parser.add_argument('foo', type=int, required=True)
parser.add_argument('bar', type=int, required=True)

# If a request comes in not containing both 'foo' and 'bar', the error that
# will come back will look something like this.

{
    "message":  {
        "foo": "foo error message",
        "bar": "bar error message"
    }
}

# The default behavior would only return the first error

parser = RequestParser()
parser.add_argument('foo', type=int, required=True)
parser.add_argument('bar', type=int, required=True)

{
    "message":  {
        "foo": "foo error message"
    }
}
```
全局使用 “BUNDLE_ERRORS”. 
```
from flask import Flask

app = Flask(__name__)
app.config['BUNDLE_ERRORS'] = True
```
> 警告
>
>BUNDLE_ERRORS 是全局设置，会覆盖所有RequestParser实例的bundle_errors设置。

[返回目录](#anchor0)
## <span id = "anchor3.9">错误消息</span>
每个字段的错误信息可以通过自定义help参数的值来指定。
如果没有提供help参数，字段错误将由字段本身的类型错误字符串来代替。help参数值可以包含 {error_msg}这样可改写的令牌，其中的值是字段类型错误的具体信息。这样可以在定制错误信息的同时保留原始的错误信息。
```
from flask_restful import reqparse

parser = reqparse.RequestParser()
parser.add_argument(
    'foo',
    choices=('one', 'two'),
    help='Bad choice: {error_msg}'
)

# If a request comes in with a value of "three" for `foo`:

{
    "message":  {
        "foo": "Bad choice: three is not a valid choice",
    }
}
```
[返回目录](#anchor0)
## <span id = "anchor4">输出字段</span>
输出字段提供了一种简单的方法来控制您在响应中实际呈现的数据。使用字段模块，您可以在资源中使用您想要的任何对象（ORM模型/自定义类等）。字段还允许格式化和过滤响应，这样就不必担心暴露内部数据结构。

在查看代码时，也会清楚地显示将呈现什么样的数据以及如何格式化数据。

[返回目录](#anchor0)
## <span id = "anchor4.1">基本用法</span>
你可以定义一个字典或有序字典（OrderedDict）类型的字段，其键值是渲染对象的属性或对象的键值，其值是该字段将被格式化并返回的值。这个例子有三个字段：两个是字符串一个是日期，被格式化为RFC 822的日期字符串（ISO 8601也是支持的）
```
from flask_restful import Resource, fields, marshal_with

resource_fields = {
    'name': fields.String,
    'address': fields.String,
    'date_updated': fields.DateTime(dt_format='rfc822'),
}

class Todo(Resource):
    @marshal_with(resource_fields, envelope='resource')
    def get(self, **kwargs):
        return db_get_todo()  # Some function that queries the db
```
这个例子假设你有一个自定义的数据库对象（todo），有name，address，和date_updated三个属性。对象的其他属性都被视为是私有属性，不会在输出中呈现。指定envelope参数可以用来包装输出结果。

装饰器 marshal_with的作用是使用数据对象和使用字段的过滤器。编排可以作用在单个对象，字典或者是对象的列表、。

> 注意
>
> marshal_with 是一个方便的装饰，这在功能上等效于下面的代码

```
class Todo(Resource):
    def get(self, **kwargs):
        return marshal(db_get_todo(), resource_fields), 200
```
这个明确的表达式可用于返回HTTP状态码，不仅仅是成功的响应的状态码200（见abort()错误）。

[返回目录](#anchor0)
## <span id = "anchor4.2">重命名属性</span>
经常公开字段的名称与内部字段名称不一致，可以通过使用attribute参数进行映射。
```
fields = {
    'name': fields.String(attribute='private_name'),
    'address': fields.String,
}
```
lambda函数或者其他可调用函数也可以被指定为attribute值
```
fields = {
    'name': fields.String(attribute=lambda x: x._private_name),
    'address': fields.String,
}
```
attribute也可以使用嵌套的属性值
```
fields = {
    'name': fields.String(attribute='people_list.0.person_dictionary.name'),
    'address': fields.String,
}
```
[返回目录](#anchor0)
## <span id = "anchor4.3">缺省值</span>
如果因为一些原因你的数据对象中没有字段列表中的规定属性，你可以指定这些属性的缺省值用以代替None
```
fields = {
    'name': fields.String(default='Anonymous User'),
    'address': fields.String,
}
```

[返回目录](#anchor0)
## <span id = "anchor4.4">自定义字段和多值</span>
有时您有自己的自定义格式化需求。你可以编写继承自fields.Raw类的子类，并完备format函数。对于一个属性值中保存多个信息的情况，这种方式非常有帮助。比如不同bit位的值代表不同的含义，可以将多个字段复用到一个属性中，并且多路输出值。
下面的例子假定flags属性的比特位1代表“Normal” 或者 “Urgent”，比特位2代表 “Read” or “Unread”，这样的存储非常方便，但是对于可读性而言，最好是将其转换为独立的字符串字段。
```
class UrgentItem(fields.Raw):
    def format(self, value):
        return "Urgent" if value & 0x01 else "Normal"

class UnreadItem(fields.Raw):
    def format(self, value):
        return "Unread" if value & 0x02 else "Read"

fields = {
    'name': fields.String,
    'priority': UrgentItem(attribute='flags'),
    'status': UnreadItem(attribute='flags'),
}
```
[返回目录](#anchor0)
## <span id = "anchor4.5">URL和其他具体的字段</span>
Flask-RESTful 包含一个特殊字段fields.Url。它根据请求的资源合成一个URI。
这也是一个很好的例子，说明如何将数据添加到您的响应中，而这些数据实际上并不存在于数据对象上。
```
class RandomNumber(fields.Raw):
    def output(self, key, obj):
        return random.random()

fields = {
    'name': fields.String,
    # todo_resource is the endpoint name when you called api.add_resource()
    'uri': fields.Url('todo_resource'),
    'random': RandomNumber,
}
```
缺省状态下 fields.Url返回一个相对的URI，在字段声明的时候使用absolute=True关键字参数可以生成一个包含协议命（http:// https://）主机名，端口号的绝对URI，如果要修改协议可以使用scheme关键字参数

```
fields = {
    'uri': fields.Url('todo_resource', absolute=True)
    'https_uri': fields.Url('todo_resource', absolute=True, scheme='https')
}
```
[返回目录](#anchor0)
## <span id = "anchor4.6">复杂结构体</span>
marshal()函数可以将扁平的数据结构转换成为嵌套结构
```
>>> from flask_restful import fields, marshal
>>> import json
>>>
>>> resource_fields = {'name': fields.String}
>>> resource_fields['address'] = {}
>>> resource_fields['address']['line 1'] = fields.String(attribute='addr1')
>>> resource_fields['address']['line 2'] = fields.String(attribute='addr2')
>>> resource_fields['address']['city'] = fields.String
>>> resource_fields['address']['state'] = fields.String
>>> resource_fields['address']['zip'] = fields.String
>>> data = {'name': 'bob', 'addr1': '123 fake street', 'addr2': '', 'city': 'New York', 'state': 'NY', 'zip': '10468'}
>>> json.dumps(marshal(data, resource_fields))
'{"name": "bob", "address": {"line 1": "123 fake street", "line 2": "", "state": "NY", "zip": "10468", "city": "New York"}}'
```
> 注意
> 
> address字段在数据对象中并不真实存在，但是它的子字段可以直接从数据对象中获取，就好像它们没有被嵌套。

[返回目录](#anchor0)
## <span id = "anchor4.7">字段列表</span>
你也可以重新编排，将字段转换成列表。
```
>>> from flask_restful import fields, marshal
>>> import json
>>>
>>> resource_fields = {'name': fields.String, 'first_names': fields.List(fields.String)}
>>> data = {'name': 'Bougnazal', 'first_names' : ['Emile', 'Raoul']}
>>> json.dumps(marshal(data, resource_fields))
>>> '{"first_names": ["Emile", "Raoul"], "name": "Bougnazal"}'
```
[返回目录](#anchor0)
## <span id = "anchor4.8">进阶:嵌套字段</span>
使用字典的嵌套结构字段可以扁平化转换成嵌套的响应，你可以用Nested函数重新编排数据转换成嵌套的数据，并且以合适的方式展现出来。
```
>>> from flask_restful import fields, marshal
>>> import json
>>>
>>> address_fields = {}
>>> address_fields['line 1'] = fields.String(attribute='addr1')
>>> address_fields['line 2'] = fields.String(attribute='addr2')
>>> address_fields['city'] = fields.String(attribute='city')
>>> address_fields['state'] = fields.String(attribute='state')
>>> address_fields['zip'] = fields.String(attribute='zip')
>>>
>>> resource_fields = {}
>>> resource_fields['name'] = fields.String
>>> resource_fields['billing_address'] = fields.Nested(address_fields)
>>> resource_fields['shipping_address'] = fields.Nested(address_fields)
>>> address1 = {'addr1': '123 fake street', 'city': 'New York', 'state': 'NY', 'zip': '10468'}
>>> address2 = {'addr1': '555 nowhere', 'city': 'New York', 'state': 'NY', 'zip': '10468'}
>>> data = { 'name': 'bob', 'billing_address': address1, 'shipping_address': address2}
>>>
>>> json.dumps(marshal_with(data, resource_fields))
'{"billing_address": {"line 1": "123 fake street", "line 2": null, "state": "NY", "zip": "10468", "city": "New York"}, "name": "bob", "shipping_address": {"line 1": "555 nowhere", "line 2": null, "state": "NY", "zip": "10468", "city": "New York"}}'
```
此示例使用了两个嵌套字段。Nested函数将对个字段组成的一个字典结构渲染成一个子字段。Nested函数构造的和嵌套字典（先前的例子）之间非常重要的不同是属性的上下文。在这个例子中billing_address是一个复杂对象，它拥有自己的字段和上下文，上下文是传入到嵌套字段的子对象，而不是原始的数据对象。换句话说，data.billing_address.addr1是一个作用域，而在前面一个例子中data.addr1是本地属性。记住，Nested和List对象，属性将产生新的作用域。
 
使用Nested和List函数重新编排更多复杂对象的列表:
```
user_fields = {
    'id': fields.Integer,
    'name': fields.String,
}

user_list_fields = {
    fields.List(fields.Nested(user_fields)),
}
```
[返回目录](#anchor0)
## <span id = "anchor5">扩展Flask-RESTful</span>
我们意识到每个人在REST框架中都有不同的需求。Flask-RESTful试图尽可能灵活，但有时你会发现内置的功能不足以满足您的需求。Flask-RESTful中有几个不同的扩展点，在这种情况下会有帮助。

[返回目录](#anchor0)
## <span id = "anchor5.1">内容协商</span>
Content Negotiation
开箱即用，Flask-RESTful仅仅配置成支持JSON格式。我们做出这个决定是为了给API的维护者对API支持格式完全的控制权。所以今后你不必支持那些要用CVS格式的人，甚至你都不知道他们的存在。为了在API中增加一些媒介类型，你需要声明在API对象中需要支持的格式。
```
app = Flask(__name__)
api = Api(app)

@api.representation('application/json')
def output_json(data, code, headers=None):
    resp = make_response(json.dumps(data), code)
    resp.headers.extend(headers or {})
    return resp
```
这些 representation 函数必须返回一个Flask 响应对象。

> 注意
> 
> Flask-RESTful 使用Python标准库中的json模块，没有使用flask.json 是因为Flask JSON的序列化模块的兼容性不能满足JSON标准。如果你的程序需要个性化定制，你可以使用上面提到的Flask JSON模块代替默认的JSON模块。
> 在应用的配置中，可以通过配置提供的 RESTFUL_JSON属性来配置默认的Flask-RESTful JSON 模块。这个设置是一个字典，其中的键值与json.dumps()的关键参数相对应。

```
class MyConfig(object):
    RESTFUL_JSON = {'separators': (', ', ': '),
                    'indent': 2,
                    'cls': MyCustomEncoder}
```
> 注意
>
> 如果程序运行在调试模式下 (app.debug = True) 而且键值sort_keys 或者 indent 没有RESTFUL_JSON配置中声明 Flask-RESTful提供True 和 4 作为缺省值。

[返回目录](#anchor0)
## <span id = "anchor5.2">自定义字段与输入</span>
对Flask-REST最常见的补充之一就是是根据自己的数据类型自定义类型或字段。

字段
自定义输出字段可以让你在不直接修改内部对象的情况下用自己的方式展现输出格式。你所需要做的就是子类Raw，并且完备format()方法。
```
class AllCapsString(fields.Raw):
    def format(self, value):
        return value.upper()


# example usage
fields = {
    'name': fields.String,
    'all_caps_name': AllCapsString(attribute=name),
}
```
输入
为了解析参数，你可能需要进行自定义的验证。创建自己的输入类型可以使你方便的扩展请求的解析器。
```
def odd_number(value):
    if value % 2 == 0:
        raise ValueError("Value is not odd")

    return value
```
请求解析器也可以在出错信息中需要使用name参数的参考值的情况下，直接获取name参数。
```
def odd_number(value, name):
    if value % 2 == 0:
        raise ValueError("The parameter '{}' is not odd. You gave us the value: {}".format(name, value))

    return value
```
你也可以将公共参数的值转换成内部的表现形式
You can also convert public parameter values to internal representations:
```
# maps the strings to their internal integer representation
# 'init' => 0
# 'in-progress' => 1
# 'completed' => 2

def task_status(value):
    statuses = [u"init", u"in-progress", u"completed"]
    return statuses.index(value)
Then you can use these custom input types in your RequestParser:

parser = reqparse.RequestParser()
parser.add_argument('OddNumber', type=odd_number)
parser.add_argument('Status', type=task_status)
args = parser.parse_args()
```
[返回目录](#anchor0)
## <span id = "anchor5.3">响应格式</span>
为了支持其他表现形式(xml,csv,html)，你可以使用representation()这个装饰器。你需要在程序中增加引用。
```
api = Api(app)

@api.representation('text/csv')
def output_csv(data, code, headers=None):
    pass
    # implement csv output!
```
这些输出函数包含三个参数data，code，和headers。
data是你的资源方法返回的对象，code是HTTP预计的状态码,headers是在响应中设置的HTTP headers。你的输出函数应当返回一个flask.Response对象。

```
def output_json(data, code, headers=None):
    """Makes a Flask response with a JSON encoded body"""
    resp = make_response(json.dumps(data), code)
    resp.headers.extend(headers or {})
    return resp
```
另外一种实现的方式是创建一个继承自Api类的一个子类，并在这个类中提供你自己的输出函数。
```
class Api(restful.Api):
    def __init__(self, *args, **kwargs):
        super(Api, self).__init__(*args, **kwargs)
        self.representations = {
            'application/xml': output_xml,
            'text/html': output_html,
            'text/csv': output_csv,
            'application/json': output_json,
        }
```
[返回目录](#anchor0)
## <span id = "anchor5.4">资源方法装饰器</span>
在资源类中有一个名为method_decorator的属性。您可以对资源进行子类化，并添加您自己的装饰器，这些装饰器将被添加到资源中的所有方法函数中。例如，如果您想要在每个请求中构建定制的身份验证。

```
def authenticate(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        if not getattr(func, 'authenticated', True):
            return func(*args, **kwargs)

        acct = basic_authentication()  # custom account lookup function

        if acct:
            return func(*args, **kwargs)

        flask_restful.abort(401)
    return wrapper


class Resource(flask_restful.Resource):
    method_decorators = [authenticate]   # applies to all inherited resources
```
或者，你可以指定一个字典，将HTTP的请求方法与装饰器进行映射，装饰器仅对匹配的请求方式进行处理。
```
def cache(f):
    @wraps(f)
    def cacher(*args, **kwargs):
        # caching stuff
    return cacher

class MyResource(restful.Resource):
    method_decorators = {'get': [cache]}

     def get(self, *args, **kwargs):
        return something_interesting(*args, **kwargs)

     def post(self, *args, **kwargs):
        return create_something(*args, **kwargs)
```
在这个例子中caching装饰器仅会应用于GET请求，对POST请求不起作用。
既然Flask-RESTful 资源是实际上的Flask视图对象，你也可以使用标准的Flask视图装饰器。

[返回目录](#anchor0)
## <span id = "anchor5.5">自定义错误处理</span>
错误处理是一个棘手的问题。你的Flask应用可能戴了很多顶帽子，然而你使用正确的内容类型和错误语法对这些Flask-RESTful错误进行处理。
Flask-RESTful在路由发生400或者500号错误时会调用handle\_error()函数。比如你想让你的应用在发生404 Not Found errors时能以正确的媒体类型返回错误信息，在这种情况下在Api的构造函数中使用catch\_all\_404参数。
```
app = Flask(__name__)
api = flask_restful.Api(app, catch_all_404s=True)
```
然后Flask-RESTful将会在它的路由上处理错误时增加404错误。

Sometimes you want to do something special when an error occurs - log to a file, send an email, etc. Use the got_request_exception() method to attach custom error handlers to an exception.
有时，当出现错误时，您想要做一些特别的事情——在文件中记录日志、发送电子邮件等等。使用got\_request\_exception()方法将定制错误处理程序附加到异常。
```
def log_exception(sender, exception, **extra):
    """ Log an exception to our logging framework """
    sender.logger.debug('Got exception during processing: %s', exception)

from flask import got_request_exception
got_request_exception.connect(log_exception, app)
```
自定义错误信息。
当在请求中遇到某些错误时，你可能希望返回一个特定的消息和/或状态代码。你可以告诉Flask-RESTful如何处理每个错误/异常，这样你的API代码就不必到处使用try/except块了。

```
errors = {
    'UserAlreadyExistsError': {
        'message': "A user with that username already exists.",
        'status': 409,
    },
    'ResourceDoesNotExist': {
        'message': "A resource with that ID no longer exists.",
        'status': 410,
        'extra': "Any extra information you want.",
    },
}
```
包含‘status’ 键值就会设置响应的状态码，如果没有指定，缺省值就是500。
如果定义了errors字典，可以传入到Api的构造函数中使用。
```
app = Flask(__name__)
api = flask_restful.Api(app, errors=errors)
```

[返回目录](#anchor0)
## <span id = "anchor6">进阶使用方法</span>
这个部分包括基于Flask-RESTful，构建一个稍微复杂一点的应用程序的方法介绍，它包含基于Flask-RESTful-base的API建立一个真实使用的应用时的一些最佳实践方案。


[返回目录](#anchor0)
## <span id = "anchor6.1">项目结构</span>
有很多不同的方式来组织你的应用，但是这里我们将描述一个在较大的应用中能很好的扩展，并且保持一个很好的级别组织的项目结构。基本的想法是将你的应用分成三个主要部分:路由、资源和其他公共基础设施。
下面是一个示例目录结构:
```
myapi/
    __init__.py
    app.py          # this file contains your app and routes
    resources/
        __init__.py
        foo.py      # contains logic for /Foo
        bar.py      # contains logic for /Bar
    common/
        __init__.py
        util.py     # just some common infrastructure
```
公共目录可能只包含一组助手函数来满足您的应用程序中的公共需求。它还可以包含任何定制的输入/输出类型，您的资源需要完成这项工作。在资源文件中，您只拥有资源对象。这就是foo.py的样子:
```
from flask_restful import Resource

class Foo(Resource):
    def get(self):
        pass
    def post(self):
        pass
```
相对应的 app.py:
```
from flask import Flask
from flask_restful import Api
from myapi.resources.foo import Foo
from myapi.resources.bar import Bar
from myapi.resources.baz import Baz

app = Flask(__name__)
api = Api(app)

api.add_resource(Foo, '/Foo', '/Foo/<str:id>')
api.add_resource(Bar, '/Bar', '/Bar/<str:id>')
api.add_resource(Baz, '/Baz', '/Baz/<str:id>')
```
正如你想象的，在一个特别大或者特别复杂的API中，这个文件作为一个资源和路由的综合列表，在你的应用中是非常有价值的。你也可以使用这个文件设置一些配置参数(before\_request(),after\_request())。一般来讲，这个文件配置了你的API的入口。
公共目录中的内容只是您想要支持资源模块的内容。

[返回目录](#anchor0)
## <span id = "anchor6.2">使用蓝图</span>
你可以在Flask文档中看到有蓝图的模块应用程序，你需要了解蓝图是什么以及为什么要使用它们。下面是一个如何将Api链接到Blueprint的示例。
```
from flask import Flask, Blueprint
from flask_restful import Api, Resource, url_for

app = Flask(__name__)
api_bp = Blueprint('api', __name__)
api = Api(api_bp)

class TodoItem(Resource):
    def get(self, id):
        return {'task': 'Say "Hello, World!"'}

api.add_resource(TodoItem, '/todos/<int:id>')
app.register_blueprint(api_bp)
```
> 注
> 
> 这里不是必须要调用Api.init应用程序()，因为在应用程序中注册该应用程序会负责设置应用程序的路由。

[返回目录](#anchor0)
## <span id = "anchor6.3">参数解析的完整示例</span>
在文档的其他地方，我们已经详细描述了如何使用reqparse示例。在这里，我们将设置一个具有多个输入参数的资源，这些参数可以执行更多的选项。我们将定义一个名为“User”的资源。

```
from flask_restful import fields, marshal_with, reqparse, Resource

def email(email_str):
    """Return email_str if valid, raise an exception in other case."""
    if valid_email(email_str):
        return email_str
    else:
        raise ValueError('{} is not a valid email'.format(email_str))

post_parser = reqparse.RequestParser()
post_parser.add_argument(
    'username', dest='username',
    location='form', required=True,
    help='The user\'s username',
)
post_parser.add_argument(
    'email', dest='email',
    type=email, location='form',
    required=True, help='The user\'s email',
)
post_parser.add_argument(
    'user_priority', dest='user_priority',
    type=int, location='form',
    default=1, choices=range(5), help='The user\'s priority',
)

user_fields = {
    'id': fields.Integer,
    'username': fields.String,
    'email': fields.String,
    'user_priority': fields.Integer,
    'custom_greeting': fields.FormattedString('Hey there {username}!'),
    'date_created': fields.DateTime,
    'date_updated': fields.DateTime,
    'links': fields.Nested({
        'friends': fields.Url('user_friends'),
        'posts': fields.Url('user_posts'),
    }),
}

class User(Resource):

    @marshal_with(user_fields)
    def post(self):
        args = post_parser.parse_args()
        user = create_user(args.username, args.email, args.user_priority)
        return user

    @marshal_with(user_fields)
    def get(self, id):
        args = post_parser.parse_args()
        user = fetch_user(id)
        return user
```
正如您所看到的，我们创建了一个post\_parser，专门用于处理在POST中提供的参数的解析。让我们逐步了解每个参数的定义。
```
post_parser.add_argument(
    'username', dest='username',
    location='form', required=True,
    help='The user\'s username',
)
```
用户名字段是所有这些字段中最正常的。它从POST主体中获取一个字符串，并将其转换为字符串类型。这个参数是必需的(required=True)，这意味着如果没有提供该参数，flask-restful将自动返回400，并返回“the username field is required”的错误消息。

```
post_parser.add_argument(
    'email', dest='email',
    type=email, location='form',
    required=True, help='The user\'s email',
)
```
email字段是一种自定义类型的email。在前面几行中，我们定义了一个email函数，它接受一个字符串,如果类型是有效就返回这个字符串，否则会抛出一个异常，并声明电子邮件类型是无效的。
```
post_parser.add_argument(
    'user_priority', dest='user_priority',
    type=int, location='form',
    default=1, choices=range(5), help='The user\'s priority',
)
```
user_priority 类型使用选择参数，意味着如果提供的user_priority 值没有落在指定的悬着范围内(在这个例子中是 [0, 1, 2, 3, 4]), Flask-RESTful 将自动响应400号错误和错误描述信息。

在这些输入中。我们还在user\_fields字典中定义了一些有趣的字段类型，以展示一些更奇特的类型。
```
user_fields = {
    'id': fields.Integer,
    'username': fields.String,
    'email': fields.String,
    'user_priority': fields.Integer,
    'custom_greeting': fields.FormattedString('Hey there {username}!'),
    'date_created': fields.DateTime,
    'date_updated': fields.DateTime,
    'links': fields.Nested({
        'friends': fields.Url('user_friends', absolute=True),
        'posts': fields.Url('user_friends', absolute=True),
    }),
}
```
首先，这有一个 fields.FormattedString.
```
'custom_greeting': fields.FormattedString('Hey there {username}!'),
```
这个字段的主要作用是向响应中插入值，在这个例子中custom_greeting将一直包含username字段的值。

其次，查看fields.Nested.
```
'links': fields.Nested({
    'friends': fields.Url('user_friends', absolute=True),
    'posts': fields.Url('user_posts', absolute=True),
}),
```
这个字段作用是在响应中创建一个子对象。在这个例子中，我们想创建一个叫links的子对象，包含对象的URLs。
注意，我们向fields.Nested传入另一个用这种方法创建的字典，那它也可以作为marshal()的参数

最后，我们使用fields.Url字段类型。
```
'friends': fields.Url('user_friends', absolute=True),
'posts': fields.Url('user_friends', absolute=True),
```
它使用的第一个参数是端点关联的urls对象的名字，这个对象在links的子对象中。使用absolute=True ensure保证urls包含主机名称。

[返回目录](#anchor0)
## <span id = "anchor6.4">将构造函数作为参数传递给资源</span>
您的资源实现可能需要外部依赖。这些依赖关系最好通过构造函数来进行松散组织在一起。Api.add\_resource()方法有两个关键字参数:resource\_class\_args和resource\_class\_kwargs。它们的值将被转发并传递到您的资源实现的构造函数中。
比如，你可能有一个资源:
```
from flask_restful import Resource

class TodoNext(Resource):
    def __init__(self, **kwargs):
        # smart_engine is a black box dependency
        self.smart_engine = kwargs['smart_engine']

    def get(self):
        return self.smart_engine.next_todo()
```
你可以将必要的外部依赖注入到TodoNext中，就像这样:
```
smart_engine = SmartEngine()

api.add_resource(TodoNext, '/next',
    resource_class_kwargs={ 'smart_engine': smart_engine })
```
同样这种方法也可以应用于转发参数。
[返回目录](#anchor0)