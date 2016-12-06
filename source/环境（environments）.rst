记录/记录集（Record/Recordset） 与 模型（Model）
===========================================

OpenERP/Odoo 8.0版引入了一个新的ORM API。

它的目标是提供更连贯和简洁的语法形式，并保持前后的兼容性。

新的API保留了之前的基础设计，例如 模型（Model）和 记录（Record），但是增加了新的概念，例如 环境（Environment）和 记录集（Recordset）。

之前API的一些内容在这个版本里也没有变化，例如 域（domain）的语法。


模型（Model）
-----------

一个模型代表了一个商业对象。

它本质上是一个类，包含各种类有关的功能定义和存储在数据库中的字段。
所有定义在模型中的方法都可以被模型本身直接调用。

现在编程范式有所改变，你不应该直接访问模型，而是应该使用 记录集（RecordSet） 参见 :ref:`recordset`

要实例化一个模型，你必须继承 openerp.model.Model: ::

    from openerp import models, fields, api, _


    class MyModel(models.Model):

        _name = 'a.model'  # 模型名称会被用作数据库表名

        firstname = fields.Char(string="Firstname")


继承（Inheritance）
#################

继承机制没有任何变化。你可以使用： ::

    class MyModelExtended(Model):
         _inherit = 'a.model'                       # 直接继承
         _inherit = ['a.model', 'a.other.model']     # 直接继承
         _inherits = {'a.model': 'field_name'}      # 多重继承

关于继承更多的细节内容，请参考

  `Inherit <https://www.odoo.com/forum/Help-1/question/The-different-openerp-model-inheritance-mechanisms-whats-the-difference-between-them-and-when-should-they-be-used--46#answer-190>`_

字段继承请参考 :ref:`fields_inherit`

.. _recordset:

记录集（Recordset）
-----------------

所有模型的实例同时也是对应记录集的实例。
一个记录集表达了对应模型的经过排序的记录的集合。

你可以在记录集里调用方法： ::

    class AModel(Model):
    # ...
        def a_fun(self):
            self.do_something() # 这里self是一个记录集，介于 类（class）与 集合（set）之间的混合体
            record_set = self
            record_set.do_something()

        def do_something(self):
            for record in self:
               print record

在这个例子里，方法定义在模型一级，但是当运行这段代码时， ``self`` 变量实际上是包含很多记录的一个记录集的实例。

所以传入 ``do_something`` 的 self 是一个含义一系列记录的记录集。

如果你用 ``@api.one`` 修饰一个方法的话，它会自动遍历当前记录集的记录，然后这时的 self 就是当前这条记录。

如我们在 :ref:`records` 中所述，你现在访问的就是一个伪 Active-Record模式。

.. note::
   如果记录集只含有一条记录，你把这个修饰符用在该记录集上会导致中断。
   If you use it on a RecordSet it will break if recordset does not contains only one item.!!


支持的操作
--------

记录集也支持集合操作，你可以使用 联合、交、补 等运算： ::

    record in recset1       # 属于
    record not in recset1   # 不属于
    recset1 + recset2       # 扩展（extend）
    recset1 | recset2       # 联合（union）
    recset1 & recset2       # 交集（intersect）
    recset1 - recset2       # 差补/相对补集（difference）
    recset.copy()           # 记录集的浅复制（被复制对象的所有变量都含有与原来的对象相同的值，而其所有的对其他对象的引用都仍然指向原来的对象）

只有 ``+`` 操作符保留集合元素的顺序

记录集也可以被排序： ::

  sorted(recordset, key=lambda x: x.column)

有用的辅助方法
-----------

新的API为记录集提供了很多有用的辅助方法。

你可以轻易的过滤已有的记录集： ::

    recset.filtered(lambda record: record.company_id == user.company_id)
    # 或使用字符串
    recset.filtered("product_id.can_be_sold")

你可以对一个记录集排序： ::
    
    # 按名称对记录排序
    recset.sorted(key=lambda r: r.name)

你也可以使用 operator 模块 ::
    
    from operator import attrgetter
    recset.sorted(key=attrgetter('partner_id', 'name'))
    
有一个映射记录集的辅助方法： ::

    recset.mapped(lambda record: record.price_unit - record.cost_price)
    
    # 返回名称列表
    recset.mapped('name')

    # 返回合作者记录集
    recset.mapped('invoice_id.partner_id')

