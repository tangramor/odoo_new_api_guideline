单元测试
========
在common.TransactionCase内或其它类里的单元测试中访问新API： ::

    class test_partner_firstname(common.TransactionCase):

        def setUp(self):
            super(test_partner_firstname, self).setUp()
            self.user_model = self.env["res.users"]
            self.partner_model = self.env["res.partner"]

YAML
====
在Python YAML标签里访问新API： ::

    !python {model: account.invoice, id: account_invoice_customer0}: |
        self  # 现在是新API记录
        assert (self.move_id), "Move falsely created at pro-forma"
