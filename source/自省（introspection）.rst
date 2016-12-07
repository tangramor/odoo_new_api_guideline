自省（Introspection）
===================
在OpenERP一个常见模式是使用 ``_columns`` 对模型的字段自省。
从8.0版本开始 ``_columns`` 已被弃用，而使用 ``_fields`` 替代，它包含了使用新、旧API初始化的整理过的字段列表。
