# Odoo 18.0 → 19.0 Migration Rules

**Reference:** See `migration-18-to-19-report.md` for full analysis.  
**Format:** Each rule has CATEGORY, TRIGGER, ACTION, EXAMPLE, NOTES.

---

## Categories

- `IMPORT` — Python import changes
- `MODEL_RENAME` — Model/class renames
- `FIELD_RENAME` — Field renames or type changes
- `FIELD_REMOVE` — Removed fields
- `METHOD_RENAME` — Method renames
- `METHOD_SIG` — Method signature changes
- `MANIFEST` — `__manifest__.py` changes
- `SECURITY` — `ir.model.access.csv` / security XML changes
- `XML_ID` — XML view ID / action changes
- `MODULE_REMOVE` — Entire module removed or merged

---

## RULE_001

**CATEGORY:** `IMPORT`  
**TRIGGER:** `from odoo.osv import expression`  
**ACTION:** Replace with `from odoo.fields import Domain` and update all `expression.*` calls.

```python
# BEFORE (18.0)
from odoo.osv import expression
domain = expression.AND([domain1, domain2])
domain = expression.OR([domain1, domain2])
domain = expression.normalize_domain(domain)

# AFTER (19.0)
from odoo.fields import Domain
domain = Domain.AND([domain1, domain2])
domain = Domain.OR([domain1, domain2])
domain = Domain(domain).to_list()  # or Domain(domain)
```

**NOTES:** The `expression` module still exists for backward compat but is deprecated. `Domain` is the preferred API in 19.0. All modules using `from odoo.osv import expression` should be updated.

---

## RULE_002

**CATEGORY:** `IMPORT`  
**TRIGGER:** `from odoo.tools import OrderedSet`  
**ACTION:** Replace with `from odoo.tools.misc import OrderedSet`

```python
# BEFORE (18.0)
from odoo.tools import OrderedSet

# AFTER (19.0)
from odoo.tools.misc import OrderedSet
```

**NOTES:** `OrderedSet` was moved from `odoo.tools` to `odoo.tools.misc`. The re-export from `odoo.tools` was removed.

---

## RULE_003

**CATEGORY:** `IMPORT`  
**TRIGGER:** `from odoo import Command`  
**ACTION:** Can now also be imported from `from odoo.fields import Command` (both work)

```python
# BEFORE / AFTER (both valid in 19.0)
from odoo import Command          # still works
from odoo.fields import Command   # preferred in 19.0
```

**NOTES:** Non-breaking. Update for consistency when touching files.

---

## RULE_004

**CATEGORY:** `MODULE_REMOVE`  
**TRIGGER:** `'depends': [..., 'hr_contract', ...]` in `__manifest__.py`  
**ACTION:** Replace `hr_contract` dependency with `hr`

```python
# BEFORE (18.0)
'depends': ['hr', 'hr_contract', 'mail'],

# AFTER (19.0)
'depends': ['hr', 'mail'],
```

**NOTES:** `hr_contract` was merged into `hr`. All contract models (`hr.contract`, `hr.payroll.structure.type`, etc.) are now in `hr`. Any custom module depending on `hr_contract` must update its manifest and check field references.

---

## RULE_005

**CATEGORY:** `MODULE_REMOVE`  
**TRIGGER:** `'depends': [..., 'web_editor', ...]` in `__manifest__.py`  
**ACTION:** Replace `web_editor` with `html_editor` (core editing) or `html_builder` (website builder)

```python
# BEFORE (18.0)
'depends': ['web', 'web_editor', 'mail'],

# AFTER (19.0) — for backend / portal modules
'depends': ['web', 'html_editor', 'mail'],

# AFTER (19.0) — for website builder modules
'depends': ['web', 'html_builder', 'mail'],
```

**NOTES:** `web_editor` was split. Backend/portal rich-text → `html_editor`. Website visual editor → `html_builder`.

---

## RULE_006

**CATEGORY:** `MODULE_REMOVE`  
**TRIGGER:** `'depends': [..., 'account_edi_ubl_cii_tax_extension', ...]`  
**ACTION:** Remove the dependency; functionality merged into `account_edi_ubl_cii`

```python
# BEFORE (18.0)
'depends': ['account_edi_ubl_cii', 'account_edi_ubl_cii_tax_extension'],

# AFTER (19.0)
'depends': ['account_edi_ubl_cii'],
```

---

## RULE_007

**CATEGORY:** `MODULE_REMOVE`  
**TRIGGER:** `'depends': [..., 'auth_totp_mail_enforce', ...]`  
**ACTION:** Replace with `auth_totp_mail`

```python
# BEFORE (18.0)
'depends': [..., 'auth_totp_mail_enforce'],

# AFTER (19.0)
'depends': [..., 'auth_totp_mail'],
```

---

## RULE_008

**CATEGORY:** `MODULE_REMOVE`  
**TRIGGER:** `'depends': [..., 'account_peppol_selfbilling', ...]`  
**ACTION:** Replace with `account_peppol` or `account_peppol_advanced_fields`

```python
# BEFORE (18.0)
'depends': [..., 'account_peppol_selfbilling'],

# AFTER (19.0)
'depends': [..., 'account_peppol'],
```

---

## RULE_009

**CATEGORY:** `MANIFEST`  
**TRIGGER:** `'category': 'Inventory/Inventory'`  
**ACTION:** Update to `'Supply Chain/Inventory'`

```python
# BEFORE (18.0)
'category': 'Inventory/Inventory',

# AFTER (19.0)
'category': 'Supply Chain/Inventory',
```

---

## RULE_010

**CATEGORY:** `MANIFEST`  
**TRIGGER:** `'category': 'Inventory/Purchase'`  
**ACTION:** Update to `'Supply Chain/Purchase'`

```python
# BEFORE (18.0)
'category': 'Inventory/Purchase',

# AFTER (19.0)
'category': 'Supply Chain/Purchase',
```

---

## RULE_011

**CATEGORY:** `MANIFEST`  
**TRIGGER:** `'web.qunit_suite_tests'` in assets  
**ACTION:** Replace with `'web.assets_unit_tests'` and rename test files to `*.test.js`

