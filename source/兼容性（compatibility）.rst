兼容性
=============
在迁移期间，有一些保持基础代码在新旧API上兼容性的模式。

访问旧 API
--------------

一般的，用新API你会使用 ``self``，这是一个新的记录集（RecordSet）类的实例。
但是旧API上下文和模型依然可以用：\n::

    self.pool
    self._model


如何优雅的对待旧的基础代码
-----------------------------------
如果旧的API基础代码必须使用到你的代码，你应该使用以下装饰器：

 * ``@api.returns`` 以确保合适的返回值
 * ``@api.model`` 以确保新的签名支持旧API调用