ids 属性
--------

ids 属性是记录集的一个特殊属性，当记录集包含一个或更多记录时它会返回对应ids。

.. _records:

记录（Record）
------------

一个记录反映了从数据库中取得的“模型记录实例”。它使用缓存和查询生成了数据库记录条目的抽象；:

  >>> record = self
  >>> record.name
  toto
  >>> record.partner_id.name
  partner name


记录的显示名称（display name）
##########################

在新API里一个叫显示名称的概念被引入。它使用 ``name_get`` 底层方法。


所以如果你希望覆盖显示名称，你需要覆盖 ``display_name`` 字段
`Example <https://github.com/odoo/odoo/blob/8.0/openerp/addons/base/res/res_partner.py#L232>`_


如果你希望覆盖显示名称和计算出的相关名称，你需要覆盖 ``name_get``。
`Example <https://github.com/odoo/odoo/blob/8.0/addons/event/event.py#L194>`_


.. _ac_pattern:

Active Record 模式
##################

在新API引入的新特性之一是对active record模式的基础支撑。你现在可以使用设置属性（setting properties）来写入数据库： ::

  record = self
  record.name = 'new name'

上面的例子会更新缓存中的值并调用写方法来触发向数据库的写入动作。


Active Record模式 注意事项
########################

使用Active Record模式写值必须要小心，因为每一个值的指定都会触发数据库的写操作： ::


    @api.one
    def dangerous_write(self):
      self.x = 1
      self.y = 2
      self.z = 4

在这个例子里每一个赋值都会触发写操作。
因为这个方法使用了 ``@api.one`` 修饰，对记录集里的每个记录的写操作都会被调用3次，那么如果你的记录集有10条记录，一共会有 10*3 = 30 次写操作。

这在高负载任务里会导致性能问题。你应该这样写： ::

    def better_write(self):
       for rec in self:
          rec.write({'x': 1, 'y': 2, 'z': 4})

    # 或者

    def better_write2(self):
       # 给所有记录赋相同值
       self.write({'x': 1, 'y': 2, 'z': 4})


空查询（Browse_null）链
#####################


空关系现在返回一个空的记录集。

在新API里，如果你关联到一个拥有很多空关系的关系（relation），每个关系都会被关联，最后会返回一个空的记录集。


环境（Environment）
=================

新API引入了环境的定义。它的主要目的是提供对于 记录指针（cursor）、用户id（user_id）、模型（model）、上下文（context）、记录集（Recordset）和缓存（cache）的封装。

.. image:: Diagram1.png


有了这个附加功能，你就不用传递那么多的方法参数了： ::


    # 以前
    def afun(self, cr, uid, ids, context=None):
        pass

    # 现在
    def afun(self):
        pass


为了访问到环境，你需要： ::

    def afun(self):
         self.env
         # 或者
         model.env

环境应该是不可变的，不能在方法里进行修改，因为它还存储着记录集的缓存等等信息。


修改环境
-------

如果你需要修改当前的上下文，你需要使用 with_context() 方法： ::

  self.env['res.partner'].with_context(tz=x).create(vals)

要小心不要使用如下方法修改当前记录集： ::

   self = self.env['res.partner'].with_context(tz=x).browse(self.ids)


它会在重新查询后修改当前记录集里的记录，从而导致缓存和记录集之间的不连贯。


改变用户
#######

环境提供了一个切换用户的辅助方法： ::

    self.sudo(user.id)
    self.sudo()   # 缺省会使用 SUPERUSER_ID
    # 或者
    self.env['res.partner'].sudo().create(vals)

访问当前用户
##########

::

    self.env.user
    

使用XML id 获取记录
#################

::

    self.env.ref('base.main_company')


清理环境缓存
----------

在前面我们介绍了环境维护着多种缓存，这些缓存用于模型、字段等类。

有时候你可能必须要使用记录指针（cursor）来直接插入/写数据，这种情况下你需要使这些缓存无效： ::

  self.env.invalidate_all()


一般动作（Common Actions）
=======================

搜索
----
搜索并没有太大变化。可惜的是宣称的域（domain）的变动没有在8.0版本里实现。

下面是一些主要的变化。


search
######