```python
# BEFORE (18.0)
'web.qunit_suite_tests': [
    'mymodule/static/tests/**/*.js',
    ('remove', 'mymodule/static/tests/tours/**/*'),
],

# AFTER (19.0)
'web.assets_unit_tests': [
    'mymodule/static/tests/**/*.test.js',
],
```

**NOTES:** The legacy QUnit test runner is gone. Tests must be in new format (`.test.js`).

---

## RULE_012

**CATEGORY:** `MANIFEST`  
**TRIGGER:** `'spreadsheet.dependencies'` in assets  
**ACTION:** Replace with `'web.chartjs_lib'`

```python
# BEFORE (18.0)
('include', 'spreadsheet.dependencies'),

# AFTER (19.0)
('include', 'web.chartjs_lib'),
```

---

## RULE_013

**CATEGORY:** `FIELD_RENAME`  
**TRIGGER:** `account.account.deprecated` field access  
**ACTION:** Replace with `active` field (inverted logic)

```python
# BEFORE (18.0)
account.deprecated = True     # mark as deprecated

# AFTER (19.0)
account.active = False        # archive the account

# Search
self.env['account.account'].search([('deprecated', '=', True)])

# AFTER (19.0)
self.env['account.account'].search([('active', '=', False)], active_test=False)
```

**NOTES:** The `deprecated` boolean is gone. Inactive (archived) accounts replace deprecated ones.

---

## RULE_014

**CATEGORY:** `FIELD_RENAME`  
**TRIGGER:** `account.tax.tag.tax_negate` field  
**ACTION:** Replace with `balance_negate`

```python
# BEFORE (18.0)
tag.tax_negate

# AFTER (19.0)
tag.balance_negate
```

---

## RULE_015

**CATEGORY:** `FIELD_RENAME`  
**TRIGGER:** `purchase.order.notes` field  
**ACTION:** Replace with `note`

```python
# BEFORE (18.0)
order.notes = '<p>Terms</p>'
domain = [('notes', '!=', False)]

# AFTER (19.0)
order.note = '<p>Terms</p>'
domain = [('note', '!=', False)]
```

---

## RULE_016

**CATEGORY:** `FIELD_RENAME`  
**TRIGGER:** `product.pricelist.item.product_uom` field  
**ACTION:** Replace with `product_uom_name`

```python
# BEFORE (18.0)
item.product_uom

# AFTER (19.0)
item.product_uom_name
```

---

## RULE_017

**CATEGORY:** `FIELD_RENAME`  
**TRIGGER:** `stock.move.line.product_packaging_id` or `product_packaging_qty`  
**ACTION:** Replace with `packaging_uom_id` and `packaging_uom_qty`

```python
# BEFORE (18.0)
line.product_packaging_id = packaging
line.product_packaging_qty = 3.0

# AFTER (19.0)
line.packaging_uom_id = uom  # packaging is now a uom.uom
line.packaging_uom_qty = 3.0
```

---

## RULE_018

**CATEGORY:** `FIELD_RENAME`  
**TRIGGER:** `account.move.stock_move_id` (Many2one to stock.move)  
**ACTION:** Replace with `stock_move_ids` (One2many)

```python
# BEFORE (18.0)
move.stock_move_id  # Many2one

# AFTER (19.0)
move.stock_move_ids  # One2many
# For single move: move.stock_move_ids[:1]
```

---

## RULE_019

**CATEGORY:** `FIELD_RENAME`  
**TRIGGER:** `uom.uom.factor_inv` field  
**ACTION:** Compute from `factor` (which is now computed from `relative_factor`)

```python
# BEFORE (18.0)
uom.factor_inv  # direct field

# AFTER (19.0)
# factor_inv removed; use: 1.0 / uom.factor if uom.factor else 0
factor_inv = 1.0 / uom.factor if uom.factor else 0.0
```

---

## RULE_020

**CATEGORY:** `FIELD_RENAME`  
**TRIGGER:** `uom.uom.uom_type` field (values: 'bigger', 'reference', 'smaller')  
**ACTION:** Use tree structure — check `relative_uom_id` and `relative_factor`

```python
# BEFORE (18.0)
uom.uom_type in ('bigger', 'reference', 'smaller')

# AFTER (19.0)
# Reference unit: relative_uom_id is False/None
# Others have relative_uom_id set
is_reference = not uom.relative_uom_id
```

---

## RULE_021

**CATEGORY:** `FIELD_RENAME`  
**TRIGGER:** `uom.uom.name` label was 'Unit of Measure' — now 'Unit Name'  
**ACTION:** Update any hardcoded string comparisons or translations

---

## RULE_022

**CATEGORY:** `FIELD_RENAME`  
**TRIGGER:** `product.template.uom_name` label change  
**ACTION:** Still field `uom_name` but `string` changed from `'Unit of Measure Name'` to `'Unit Name'`

---

## RULE_023

**CATEGORY:** `FIELD_RENAME`  
**TRIGGER:** `sale.order.line.sale_line_warn_msg` moved  
**ACTION:** Field moved from `res.partner` (sale) to separate compute

```python
# BEFORE (18.0)
# sale_line_warn_msg was on res.partner via sale module

# AFTER (19.0)
# sale_line_warn_msg is computed on product.product
```

---

## RULE_024

**CATEGORY:** `FIELD_REMOVE`  
**TRIGGER:** `product.packaging` model usage  
**ACTION:** Migrate to UoM-based packaging with `uom.uom` and `product.template.uom_ids`

```python
# BEFORE (18.0)
self.env['product.packaging'].create({
    'name': 'Box of 12',
    'product_id': product.id,
    'qty': 12.0,
})

# AFTER (19.0)
# product.packaging no longer exists
# Use product.template.uom_ids (Many2many to uom.uom)
# with packaging-specific UoMs
```

**NOTES:** This is one of the biggest breaking changes. Any module using `product.packaging`, `product.product.packaging_ids`, or checking `product.packaging` must be fully rewritten.

---

## RULE_025

**CATEGORY:** `FIELD_REMOVE`  
**TRIGGER:** `product.template.uom_po_id` field  
**ACTION:** Purchase UoM is now the same as sales UoM or handled differently

