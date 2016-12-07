方法与修饰符
============

新的修饰符只用于新API。
因为网页客户端（webclient）和HTTP控制器与新API不兼容，所以对新API修饰符是强制使用的。

``api`` 命名空间下的修饰符会检查方法的参数名称以确定是否符合旧参数。

已认定的参数名称有：

``cr, cursor, uid, user, user_id, id, ids, context``


@api.returns
------------

这个修饰符保证返回值的一致性。This decorator guaranties unity of returned value.
它基于原始返回值返回指定模型的一个记录集：::

    @api.returns('res.partner')
    def afun(self):
        ...
        return x  # 一个记录集

如果一个旧API方法调用新API方法，它会自动转换为一个id列表。

所有修饰符都继承自这个修饰符，以升级或降级返回值。

@api.one
--------

这个修饰符自动遍历记录集的记录，self被重新定义为当前记录：::

  @api.one
  def afun(self):
      self.name = 'toto'


.. note::
   注意：返回值被放进一个列表里，这种做法并不是总被网页客户端支持，例如在按钮动作方法里。在那种情况下，你应该用 ``@api.multi`` 来修饰你的方法，并且可能需要在方法定义里调用 `self.ensure_one()` 。


@api.multi
----------

self为当前记录集，无迭代。
以下是缺省做法：::

   @api.multi
   def afun(self):
       len(self)

@api.model
----------

这个修饰符会把旧API对它修饰的方法的调用转换到新API参数。
它让我们可以优雅的迁移旧代码：::

    @api.model
    def afun(self):
        pass

@api.constrains
---------------

这个修饰符确保当进行创建、写入、删除等操作时，被修饰的方法会被调用。
如果一个约束条件被满足，这个方法应该抛出 `openerp.exceptions.Warning` 并给出相应警告消息。

@api.depends
------------

当这个修饰符指定的任何字段被ORM或表单修改，它所修饰的方法会被触发调用：::

    @api.depends('name', 'an_other_field')
    def afun(self):
        pass


.. note::
   当你重新定义依赖时，你必须重新定义所有的 @api.depends，否则它会丢失监视的字段。

视图管理
#######
新API的一个重大提升是依赖会通过一种简单的方式自动插入表单，你不用再担心修改视图的事情。



.. _@api.onchange:

@api.onchange
--------------
当这个修饰符指定的字段在表单里被修改时，它所修饰的方法会被触发调用：::

  @api.onchange('fieldx')
  def do_stuff(self):
     if self.fieldx == x:
        self.fieldy = 'toto'

例子里面的 `self` 对应的记录现在被用户在表单里修改了。
在 on_change 上下文里所有的工作都是在缓存里完成。
所以你可以在你的方法里修改记录集而不用担心修改了数据库记录。
这是跟 ``@api.depends`` 的主要区别。

在方法返回时，缓存和记录集之间的差异会被返回给表单（At function return, differences between the cache and the RecordSet will be returned
to the form.）。

视图管理
###############
新API的一个重大提升是 变动（onchange）会通过一种简单的方式自动插入表单，你不用再担心修改视图的事情。

警告和域（Warning and Domain）
###########################
要改变域或发送一个警告，返回正常的字典（dictionary）即可。
要小心这种情况下不要使用 ``@api.one`` ，因为它会破坏字典（把它放入一个列表，这个不被网页客户端支持）。


@api.noguess
------------

这个修饰符阻止新API修饰符去改变一个方法的输出。
