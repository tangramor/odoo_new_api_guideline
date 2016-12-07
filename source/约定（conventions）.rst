约定
========

下划线（Snake_casing）还是 驼峰（CamelCasing）
---------------------------------------------------
目前没有定论。
但是现在看起来 OpenERP SA 在9.0版开始使用驼峰命名方式。

导入（Imports）
-------------------
通过于 Raphaël Collet 的讨论，在8.0 RC1之后将会使用下面的约定

模型（Model）
####################

::

  from openerp import models

字段（Fields）
########################

::

  from openerp import fields

翻译（Translation）
##########################

::

  from openerp import _

开发接口（API）
#####################

::

  from openerp import api

例外（Exceptions）
#########################

::

  from openerp import exceptions

一个标准的模块导入： ::
  
  from openerp import models, fields, api, _


类（Classes）
------------------
类应该以如下方式初始化： ::

    class Toto(models.Model):
       pass
    
    
    class Titi(models.TransientModel):
       pass


新的例外类
--------------

``except_orm`` 例外已经弃用。
我们应该使用 ``openerp.exceptions.Warning`` 及其子类实例

.. note::
  不要与Python内建的警告混合。


重定向警告（RedirectWarning）
##################################

警告用户有可能被重定向而不是简单的显示一个警示消息

应该作为参数接收：

* :param int action_id: 执行重定向的动作的id
* :param string button_text: 触发重定向的按钮上的文字

禁止访问（AccessDenied）
##############################

登录、密码错误。无消息，无追溯。

访问错误（AccessError）
#############################

访问权限错误。

缺失错误（MissingError）
##############################

记录缺失。

推迟例外（DeferredException）
###################################

持有对异步报告的追溯的例外对象。

有一些远程调用（RPC）（创建数据库、生成报告）发生时的初始请求伴随着多个或轮询请求。这个类用来存储在该线程处理初始请求时并然后要发送给一个轮询请求时，发生的可能例外。

.. note::
   追溯让人迷惑，这真的是一个 ``sys.exc_info()`` triplet.


兼容性（Compatibility）
#############################

当捕捉ORM例外时，我们应该同时捕捉这两种例外： ::

    try:
        pass
    except (Warning, except_orm) as exc:
        pass


字段（Fields）
------------------

字段应该使用新API的字段声明方式。
使用string关键字来作说明比使用一个长长的属性名称要更好： ::

    class AClass(models.Model):

        name = fields.Char(string="This is a really long long name")  # ok
        really_long_long_long_name = fields.Char()

属性名称应该有意义，避免使用类似“nb”这样的名字。


缺省 或 计算
----------------

``compute`` 选项不应该用于设置缺省值。
缺省值应该只用于属性初始化。

上面的意思是他们可能共用一个方法。

在方法内修改自身
-------------------

我们永远不要在一个模型方法内修改自身。
这种做法会破坏与当前环境缓存的关联性。


在演习（dry run）中执行
----------------------------

如果你使用环境上下文管理器的 ``do_in_draft``，它将只在缓存中执行而不会提交到数据库。


使用记录指针（Cursor）
--------------------------

使用记录指针时你应该使用当前环境的记录指针： ::

      self.env.cr

except if you need to use threads: ::

    with Environment.manage():  # class function
        env = Environment(cr, uid, context)

显示名称
-----------

`_name_get` 已弃用。

你应该定义 display_name 字段，可包含以下选项：

 * ``compute``
 * ``inverse``


约束
--------

在性能允许的情况下，应该使用 ``@api.constrains`` 装饰器与 ``@api.one`` 装饰器。


Qweb视图 或 非Qweb视图
------------------------------

如果在模型视图里不需要高级的行为，应首先选择标准视图（非Qweb）。


Javascript 和 网站（Website）相关代码
----------------------------------------------

可以在下面链接找到相关指南：

 * https://doc.openerp.com/trunk/web/guidelines/
 * https://doc.openerp.com/trunk/server/howto/howto_website/