```python
# BEFORE (18.0)
product.uom_po_id  # purchase unit of measure

# AFTER (19.0)
# uom_po_id removed
# Use product.uom_id or packaging UoMs
```

---

## RULE_026

**CATEGORY:** `FIELD_REMOVE`  
**TRIGGER:** `product.template.uom_category_id` field  
**ACTION:** Access via `uom_id.category_id` instead

```python
# BEFORE (18.0)
product.uom_category_id

# AFTER (19.0)
product.uom_id.category_id
```

---

## RULE_027

**CATEGORY:** `FIELD_REMOVE`  
**TRIGGER:** `product.template.packaging_ids` or `product.product.packaging_ids`  
**ACTION:** Use `product.template.uom_ids` for packaging UoMs

```python
# BEFORE (18.0)
product.packaging_ids

# AFTER (19.0)
product.uom_ids  # Many2many of uom.uom used as packagings
```

---

## RULE_028

**CATEGORY:** `FIELD_REMOVE`  
**TRIGGER:** `sale.order.option` model / `sale.order.option_ids`  
**ACTION:** Use `sale.order.line.is_optional` instead

```python
# BEFORE (18.0)
# Optional products as separate records
option = self.env['sale.order.option'].create({
    'order_id': order.id,
    'product_id': product.id,
    'quantity': 1.0,
})

# AFTER (19.0)
# Optional lines are regular order lines with is_optional=True
line = self.env['sale.order.line'].create({
    'order_id': order.id,
    'product_id': product.id,
    'product_uom_qty': 1.0,
    'is_optional': True,
})
```

---

## RULE_029

**CATEGORY:** `FIELD_REMOVE`  
**TRIGGER:** `stock.change.product.qty` wizard usage  
**ACTION:** Use direct quant manipulation or inventory adjustment

```python
# BEFORE (18.0)
self.env['stock.change.product.qty'].create({
    'product_id': product.id,
    'new_quantity': 100.0,
}).change_product_qty()

# AFTER (19.0)
# Use product._reset_inventory() or quant.inventory_quantity_auto_apply
quant = self.env['stock.quant'].search([
    ('product_id', '=', product.id),
    ('location_id', '=', location.id),
])
quant.inventory_quantity = 100.0
quant.action_apply_inventory()
```

---

## RULE_030

**CATEGORY:** `FIELD_REMOVE`  
**TRIGGER:** `stock.quant.package` model reference  
**ACTION:** Replace with `stock.package`

```python
# BEFORE (18.0)
self.env['stock.quant.package']

# AFTER (19.0)
self.env['stock.package']
```

**NOTES:** Access rights CSV also changed: `model_stock_quant_package` → `model_stock_package`.

---

## RULE_031

**CATEGORY:** `FIELD_REMOVE`  
**TRIGGER:** `stock.package_level` model reference  
**ACTION:** Check if `stock.package_history` serves the use case

---

## RULE_032

**CATEGORY:** `FIELD_REMOVE`  
**TRIGGER:** `bus.presence` model usage  
**ACTION:** Use `mail.presence` for IM status; check `res.users.im_status`

```python
# BEFORE (18.0)
self.env['bus.presence'].search([('user_id', '=', uid)])

# AFTER (19.0)
self.env['mail.presence'].search([('user_id', '=', uid)])
# or simply:
user.im_status
```

---

## RULE_033

**CATEGORY:** `FIELD_REMOVE`  
**TRIGGER:** `account.move.line.stock_valuation_layer_ids` field  
**ACTION:** Use `stock.valuation.layer` with its own `account_move_line_id` field

```python
# BEFORE (18.0)
move_line.stock_valuation_layer_ids

# AFTER (19.0)
self.env['stock.valuation.layer'].search([
    ('account_move_line_id', '=', move_line.id)
])
```

---

## RULE_034

**CATEGORY:** `FIELD_REMOVE`  
**TRIGGER:** `crm.team.dashboard_graph_data` field or graph methods  
**ACTION:** Dashboard graph refactored — remove these references

```python
# BEFORE (18.0)
team.dashboard_graph_data
team._compute_dashboard_graph()
team._get_dashboard_graph_data()

# AFTER (19.0) — these are removed
# Use the new action-based dashboard approach
```

---

## RULE_035

**CATEGORY:** `FIELD_REMOVE`  
**TRIGGER:** `sale.order.message_partner_ids` in security domain  
**ACTION:** Use `partner_id` directly in domain rules

```python
# BEFORE (18.0) — ir.rule domain
[('message_partner_ids', 'child_of', [user.commercial_partner_id.id])]

# AFTER (19.0)
[('partner_id', 'child_of', [user.commercial_partner_id.id])]
```

---

## RULE_036

**CATEGORY:** `METHOD_RENAME`  
**TRIGGER:** `payment._handle_notification_data(provider_code, notification_data)`  
**ACTION:** Rename to `_process(provider_code, payment_data)`

```python
# BEFORE (18.0)
def _handle_notification_data(self, provider_code, notification_data):
    ...

# AFTER (19.0)
def _process(self, provider_code, payment_data):
    ...
```

---

## RULE_037

**CATEGORY:** `METHOD_RENAME`  
**TRIGGER:** `payment.provider._get_removal_domain(provider_code, **kwargs)`  
**ACTION:** Rename to `_get_provider_domain(provider_code, **kwargs)`

```python
# BEFORE (18.0)
def _get_removal_domain(self, provider_code, **kwargs):

# AFTER (19.0)
def _get_provider_domain(self, provider_code, **kwargs):
```

---

## RULE_038

**CATEGORY:** `METHOD_RENAME`  
**TRIGGER:** `account_edi_ubl_cii._get_tax_unece_codes(customer, supplier, tax)`  
**ACTION:** Rename to `_get_tax_category_code(customer, supplier, tax)`

```python
# BEFORE (18.0)
def _get_tax_unece_codes(self, customer, supplier, tax):

# AFTER (19.0)
def _get_tax_category_code(self, customer, supplier, tax):
```

---

## RULE_039

