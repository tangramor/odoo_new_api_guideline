字段
====

现在字段是类的属性：::

    from openerp import models, fields

    class AModel(models.Model):

        _name = 'a_name'

        name = fields.Char(
            string="Name",                   # 字段的可选标签
            compute="_compute_name_custom",  # 在计算字段里转换的字段
            store=True,                      # 如果计算，就保存结果
            select=True,                     # 强制对字段索引
            readonly=True,                   # 在视图中字段将是只读的
            inverse="_write_name"            # On update trigger
            required=True,                   # 必需的字段
            translate=True,                  # 启用翻译
            help='blabla',                   # 工具提示帮助文本
            company_dependent=True,          # 把列转换到 ir.property
            search='_search_function'        # 自定义搜索功能，主要和计算一起使用
        )

       # string 关键字不是必需的
       # 缺省情况下使用大写属性名称

       name = fields.Char()  # 合法定义


.. _fields_inherit:

字段继承
-------

新API的一个新特性就是可以修改字段的一个属性：::

   name = fields.Char(string='New Value')

字段类型
-------

布尔型（Boolean）
##############

布尔型的字段：::

    abool = fields.Boolean()

字符型（Char）
###########

存储字符串并包含变量长度：::

    achar = fields.Char()


特殊选项：

 * size: 数据会根据指定长度尺寸裁剪
 * translate: 字段可以被翻译

文本型（Text）
###########

用于存储长文本：::

    atext = fields.Text()


特殊选项：

 * translate: 字段可以被翻译

HTML
####

用于存储 HTML 代码，提供一个html小部件：::

    anhtml = fields.Html()


特殊选项：

 * translate: 字段可以被翻译


整型（Integer）
#############

存储整数值。不支持 NULL 值，如果未设定值则返回 0：::

    anint = fields.Integer()

浮点型（Float）
############

存储浮点值。不支持 NULL 值，如果未设定值则返回 0.0
如果设定了 digits 选项，那么将使用数值（numeric）类型：::


    afloat = fields.Float()
    afloat = fields.Float(digits=(32, 32))
    afloat = fields.Float(digits=lambda cr: (32, 32))

特殊选项：

  * digits: 强制使用数据库的数值（numeric）类型。参数可以是一个元组（tuple） (int len, float len) 或者一个使用记录指针（cursor）作为参数的、返回值为一个元组（tuple）的可调用方法

日期型（Date）
###########

存储日期。
这种字段提供一些辅助方法：

  * ``context_today`` 返回基于时区（tz）的当日日期字符串
  * ``today`` 返回当前系统日期字符串
  * ``from_string`` 返回从字符串转换来的 datetime.date() 值
  * ``to_string`` 返回从datetime.date来的日期字符串

::

    >>> from openerp import fields

    >>> adate = fields.Date()
    >>> fields.Date.today()
    '2014-06-15'
    >>> fields.Date.context_today(self)
    '2014-06-15'
    >>> fields.Date.context_today(self, timestamp=datetime.datetime.now())
    '2014-06-15'
    >>> fields.Date.from_string(fields.Date.today())
    datetime.datetime(2014, 6, 15, 19, 32, 17)
    >>> fields.Date.to_string(datetime.datetime.today())
    '2014-06-15'

日期和时间型（DateTime）
####################

存储日期和时间。
这种字段提供一些辅助方法：

  * ``context_timestamp`` 返回基于时区（tz）的当日日期时间戳字符串
  * ``now`` 返回当前系统日期和时间字符串
  * ``from_string`` 返回从字符串转换来的 datetime.datetime() 值
  * ``to_string`` 返回从datetime.date来的日期和时间字符串

::

    >>> fields.Datetime.context_timestamp(self, timestamp=datetime.datetime.now())
    datetime.datetime(2014, 6, 15, 21, 26, 1, 248354, tzinfo=<DstTzInfo 'Europe/Brussels' CEST+2:00:00 DST>)
    >>> fields.Datetime.now()
    '2014-06-15 19:26:13'
    >>> fields.Datetime.from_string(fields.Datetime.now())
    datetime.datetime(2014, 6, 15, 19, 32, 17)
    >>> fields.Datetime.to_string(datetime.datetime.now())
    '2014-06-15 19:26:13'


二进制型（Binary）
###############

存储以base64编码的文件到字节列：::

    abin = fields.Binary()

可选型（Selection）
################

存储文本，但给出一个可选项小部件。
它向数据库引入了非可选约束。（It induces no selection constraint in database.）
可选型必须设置为元组（tuples）列表或者一个返回元组（tuples）列表的可调用方法：::

    aselection = fields.Selection([('a', 'A')])
    aselection = fields.Selection(selection=[('a', 'A')])
    aselection = fields.Selection(selection='a_function_name')

特殊选项：

  * selection: 元组（tuples）列表或者一个使用记录集为输入参数的返回元组（tuples）列表的可调用方法名称
  * size: 当使用的索引是整型而非字符串时，必须设定本选项且须设置size=1

