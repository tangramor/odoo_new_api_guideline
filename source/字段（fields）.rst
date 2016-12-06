字段
====

现在字段是类的属性： ::

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

新API的一个新特性就是可以修改字段的一个属性： ::

   name = fields.Char(string='New Value')

字段类型
-------

布尔型（Boolean）
##############

布尔型的字段： ::

    abool = fields.Boolean()

字符型（Char）
###########

存储字符串并包含变量长度： ::

    achar = fields.Char()


特殊选项：

 * size: 数据会根据指定长度尺寸裁剪
 * translate: 字段可以被翻译

文本型（Text）
###########

用于存储长文本： ::

    atext = fields.Text()


特殊选项：

 * translate: 字段可以被翻译

HTML
####

用于存储 HTML 代码，提供一个html小部件： ::

    anhtml = fields.Html()


特殊选项：

 * translate: 字段可以被翻译


整型（Integer）
#############

存储整数值。不支持 NULL 值，如果未设定值则返回 0： ::

    anint = fields.Integer()

浮点型（Float）
############

存储浮点值。不支持 NULL 值，如果未设定值则返回 0.0
如果设定了 digits 选项，那么将使用数值（numeric）类型： ::


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

: ::

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

: ::

    >>> fields.Datetime.context_timestamp(self, timestamp=datetime.datetime.now())
    datetime.datetime(2014, 6, 15, 21, 26, 1, 248354, tzinfo=<DstTzInfo 'Europe/Brussels' CEST+2:00:00 DST>)
    >>> fields.Datetime.now()
    '2014-06-15 19:26:13'
    >>> fields.Datetime.from_string(fields.Datetime.now())
    datetime.datetime(2014, 6, 15, 19, 32, 17)
    >>> fields.Datetime.to_string(datetime.datetime.now())
    '2014-06-15 19:26:13'


Binary
######

Store file encoded in base64 in bytea column: ::

    abin = fields.Binary()

Selection
#########

Store text in database but propose a selection widget.
It induces no selection constraint in database.
Selection must be set as a list of tuples or a callable that returns a list of tuples: ::

    aselection = fields.Selection([('a', 'A')])
    aselection = fields.Selection(selection=[('a', 'A')])
    aselection = fields.Selection(selection='a_function_name')

Specific options:

  * selection: a list of tuple or a callable name that take recordset as input
  * size: the option size=1 is mandatory when using indexes that are integers, not strings

When extending a model, if you want to add possible values to a selection field,
you may use the `selection_add` keyword argument::

   class SomeModel(models.Model):
       _inherits = 'some.model'
       type = fields.Selection(selection_add=[('b', 'B'), ('c', 'C')])

Reference
#########

Store an arbitrary reference to a model and a row: ::

    aref = fields.Reference([('model_name', 'String')])
    aref = fields.Reference(selection=[('model_name', 'String')])
    aref = fields.Reference(selection='a_function_name')

Specific options:

  * selection: a list of tuple or a callable name that take recordset as input


Many2one
########

Store a relation against a co-model: ::

    arel_id = fields.Many2one('res.users')
    arel_id = fields.Many2one(comodel_name='res.users')
    an_other_rel_id = fields.Many2one(comodel_name='res.partner', delegate=True)


Specific options:

  * comodel_name: name of the opposite model
  * delegate: set it to ``True`` to make fields of the target model accessible from the current model (corresponds to ``_inherits``)

One2many
########

Store a relation against many rows of co-model: ::

    arel_ids = fields.One2many('res.users', 'rel_id')
    arel_ids = fields.One2many(comodel_name='res.users', inverse_name='rel_id')

Specific options:

  * comodel_name: name of the opposite model
  * inverse_name: relational column of the opposite model


Many2many
#########

Store a relation against many2many rows of co-model: ::

    arel_ids = fields.Many2many('res.users')
    arel_ids = fields.Many2many(comodel_name='res.users',
                                relation='table_name',
                                column1='col_name',
                                column2='other_col_name')


Specific options:

  * comodel_name: name of the opposite model
  * relation: relational table name
  * columns1: relational table left column name
  * columns2: relational table right column name


Name Conflicts
--------------

.. note::
   fields and method name can conflict.

When you call a record as a dict it will force to look on the columns.


Fields Defaults
---------------

Default is now a keyword of a field:

You can attribute it a value or a function

::

   name = fields.Char(default='A name')
   # or
   name = fields.Char(default=a_fun)

   #...
   def a_fun(self):
      return self.do_something()

Using a fun will force you to define function before fields definition.




Computed Fields
---------------
There is no more direct creation of fields.function.

Instead you add a ``compute`` kwarg. The value is the name of the function as a string or a function.
This allows to have fields definition atop of class: ::

    class AModel(models.Model):
        _name = 'a_name'

        computed_total = fields.Float(compute='compute_total')

        def compute_total(self):
            ...
            self.computed_total = x


The function can be void.
It should modify record property in order to be written to the cache: ::

  self.name = new_value

Be aware that this assignation will trigger a write into the database.
If you need to do bulk change or must be careful about performance,
you should do classic call to write

To provide a search function on a non stored computed field
you have to add a ``search`` kwarg on the field. The value is the name of the function
as a string or a reference to a previously defined method. The function takes the second
and third member of a domain tuple and returns a domain itself ::

        def search_total(self, operator, operand):
	    ...
            return domain  # e.g. [('id', 'in', ids)] 

Inverse
-------

The inverse key allows to trigger call of the decorated function
when the field is written/"created"


Multi Fields
------------
To have one function that compute multiple values: ::

    @api.multi
    @api.depends('field.relation', 'an_otherfield.relation')
    def _amount(self):
        for x in self:
            x.total = an_algo
            x.untaxed = an_algo


Related Field
-------------

There is not anymore ``fields.related`` fields.

Instead you just set the name argument related to your model: ::

  participant_nick = fields.Char(string='Nick name',
                                 related='partner_id.name')

The ``type`` kwarg is not needed anymore.

Setting the ``store`` kwarg will automatically store the value in database.
With new API the value of the related field will be automatically
updated, sweet. ::

  participant_nick = fields.Char(string='Nick name',
                                 store=True,
                                 related='partner_id.name')

.. note::
   When updating any related field not all
   translations of related field are translated if field
   is stored!!

Chained related fields modification will trigger invalidation of the cache
for all elements of the chain.


Property Field
--------------

There is some use cases where value of the field must change depending of
the current company.

To activate such behavior you can now use the `company_dependent` option.

A notable evolution in new API is that "property fields" are now searchable.

WIP copyable option
-------------------

There is a dev running that will prevent to redefine copy by simply
setting a copy option on fields: ::

  copy=False  # !! WIP to prevent redefine copy