**CATEGORY:** `METHOD_RENAME`  
**TRIGGER:** `account_edi._retry_edi_documents_error_hook()`  
**ACTION:** Rename to `_retry_edi_documents_error()`

```python
# BEFORE (18.0)
def _retry_edi_documents_error_hook(self):

# AFTER (19.0)
def _retry_edi_documents_error(self):
```

---

## RULE_040

**CATEGORY:** `METHOD_RENAME`  
**TRIGGER:** `stock_account._stock_account_prepare_anglo_saxon_out_lines_vals()`  
**ACTION:** Rename to `_stock_account_prepare_realtime_out_lines_vals()`

```python
# BEFORE (18.0)
def _stock_account_prepare_anglo_saxon_out_lines_vals(self):

# AFTER (19.0)
def _stock_account_prepare_realtime_out_lines_vals(self):
```

---

## RULE_041

**CATEGORY:** `METHOD_RENAME`  
**TRIGGER:** `mail.thread._get_allowed_message_post_params()`  
**ACTION:** Rename to `_get_allowed_message_params()`

```python
# BEFORE (18.0)
def _get_allowed_message_post_params(self):
    return {'body', 'partner_ids', ...}

# AFTER (19.0)
def _get_allowed_message_params(self):
    return {'body', 'partner_ids', ...}
```

---

## RULE_042

**CATEGORY:** `METHOD_RENAME`  
**TRIGGER:** `mail.thread._get_allowed_message_update_params()`  
**ACTION:** Rename to `_get_allowed_access_params()` for access control; `_get_allowed_message_params()` for message content

---

## RULE_043

**CATEGORY:** `METHOD_RENAME`  
**TRIGGER:** `crm.lead.toggle_active()`  
**ACTION:** Split into `action_unarchive()` and `action_restore()`

```python
# BEFORE (18.0)
lead.toggle_active()

# AFTER (19.0)
lead.action_unarchive()   # un-archive a lost/archived lead
lead.action_restore()     # restore (was toggle_active equivalent)
```

---

## RULE_044

**CATEGORY:** `METHOD_RENAME`  
**TRIGGER:** `stock._get_domain_locations_new()` return type  
**ACTION:** Now type-annotated with `tuple[Domain, Domain, Domain]`

---

## RULE_045

**CATEGORY:** `METHOD_RENAME`  
**TRIGGER:** `analytic._read_group_groupby(groupby_spec, query)`  
**ACTION:** Add `alias` as first argument

```python
# BEFORE (18.0)
def _read_group_groupby(self, groupby_spec: str, query: Query) -> SQL:

# AFTER (19.0)
def _read_group_groupby(self, alias: str, groupby_spec: str, query: Query) -> SQL:
```

---

## RULE_046

**CATEGORY:** `METHOD_RENAME`  
**TRIGGER:** `account.bank.statement.line.read_group()`  
**ACTION:** Use `formatted_read_group()` instead

```python
# BEFORE (18.0)
records.read_group(domain, fields, groupby, ...)

# AFTER (19.0)
records.formatted_read_group(domain, groupby, aggregates, ...)
```

---

## RULE_047

**CATEGORY:** `METHOD_SIG`  
**TRIGGER:** `_compute_reference_prefix(self, provider_code, separator, **values)`  
**ACTION:** Remove `provider_code` argument

```python
# BEFORE (18.0)
def _compute_reference_prefix(self, provider_code, separator, **values):
    ...

# AFTER (19.0)
def _compute_reference_prefix(self, separator, **values):
    # use self.provider_id.code if needed
    ...
```

---

## RULE_048

**CATEGORY:** `METHOD_SIG`  
**TRIGGER:** `_setup_provider(self, provider_code)`  
**ACTION:** Add `**kwargs`

```python
# BEFORE (18.0)
def _setup_provider(self, provider_code):

# AFTER (19.0)
def _setup_provider(self, provider_code, **kwargs):
```

---

## RULE_049

**CATEGORY:** `METHOD_SIG`  
**TRIGGER:** `name_search(self, name='', args=None, ...)`  
**ACTION:** Rename `args` parameter to `domain`

```python
# BEFORE (18.0)
def name_search(self, name='', args=None, operator='ilike', limit=100):
    return super().name_search(name=name, args=args, operator=operator, limit=limit)

# AFTER (19.0)
def name_search(self, name='', domain=None, operator='ilike', limit=100):
    return super().name_search(name=name, domain=domain, operator=operator, limit=limit)
```

**NOTES:** Keyword argument rename. Positional callers will break if they pass args as second positional.

---

## RULE_050

**CATEGORY:** `METHOD_SIG`  
**TRIGGER:** `_notify_get_recipients_groups(self, message, model_description, msg_vals=None)`  
**ACTION:** Change default to `msg_vals=False`

```python
# BEFORE (18.0)
def _notify_get_recipients_groups(self, message, model_description, msg_vals=None):

# AFTER (19.0)
def _notify_get_recipients_groups(self, message, model_description, msg_vals=False):
```

**NOTES:** Logic that does `if msg_vals:` is fine, but `if msg_vals is None:` checks will break.

---

## RULE_051

**CATEGORY:** `METHOD_SIG`  
**TRIGGER:** `_notify_get_recipients(self, message, msg_vals, **kwargs)`  
**ACTION:** Add default `msg_vals=False`

```python
# BEFORE (18.0)
def _notify_get_recipients(self, message, msg_vals, **kwargs):

# AFTER (19.0)
def _notify_get_recipients(self, message, msg_vals=False, **kwargs):
```

---

## RULE_052

**CATEGORY:** `METHOD_SIG`  
**TRIGGER:** `_message_compute_author(self, author_id=None, email_from=None, raise_on_email=True)`  
**ACTION:** Remove `raise_on_email` parameter

```python
# BEFORE (18.0)
def _message_compute_author(self, author_id=None, email_from=None, raise_on_email=True):

# AFTER (19.0)
def _message_compute_author(self, author_id=None, email_from=None):
```

---

## RULE_053

**CATEGORY:** `METHOD_SIG`  
**TRIGGER:** `_notify_thread_by_email(self, message, recipients_data, msg_vals=False, ...)`  
**ACTION:** `msg_vals` is now keyword-only