现在 ``seach`` 方法直接返回一个记录集： ::

    >>> self.search([('is_company', '=', True)])
    res.partner(7, 6, 18, 12, 14, 17, 19, 8,...)
    >>> self.search([('is_company', '=', True)])[0].name
    'Camptocamp'

你可以使用env来调用 search ： ::

    >>> self.env['res.users'].search([('login', '=', 'admin')])
    res.users(1,)


search_read
###########

``search_read`` 方法加入进来了。它会执行一个 search 并返回一个字典（dict）列表（list）。

这里我们获取所有合作伙伴名称： ::

    >>> self.search_read([], ['name'])
    [{'id': 3, 'name': u'Administrator'},
     {'id': 7, 'name': u'Agrolait'},
     {'id': 43, 'name': u'Michel Fletcher'},
     ...]

search_count
############
``search_count`` 方法返回符合搜索域（domain）定义的记录数量： ::

    >>> self.search_count([('is_company', '=', True)])
    26L


检索
----
检索是从数据获取记录的标准方法。现在检索会返回一个记录集： ::

    >>> self.browse([1, 2, 3])
    res.partner(1, 2, 3)

更多关于记录的信息请参考 :ref:`records`


写入
----

使用 Active Record 模式
######################

现在可以用 Active Record 模式来写入： ::

    @api.one
    def any_write(self):
      self.x = 1
      self.name = 'a'

更多关于Active Record 模式来写入的小窍门，请参考 :ref:`records`

传统的写入方式仍然可用。

从记录写入
########

从记录写入：  ::

    @api.one
    ...
    self.write({'key': value })
    # 或者
    record.write({'key': value})


从记录集写入
##########

从记录集写入： ::

    @api.multi
    ...
    self.write({'key': value })
    # 它将写入到所有记录里
    self.line_ids.write({'key': value })

它将写入到所有关联的线索（line）的记录里。

多对多（Many2many） 一对多（One2many） 写入行为
#########################################

一对多（One2many） 和 多对多（Many2many）字段有一些特殊行为需要考虑到。
At that time (this may change at release) using create on a multiple relation fields
will not introspect to look for the relation. ::

  self.line_ids.create({'name': 'Tho'})   # 这个调用将会失败，因为没有指定订单（order）
  self.line_ids.create({'name': 'Tho', 'order_id': self.id})  # 这个调用将会正常执行
  self.line_ids.write({'name': 'Tho'})    # 这个调用将会写到所有相关的线索（line）记录里

当在一个修饰了 :ref:`@api.onchange` 的方法里添加新的关联记录时，你可以使用 :py:meth:`openerp.models.BaseModel.new` 构造方法。这个方法会创建一个未提交至数据库的记录，包含一个 :py:class:`openerp.models.NewId` 类型的id。 ::

    self.child_ids += self.new({'key': value})

这种记录在表单保存时会写入数据库。


拷贝
----

.. note::
   标题得改，目前还是有很多问题！！！

从记录拷贝
########

从记录拷贝： ::

    >>> @api.one
    >>> ...
    >>>     self.copy()
    broken


从记录集拷贝
##########

从记录集拷贝： ::

    >>> @api.multi
    >>> ...
    >>>     self.copy()
    broken


创建
----

创建方法没有变化，除了它现在也是返回一个记录集： ::

  self.create({'name': 'New name'})


演习（Dry run）
-------------

你可以通过 ``do_in_draft`` 这个环境上下文管理器的辅助方法来只在缓存中执行动作。


使用记录指针
==========

记录、记录集和环境共用同一个记录指针。

所以你可以用如下方法来访问记录指针： ::

  def my_fun(self):
      cursor = self._cr
      # 或者
      self.env.cr

然后你就可以像以前的API里一样使用记录指针了。


使用线程
=======
使用线程时你必须创建自己的记录指针，并且在每个线程里初始化一个新的环境。
数据库操作提交在提交记录指针时完成： ::

   with Environment.manage():  # 类方法
       env = Environment(cr, uid, context)

新 ids
======

当创建一个包含计算字段的记录或模型时，记录集的记录只在内存里。此时记录的 `id` 将是一个 :py:class:`openerp.models.NewId` 类型的虚拟id。

所以如果你在你的代码里（例如一段SQL查询）用到了记录 `id` 的话，你应该先检查它是否存在：::

   if isinstance(current_record.id, models.NewId):
       # 你的代码