在扩展一个模型时，如果你希望向可选型字段添加可能的值，你应该用到 `selection_add` 关键字参数：::

   class SomeModel(models.Model):
       _inherits = 'some.model'
       type = fields.Selection(selection_add=[('b', 'B'), ('c', 'C')])


引用型（Reference）
################

存储到一个模型和一行记录的任意引用：::

    aref = fields.Reference([('model_name', 'String')])
    aref = fields.Reference(selection=[('model_name', 'String')])
    aref = fields.Reference(selection='a_function_name')

特殊选项：

  * selection: 元组（tuples）列表或者一个使用记录集为输入参数的返回元组（tuples）列表的可调用方法名称


多对一型（Many2one）
#################

存储模型之间的多对一关联：::

    arel_id = fields.Many2one('res.users')
    arel_id = fields.Many2one(comodel_name='res.users')
    an_other_rel_id = fields.Many2one(comodel_name='res.partner', delegate=True)


特殊选项：

  * comodel_name: 对应的模型名称
  * delegate: 为了从当前模型访问目标模型的字段，需要将此选项设为 ``True`` （对应于``_inherits``）


一对多型（One2many）
#################

存储到对应模型的多个记录的关联：::

    arel_ids = fields.One2many('res.users', 'rel_id')
    arel_ids = fields.One2many(comodel_name='res.users', inverse_name='rel_id')

特殊选项：

  * comodel_name: 对应的模型名称
  * inverse_name: 对应的模型的关联字段名称


多对多型（Many2many）
##################

存储到对应模型的多对多个记录的关联：::

    arel_ids = fields.Many2many('res.users')
    arel_ids = fields.Many2many(comodel_name='res.users',
                                relation='table_name',
                                column1='col_name',
                                column2='other_col_name')


特殊选项：

  * comodel_name: 对应的模型名称
  * relation: 关联的表名称
  * columns1: 关联表左字段名称
  * columns2: 关联表右字段名称


名称冲突
-------

.. note::
   字段和方法名称有可能出现冲突。

当你使用字典（dict）类型来调用一个记录时，将强制查询这个名字的字段。


字段缺省值
--------

`default`现在是字段的一个关键字，你可以使用值或者方法来为该属性赋值：::

   name = fields.Char(default='A name')
   # 或者
   name = fields.Char(default=a_fun)

   #...
   def a_fun(self):
      return self.do_something()

当使用方法时，你必须在字段定义前定义该方法。




计算字段（Computed Fields）
------------------------

不再有直接的 `fields.function` 创建方式。

作为替代，你可以添加一个``compute``关键字。该关键字属性的值就是一个方法名字符串或者一个返回方法名称的方法。
它允许你在类一开始的部分就定义字段：::

    class AModel(models.Model):
        _name = 'a_name'

        computed_total = fields.Float(compute='compute_total')

        def compute_total(self):
            ...
            self.computed_total = x


这个方法可以是空的。
它应该修改记录属性以便写入到缓存里：::

  self.name = new_value

要注意这个赋值会触发数据库的写操作。
如果你需要对大量数据进行修改或者必须考虑性能，你应该使用经典方式来写数据库。

为了提供搜索功能到未持久化的计算字段，你必须为该字段添加 ``search`` 关键字。该关键字属性的值就是一个方法名字符串或者或者之前定义的一个返回方法名称的方法，这个方法的第2和第3个参数均为域元组（domain tuple），返回一个域（domain）本身（The function takes the second and third member of a domain tuple and returns a domain itself）：::

        def search_total(self, operator, operand):
	    ...
            return domain  # e.g. [('id', 'in', ids)] 

翻转（Inverse）
-------

翻转``inverse``关键字允许在字段写入或“创建”时触发修饰方法。


多字段（Multi Fields）
-------------------

一个方法计算多个值：::

    @api.multi
    @api.depends('field.relation', 'an_otherfield.relation')
    def _amount(self):
        for x in self:
            x.total = an_algo
            x.untaxed = an_algo


关联字段（Related Field）
----------------------

这不再是``fields.related``字段。

作为替代你仅仅需要设定``related``关键字属性值为关联字段名：::

  participant_nick = fields.Char(string='Nick name',
                                 related='partner_id.name')

``type``关键字不再需要。

设定``store``关键字属性会自动存储值到数据库。新API会自动更新关联字段的值，贴心：::

  participant_nick = fields.Char(string='Nick name',
                                 store=True,
                                 related='partner_id.name')

.. note::
   当更新任何关联字段时，如果有字段已经保存，不是所有关联字段的翻译都会更新！！

关联字段的链条上的改动会触发对链条上所有元素缓存的失效。


属性字段（Property Field）
-----------------------

在有一些用例里，字段值必须修改到当前公司的依赖。

要启用这种动作，你现在可以使用`company_dependent`选项。

一个值得注意的进展是，在新API里属性字段（property fields）现在是可搜索的。


半成品可拷贝选项（WIP copyable option）
----------------------------------

在字段上简单的设定``copy``选项，可以防止重新定义拷贝（There is a dev running that will prevent to redefine copy by simply
setting a copy option on fields）：::

  copy=False  # !! WIP to prevent redefine copy