```python
# BEFORE (18.0)
def _notify_thread_by_email(self, message, recipients_data, msg_vals=False, ...):

# AFTER (19.0)
def _notify_thread_by_email(self, message, recipients_data, *, msg_vals=False, ...):
```

---

## RULE_054

**CATEGORY:** `METHOD_SIG`  
**TRIGGER:** `_notify_by_email_get_base_mail_values(self, message, additional_values=None)`  
**ACTION:** Add `recipients_data` parameter

```python
# BEFORE (18.0)
def _notify_by_email_get_base_mail_values(self, message, additional_values=None):

# AFTER (19.0)
def _notify_by_email_get_base_mail_values(self, message, recipients_data, additional_values=None):
```

---

## RULE_055

**CATEGORY:** `METHOD_SIG`  
**TRIGGER:** `_notify_by_web_push_prepare_payload(self, message, msg_vals=False)`  
**ACTION:** Add `force_record_name=False` parameter

```python
# BEFORE (18.0)
def _notify_by_web_push_prepare_payload(self, message, msg_vals=False):

# AFTER (19.0)
def _notify_by_web_push_prepare_payload(self, message, msg_vals=False, force_record_name=False):
```

---

## RULE_056

**CATEGORY:** `METHOD_SIG`  
**TRIGGER:** `_message_update_content(self, message, body, ...)` (positional)  
**ACTION:** `message` is now positional-only, all others keyword-only

```python
# BEFORE (18.0)
def _message_update_content(self, message, body, attachment_ids=None, partner_ids=None, ...):

# AFTER (19.0)
def _message_update_content(self, message, /, *, body, attachment_ids=None, partner_ids=None, ...):
# message is positional-only; body and others are keyword-only
```

---

## RULE_057

**CATEGORY:** `METHOD_SIG`  
**TRIGGER:** `_thread_to_store(self, store, /, *, fields=None, request_list=None)`  
**ACTION:** `fields` is now required positional (not optional keyword)

```python
# BEFORE (18.0)
def _thread_to_store(self, store: Store, /, *, fields=None, request_list=None):

# AFTER (19.0)
def _thread_to_store(self, store: Store, fields, *, request_list=None):
# fields must be passed explicitly
```

---

## RULE_058

**CATEGORY:** `METHOD_SIG`  
**TRIGGER:** `_get_thread_with_access(self, thread_id, mode="read", **kwargs)`  
**ACTION:** `mode` is now keyword-only

```python
# BEFORE (18.0)
def _get_thread_with_access(self, thread_id, mode="read", **kwargs):

# AFTER (19.0)
def _get_thread_with_access(self, thread_id, *, mode="read", **kwargs):
# Cannot call: obj._get_thread_with_access(123, "write")
# Must use:    obj._get_thread_with_access(123, mode="write")
```

---

## RULE_059

**CATEGORY:** `METHOD_SIG`  
**TRIGGER:** `_get_product_catalog_record_lines(self, product_ids, child_field=False, **kwargs)`  
**ACTION:** Remove `child_field` param; add `section_id` as keyword-only

```python
# BEFORE (18.0)
def _get_product_catalog_record_lines(self, product_ids, child_field=False, **kwargs):

# AFTER (19.0)
def _get_product_catalog_record_lines(self, product_ids, *, section_id=None, **kwargs):
```

---

## RULE_060

**CATEGORY:** `METHOD_SIG`  
**TRIGGER:** `crm.lead._handle_won_lost(self, vals)`  
**ACTION:** Signature completely changed

```python
# BEFORE (18.0)
def _handle_won_lost(self, vals):
    # vals was the write dict

# AFTER (19.0)
def _handle_won_lost(self, old_status_by_lead, new_status_by_lead):
    # dict mapping lead → old/new won_status
```

---

## RULE_061

**CATEGORY:** `METHOD_SIG`  
**TRIGGER:** `crm.lead._find_matching_partner(self, email_only=False)`  
**ACTION:** Remove `email_only` parameter

```python
# BEFORE (18.0)
def _find_matching_partner(self, email_only=False):

# AFTER (19.0)
def _find_matching_partner(self):
```

---

## RULE_062

**CATEGORY:** `METHOD_SIG`  
**TRIGGER:** `stock.move._action_confirm(self, merge=True, merge_into=False)`  
**ACTION:** Add `create_proc=True` parameter

```python
# BEFORE (18.0)
def _action_confirm(self, merge=True, merge_into=False):

# AFTER (19.0)
def _action_confirm(self, merge=True, merge_into=False, create_proc=True):
```

---

## RULE_063

**CATEGORY:** `METHOD_SIG`  
**TRIGGER:** `purchase.order.action_create_invoice(self)`  
**ACTION:** Add `attachment_ids=False` parameter

```python
# BEFORE (18.0)
def action_create_invoice(self):

# AFTER (19.0)
def action_create_invoice(self, attachment_ids=False):
```

---

## RULE_064

**CATEGORY:** `METHOD_SIG`  
**TRIGGER:** `sale.order._send_order_notification_mail(self, mail_template)`  
**ACTION:** Add `allow_deferred_sending=True` parameter

```python
# BEFORE (18.0)
def _send_order_notification_mail(self, mail_template):

# AFTER (19.0)
def _send_order_notification_mail(self, mail_template, allow_deferred_sending=True):
```

---

## RULE_065

**CATEGORY:** `METHOD_SIG`  
**TRIGGER:** `_field_to_sql(self, alias, fname, query, flush=True)`  
**ACTION:** Remove `flush` parameter, rename `fname` to `field_expr`

```python
# BEFORE (18.0)
def _field_to_sql(self, alias: str, fname: str, query=None, flush: bool = True) -> SQL:

# AFTER (19.0)
def _field_to_sql(self, alias: str, field_expr: str, query=None) -> SQL:
```

---

## RULE_066

**CATEGORY:** `SECURITY`  
**TRIGGER:** `<field name="users" .../>` on `res.groups`  
**ACTION:** Rename to `<field name="user_ids" .../>`

```xml
<!-- BEFORE (18.0) -->
<field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>

<!-- AFTER (19.0) -->
<field name="user_ids" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
```

---

## RULE_067

**CATEGORY:** `SECURITY`  
**TRIGGER:** `<field name="groups_id" .../>` on `res.users`  
**ACTION:** May need to use `<field name="group_ids" .../>`

```xml
<!-- BEFORE (18.0) -->
<field name="groups_id" eval="[(4, ref('module.group_name'))]"/>

<!-- AFTER (19.0) — in some contexts -->
<field name="group_ids" eval="[(4, ref('module.group_name'))]"/>
```

---

## RULE_068

**CATEGORY:** `SECURITY`  
**TRIGGER:** `model_mail_wizard_invite` in ir.model.access.csv  
**ACTION:** Replace with `model_mail_followers_edit`

```csv
# BEFORE (18.0)
access_mail_wizard_invite,...,model_mail_wizard_invite,...

# AFTER (19.0)
access_mail_followers_edit,...,model_mail_followers_edit,...
```

---

## RULE_069

**CATEGORY:** `SECURITY`  
**TRIGGER:** `model_mail_resend_message` or `model_mail_resend_partner` in access CSV  
**ACTION:** Remove these entries — models no longer exist

---

## RULE_070

**CATEGORY:** `SECURITY`  
**TRIGGER:** `model_stock_quant_package` in ir.model.access.csv  
**ACTION:** Replace with `model_stock_package`

```csv
# BEFORE (18.0)
access_stock_quant_package_all,...,model_stock_quant_package,base.group_user,1,0,0,0

# AFTER (19.0)
access_stock_package_all,...,model_stock_package,base.group_user,1,0,0,0
```

---

## RULE_071

**CATEGORY:** `SECURITY`  
**TRIGGER:** `model_stock_change_product_qty` or `model_stock_track_confirmation` in access CSV  
**ACTION:** Remove — these wizard models no longer exist

---

## RULE_072

**CATEGORY:** `SECURITY`  
**TRIGGER:** `<record model="ir.module.category" id="base.module_category_accounting_accounting">`  
**ACTION:** Replace with `res.groups.privilege` record

```xml
<!-- BEFORE (18.0) -->
<record model="ir.module.category" id="base.module_category_accounting_accounting">
    <field name="description">...</field>
</record>

<!-- AFTER (19.0) -->
<record model="res.groups.privilege" id="res_groups_privilege_accounting">
    <field name="name">Accounting</field>
    <field name="category_id" ref="base.module_category_accounting"/>
    <field name="sequence">20</field>
    <field name="comment">...</field>
</record>
```

---

## RULE_073

**CATEGORY:** `XML_ID`  
**TRIGGER:** `product.action_packaging_view` XML action reference  
**ACTION:** Packaging view removed — no direct replacement

```xml
<!-- BEFORE (18.0) -->
<menuitem action="product.action_packaging_view" .../>

<!-- AFTER (19.0) — removed; use UoM-based approach -->
```

---

## RULE_074

**CATEGORY:** `XML_ID`  
**TRIGGER:** `stock.menu_product_uom_categ_form_action` reference  
**ACTION:** Update to new menu path under stock config

```xml
<!-- BEFORE (18.0) -->
<menuitem id="menu_product_uom_categ_form_action" parent="stock.menu_stock_config_settings" .../>

<!-- AFTER (19.0) — moved to new location -->
<menuitem id="menu_stock_uom_categ_form_action" .../>
```

---

## RULE_075

**CATEGORY:** `XML_ID`  
**TRIGGER:** `sale.product_packaging_form_view_sale` or `sale.product_packaging_tree_view_sale`  
**ACTION:** These views are removed along with `product.packaging`

---

## RULE_076

**CATEGORY:** `XML_ID`  
**TRIGGER:** `product.product_packaging_tree_view` etc.  
**ACTION:** Removed — product packaging views are gone

---

## RULE_077

**CATEGORY:** `XML_ID`  
**TRIGGER:** `stock.product_packaging_tree_view` etc.  
**ACTION:** Removed

---

## RULE_078

**CATEGORY:** `XML_ID`  
**TRIGGER:** `account.action_move_out_refund_type` action  
**ACTION:** Replaced by `action_move_out_refund_type_non_legacy`

```xml
<!-- BEFORE (18.0) -->
action="account.action_move_out_refund_type"

<!-- AFTER (19.0) -->
action="account.action_move_out_refund_type_non_legacy"
```

---

## RULE_079

**CATEGORY:** `MODEL_RENAME`  
**TRIGGER:** `self.env['bus.listener.mixin']` or `_inherit = 'bus.listener.mixin'`  
**ACTION:** No rename needed — `bus.listener.mixin` still exists, but class was renamed from `BusListenerMixin` to `BusListenerMixin` (Python-class-level only change)

---

## RULE_080

**CATEGORY:** `MODEL_RENAME`  
**TRIGGER:** `self.env['discuss.channel']` (was using Channel class)  
**ACTION:** Model name unchanged (`discuss.channel`); Python class `Channel` → `DiscussChannel`

```python
# No change needed for env usage:
self.env['discuss.channel']  # still works

# If subclassing/inheriting in Python:
# BEFORE
class Channel(models.Model):
    _inherit = 'discuss.channel'

# AFTER
class DiscussChannel(models.Model):
    _inherit = 'discuss.channel'
```

---

## RULE_081

**CATEGORY:** `MODEL_RENAME`  
**TRIGGER:** `self.env['crm.lead']` with Lead class  
**ACTION:** Model name unchanged; Python class `Lead` → `CrmLead`

---

## RULE_082

**CATEGORY:** `MODEL_RENAME`  
**TRIGGER:** `_name = 'uom.uom'` class was `UoM`  
**ACTION:** Update Python class name from `UoM` to `UomUom`

```python
# BEFORE
class UoM(models.Model):
    _inherit = 'uom.uom'

# AFTER
class UomUom(models.Model):
    _inherit = 'uom.uom'
```

---

## RULE_083

**CATEGORY:** `MODEL_RENAME`  
**TRIGGER:** `_name = 'product.pricelist'` class was `Pricelist`  
**ACTION:** Update to `ProductPricelist`

---

## RULE_084

**CATEGORY:** `MODEL_RENAME`  
**TRIGGER:** `_name = 'product.pricelist.item'` class was `PricelistItem`  
**ACTION:** Update to `ProductPricelistItem`

---

## RULE_085

**CATEGORY:** `MODEL_RENAME`  
**TRIGGER:** `self.env['crm.lost.reason']` / `LostReason` class  
**ACTION:** Python class renamed to `CrmLostReason`; model name unchanged

---

## RULE_086

**CATEGORY:** `MODEL_RENAME`  
**TRIGGER:** `self.env['purchase.bill.line.match']` / `PurchaseBillMatch` class  
**ACTION:** Python class renamed to `PurchaseBillLineMatch`; model name unchanged

---

## RULE_087

**CATEGORY:** `MANIFEST`  
**TRIGGER:** Asset references to `web/static/src/legacy/scss/*.scss`  
**ACTION:** Remove legacy SCSS references — legacy SCSS files were removed

```python
# BEFORE (18.0)
'web.assets_backend': [
    'web/static/src/legacy/scss/ui.scss',
    'web/static/src/legacy/scss/dropdown.scss',
    'web/static/src/legacy/scss/fields.scss',
]

# AFTER (19.0) — remove these lines
```

---

## RULE_088

**CATEGORY:** `MANIFEST`  
**TRIGGER:** `'web.assets_frontend'` with `portal.js`, `portal_sidebar.js` etc.  
**ACTION:** Use interactions pattern instead

```python
# BEFORE (18.0) — custom portal JS
'web.assets_frontend': [
    'mymodule/static/src/js/my_portal.js',
    'mymodule/static/src/js/my_portal_sidebar.js',
]

# AFTER (19.0) — use interactions
'web.assets_frontend': [
    'mymodule/static/src/interactions/**/*',
]
```

---

## RULE_089

**CATEGORY:** `MANIFEST`  
**TRIGGER:** References to `mail/static/src/utils/common/**/*`  
**ACTION:** Update to `mail/static/src/**/common/**/*`

```python
# BEFORE (18.0)
"mail/static/src/utils/common/**/*",

# AFTER (19.0)
"mail/static/src/**/common/**/*",
```

---

## RULE_090

**CATEGORY:** `MANIFEST`  
**TRIGGER:** `('include', 'mail.assets_discuss_public')` with specific mail paths  
**ACTION:** Add html_editor assets include

```python
# 19.0 addition in portal
('include', 'html_editor.assets_editor'),
```

---

## RULE_091

**CATEGORY:** `FIELD_RENAME`  
**TRIGGER:** `res.config.settings.module_delivery_fedex`  
**ACTION:** Rename to `module_delivery_fedex_rest`

```python
# BEFORE (18.0)
module_delivery_fedex = fields.Boolean("FedEx Connector")
module_delivery_ups = fields.Boolean("UPS Connector")
module_delivery_usps = fields.Boolean("USPS Connector")

# AFTER (19.0)
module_delivery_fedex_rest = fields.Boolean("FedEx Connector")
module_delivery_ups_rest = fields.Boolean("UPS Connector")
module_delivery_usps_rest = fields.Boolean("USPS Connector")
```

---

## RULE_092

**CATEGORY:** `FIELD_RENAME`  
**TRIGGER:** `stock.move.line.product_packaging_quantity` (different from `product_packaging_qty`)  
**ACTION:** Both removed; use `packaging_uom_qty`

---

## RULE_093

**CATEGORY:** `FIELD_REMOVE`  
**TRIGGER:** `sale.order.line.sale_line_warn` + `sale_line_warn_msg` on `res.partner`  
**ACTION:** Warning text now computed differently; `sale_line_warn` on partner removed

```python
# BEFORE (18.0) — on res.partner (via sale module)
partner.sale_line_warn          # selection field
partner.sale_line_warn_msg      # text field

# AFTER (19.0) — sale_line_warn removed; warn_msg computed on product.product
```

---

## RULE_094

**CATEGORY:** `FIELD_RENAME`  
**TRIGGER:** `product.template.uom_name` string label  
**ACTION:** Label changed from `'Unit of Measure Name'` to `'Unit Name'` — update string searches

---

## RULE_095

**CATEGORY:** `METHOD_SIG`  
**TRIGGER:** `crm.lead._pls_get_lead_pls_values(self, domain=[])`  
**ACTION:** Default changed from `[]` to `None`

```python
# BEFORE (18.0)
def _pls_get_lead_pls_values(self, domain=[]):

# AFTER (19.0)
def _pls_get_lead_pls_values(self, domain=None):
```

---

## RULE_096

**CATEGORY:** `METHOD_SIG`  
**TRIGGER:** `portal.mixin._get_thread_with_access()` usage with positional `hash`/`pid`/`token`  
**ACTION:** Now keyword-only

```python
# BEFORE (18.0)
thread = model._get_thread_with_access(thread_id, token=token)

# AFTER (19.0) — these are keyword-only in portal.mixin override
thread = model._get_thread_with_access(thread_id, hash=hash_val, pid=pid, token=token)
```

---

## RULE_097

**CATEGORY:** `FIELD_REMOVE`  
**TRIGGER:** `hr.employee.base` model inheritance  
**ACTION:** Replace with `hr.employee` inheritance

```python
# BEFORE (18.0) — many modules
class MyModel(models.Model):
    _inherit = 'hr.employee.base'

# AFTER (19.0)
class MyModel(models.Model):
    _inherit = 'hr.employee'
```

---

## RULE_098

**CATEGORY:** `FIELD_REMOVE`  
**TRIGGER:** `account.move.partner_credit` field  
**ACTION:** Use `partner_id.credit` directly

```python
# BEFORE (18.0)
move.partner_credit

# AFTER (19.0)
move.partner_id.credit
```

---

## RULE_099

**CATEGORY:** `MANIFEST`  
**TRIGGER:** Reference to `web/static/src/polyfills/clipboard.js`  
**ACTION:** Replace with `web/static/src/polyfills/**/*.js`

```python
# BEFORE (18.0)
'web/static/src/polyfills/clipboard.js',

# AFTER (19.0)
'web/static/src/polyfills/**/*.js',
```

---

## RULE_100

**CATEGORY:** `FIELD_RENAME`  
**TRIGGER:** `sale.order.option_ids` One2many on `sale.order`  
**ACTION:** Field removed — use `order_line.filtered(lambda l: l.is_optional)`

```python
# BEFORE (18.0)
order.sale_order_option_ids  # One2many to sale.order.option

# AFTER (19.0)
order.order_line.filtered(lambda l: l.is_optional)
```

---

## RULE_101

**CATEGORY:** `FIELD_RENAME`  
**TRIGGER:** `sale.order.template.sale_order_template_option_ids`  
**ACTION:** Use `sale_order_template_line_ids.filtered(lambda l: l.is_optional)`

```python
# BEFORE (18.0)
template.sale_order_template_option_ids

# AFTER (19.0)
template.sale_order_template_line_ids.filtered(lambda l: l.is_optional)
```

---

## RULE_102

**CATEGORY:** `METHOD_SIG`  
**TRIGGER:** `account_edi_ubl_cii._ubl_add_invoice_delivery_nodes(vals)`  
**ACTION:** Rename to `_ubl_add_delivery_nodes(vals)`

```python
# BEFORE (18.0)
def _ubl_add_invoice_delivery_nodes(self, vals):

# AFTER (19.0)
def _ubl_add_delivery_nodes(self, vals):
```

---

## RULE_103

**CATEGORY:** `FIELD_REMOVE`  
**TRIGGER:** `payment.provider.onboarding.wizard` usage  
**ACTION:** Use `provider.action_start_onboarding()` instead

```python
# BEFORE (18.0)
self.env['payment.provider.onboarding.wizard'].create({...}).add_payment_methods()

# AFTER (19.0)
provider.action_start_onboarding()
```

---

## RULE_104

**CATEGORY:** `FIELD_REMOVE`  
**TRIGGER:** `onboarding.onboarding.step` inheritance in payment modules  
**ACTION:** Remove — onboarding step model pattern removed from payment

---

## RULE_105

**CATEGORY:** `FIELD_RENAME`  
**TRIGGER:** `purchase.order.mail_reminder_confirmed` field  
**ACTION:** Replaced by `receipt_reminder_email` (stored, not just computed)

```python
# BEFORE (18.0)
order.mail_reminder_confirmed
order.mail_reception_confirmed
order.mail_reception_declined

# AFTER (19.0) — new acknowledgement flow
order.acknowledged       # vendor has acknowledged
order.receipt_reminder_email   # reminder setting (stored)
```

---

## Summary Table

| Rule ID  | Category        | Key Change                                      |
|----------|-----------------|-------------------------------------------------|
| RULE_001 | IMPORT          | `osv.expression` → `fields.Domain`             |
| RULE_002 | IMPORT          | `tools.OrderedSet` → `tools.misc.OrderedSet`   |
| RULE_004 | MODULE_REMOVE   | `hr_contract` merged into `hr`                  |
| RULE_005 | MODULE_REMOVE   | `web_editor` → `html_editor` / `html_builder`  |
| RULE_011 | MANIFEST        | `qunit_suite_tests` → `assets_unit_tests`      |
| RULE_013 | FIELD_RENAME    | `account.account.deprecated` → `active`        |
| RULE_014 | FIELD_RENAME    | `account.tax.tag.tax_negate` → `balance_negate`|
| RULE_015 | FIELD_RENAME    | `purchase.order.notes` → `note`                |
| RULE_016 | FIELD_RENAME    | `pricelist.item.product_uom` → `product_uom_name` |
| RULE_017 | FIELD_RENAME    | `stock.move.line` packaging fields             |
| RULE_018 | FIELD_RENAME    | `account.move.stock_move_id` → `stock_move_ids`|
| RULE_019 | FIELD_REMOVE    | `uom.uom.factor_inv` removed                   |
| RULE_020 | FIELD_REMOVE    | `uom.uom.uom_type` removed                     |
| RULE_024 | FIELD_REMOVE    | `product.packaging` model removed              |
| RULE_025 | FIELD_REMOVE    | `product.template.uom_po_id` removed           |
| RULE_028 | FIELD_REMOVE    | `sale.order.option` → `sale.order.line.is_optional` |
| RULE_029 | FIELD_REMOVE    | `stock.change.product.qty` wizard removed      |
| RULE_030 | FIELD_REMOVE    | `stock.quant.package` → `stock.package`        |
| RULE_032 | FIELD_REMOVE    | `bus.presence` → `mail.presence`               |
| RULE_033 | FIELD_REMOVE    | `account.move.line.stock_valuation_layer_ids` removed |
| RULE_036 | METHOD_RENAME   | `_handle_notification_data` → `_process`       |
| RULE_038 | METHOD_RENAME   | `_get_tax_unece_codes` → `_get_tax_category_code` |
| RULE_040 | METHOD_RENAME   | Anglo-Saxon → realtime in stock_account        |
| RULE_041 | METHOD_RENAME   | `_get_allowed_message_post_params` → `_get_allowed_message_params` |
| RULE_047 | METHOD_SIG      | `_compute_reference_prefix` loses `provider_code` |
| RULE_049 | METHOD_SIG      | `name_search(args=)` → `name_search(domain=)`  |
| RULE_050 | METHOD_SIG      | `msg_vals=None` → `msg_vals=False` throughout  |
| RULE_056 | METHOD_SIG      | `_message_update_content` now positional-only first arg |
| RULE_057 | METHOD_SIG      | `_thread_to_store(fields=None)` → `fields` required |
| RULE_058 | METHOD_SIG      | `_get_thread_with_access(mode=)` → keyword-only |
| RULE_066 | SECURITY        | `res.groups.users` → `res.groups.user_ids`     |
| RULE_068 | SECURITY        | `mail_wizard_invite` → `mail_followers_edit`   |
| RULE_070 | SECURITY        | `stock.quant.package` → `stock.package` access |
| RULE_091 | FIELD_RENAME    | Delivery connector fields get `_rest` suffix   |
| RULE_097 | FIELD_REMOVE    | `hr.employee.base` → `hr.employee`             |
