# Odoo 18.0 → 19.0 Migration Report

**Generated from:** `git diff origin/18.0 origin/19.0`  
**Scope:** All `addons/` modules with meaningful code changes  
**Total files changed:** ~3,972 files across 150+ modules

---

## Table of Contents

1. [Overview & Statistics](#overview--statistics)
2. [Removed / Merged Modules](#removed--merged-modules)
3. [Core Framework Changes](#core-framework-changes)
4. [Accounting (account, account_payment, payment, account_edi)](#accounting)
5. [Sales & CRM](#sales--crm)
6. [Inventory & Purchasing](#inventory--purchasing)
7. [Human Resources](#human-resources)
8. [Project & Timesheet](#project--timesheet)
9. [Messaging & Communication (mail, bus)](#messaging--communication)
10. [Website & Portal](#website--portal)
11. [Authentication](#authentication)
12. [UoM (Unit of Measure)](#uom-unit-of-measure)
13. [Utilities (base_automation, analytic, digest, spreadsheet)](#utilities)
14. [Security Model Changes](#security-model-changes)
15. [JavaScript / OWL Front-End Changes](#javascript--owl-front-end-changes)
16. [Summary of Breaking Changes](#summary-of-breaking-changes)

---

## Overview & Statistics

### Top 20 Most Changed Modules (by file count)

| Rank | Module              | Files Changed |
|------|---------------------|---------------|
| 1    | mail                | 397           |
| 2    | web                 | 350           |
| 3    | website             | 338           |
| 4    | odoo/base           | 230           |
| 5    | stock               | 194           |
| 6    | account             | 193           |
| 7    | point_of_sale       | 159           |
| 8    | sale                | 158           |
| 9    | html_builder        | 158 (NEW)     |
| 10   | hr                  | 151           |
| 11   | mrp                 | 144           |
| 12   | html_editor         | 144 (NEW)     |
| 13   | project             | 129           |
| 14   | product             | 129           |
| 15   | website_sale        | 128           |
| 16   | crm                 | 128           |
| 17   | payment             | 124           |
| 18   | survey              | 120           |
| 19   | hr_holidays         | 116           |
| 20   | im_livechat         | 114           |

---

## Removed / Merged Modules

### ⚠️ BREAKING: Fully Removed Modules

| Module                          | Status   | Replacement / Notes                                   |
|---------------------------------|----------|-------------------------------------------------------|
| `account_edi_ubl_cii_tax_extension` | REMOVED | Merged into `account_edi_ubl_cii`                 |
| `account_peppol_selfbilling`    | REMOVED  | Merged into `account_peppol`                          |
| `auth_totp_mail_enforce`        | REMOVED  | Merged into `auth_totp_mail`                          |
| `hr_contract`                   | REMOVED  | Merged into `hr` (contracts now live in `hr`)         |
| `hr_holidays_contract`          | REMOVED  | Merged into `hr` / `hr_holidays`                      |
| `web_editor`                    | REPLACED | Split into `html_editor` (core) + `html_builder` (website) |

### ✨ NEW Modules

| Module                      | Description                                              |
|-----------------------------|----------------------------------------------------------|
| `html_editor`               | Standalone rich-text HTML editor (extracted from `web_editor`) |
| `html_builder`              | Website visual page builder (extracted from `web_editor`) |
| `account_peppol_advanced_fields` | Advanced PEPPOL fields (merged from `account_peppol_selfbilling`) |
| `auth_passkey_portal`       | Passkey authentication for portal users                   |
| `cloud_storage_migration`   | Cloud storage migration tooling                           |
| `certificate`               | Certificate management module                             |

---

## Core Framework Changes

### Python Imports

#### ⚠️ BREAKING: `expression` module moved

```python
# 18.0
from odoo.osv import expression
domain = expression.AND([domain1, domain2])

# 19.0
from odoo.fields import Domain
domain = Domain.AND([domain1, domain2])
```

#### ⚠️ BREAKING: `OrderedSet` import location changed

```python
# 18.0
from odoo.tools import OrderedSet

# 19.0
from odoo.tools.misc import OrderedSet
```

#### ✨ NEW: `Command` importable from `odoo.fields`

```python
# 18.0
from odoo import Command

# 19.0 (also valid)
from odoo.fields import Command
```

#### ✨ NEW: `Domain` class available from `odoo.fields`

```python
from odoo.fields import Domain, Command
```

### ORM API Changes

#### ⚠️ BREAKING: `name_search()` signature

```python
# 18.0
def name_search(self, name='', args=None, operator='ilike', limit=100):

# 19.0
def name_search(self, name='', domain=None, operator='ilike', limit=100):
```

#### ⚠️ BREAKING: `default_get()` parameter rename

The positional parameter was renamed from `fields_list` to `fields`. This is only breaking if overrides call `super().default_get(fields_list=...)` using the keyword argument.

```python
# 18.0
def default_get(self, fields_list):

# 19.0
def default_get(self, fields):
```

#### Style change: `write()` parameter name

The parameter was renamed from `values` to `vals` (convention alignment). This is **not a breaking change** unless callers use the keyword argument `write(values=...)`.

```python
# 18.0
def write(self, values):

# 19.0
def write(self, vals):
```

#### ✨ NEW: `web_save_multi()` bulk save method

```python
def web_save_multi(self, vals_list: list[dict], specification: dict[str, dict]) -> list[dict]:
```

#### ✨ NEW: `web_read_group()` signature extended

```python
# 19.0 adds formatted_read_grouping_sets() and formatted_read_group()
```

#### ✨ NEW: `_field_to_sql()` signature

```python
# 18.0
def _field_to_sql(self, alias: str, fname: str, query=None, flush: bool = True) -> SQL:

# 19.0
def _field_to_sql(self, alias: str, field_expr: str, query=None) -> SQL:
```

#### ✨ NEW: `web_name_search()` method on `BaseModel`

```python
def web_name_search(self, name, specification, domain=None, operator='ilike', limit=100):
```

### Class Naming Conventions

19.0 enforces consistent PascalCase class names matching model names. Many classes were renamed:

| Module  | Old Class Name       | New Class Name           |
|---------|----------------------|--------------------------|
| `web`   | `Http`               | `IrHttp`                 |
| `web`   | `Image`              | `IrQwebFieldImage`       |
| `web`   | `ImageUrlConverter`  | `IrQwebFieldImage_Url`   |
| `web`   | `View`               | `IrUiView`               |
| `bus`   | `ImBus`              | `BusBus`                 |
| `bus`   | `Http` (mixin)       | `IrHttp`                 |
| `mail`  | `Channel`            | `DiscussChannel`         |
| `crm`   | `Lead`               | `CrmLead`                |
| `crm`   | `LeadScoringFrequency` | `CrmLeadScoringFrequency` |
| `crm`   | `FrequencyField`     | `CrmLeadScoringFrequencyField` |
| `crm`   | `LostReason`         | `CrmLostReason`          |
| `crm`   | `RecurringPlan`      | `CrmRecurringPlan`       |
| `crm`   | `Stage` (tag)        | `CrmTag`                 |
| `uom`   | `UoM`                | `UomUom`                 |
| `stock` | `Product`            | `ProductProduct`         |
| `stock` | `UoM`                | `UomUom`                 |
| `stock` | `RemovalStrategy`    | `ProductRemoval`         |
| `stock` | `Company`            | `ResCompany`             |
| `portal`| `IrQWeb`             | `IrQweb`                 |
| `portal`| `View`               | `IrUiView`               |
| `portal`| `APIKeyDescription`  | `ResUsersApikeysDescription` |
| `website` | `Assets`           | `WebsiteAssets`          |
| `website` | `MergePartnerAutomatic` | `BasePartnerMergeAutomaticWizard` |
| `website` | `ServerAction`     | `IrActionsServer`        |
| `website` | `Attachment`       | `IrAttachment`           |
| `website` | `Http`             | `IrHttp`                 |
| `website` | `BaseModel`        | `Base`                   |
| `auth_signup` | `Http`         | `IrHttp`                 |
| `auth_totp` | `AuthTotpDevice` | `Auth_TotpDevice`        |
| `auth_totp` | `Users`          | `ResUsers`               |
| `digest`  | `Digest`           | `DigestDigest`           |
| `analytic`| `AnalyticPlanFields` | `AnalyticPlanFieldsMixin` |
| `base_automation` | `ServerAction` | `IrActionsServer`    |
| `sales_team` | `Tag` (crm.tag) | `CrmTag`               |

### Wizard Class Renames

| Old Class Name             | New Class Name                    |
|----------------------------|-----------------------------------|
| `AutomaticEntryWizard`     | `AccountAutomaticEntryWizard`     |
| `AutoPostBillsWizard`      | `AccountAutopostBillsWizard`      |
| `MassCancelOrders`         | `SaleMassCancelOrders`            |
| `ProductLabelLayout`       | `PickingLabelType` / `LotLabelLayout` |
| `ChooseDestinationLocation`| `StockPackageDestination`         |
| `ReturnPickingLine`        | `StockReturnPickingLine`          |
| `ReturnPicking`            | `StockReturnPicking`              |
| `CrmUpdateProbabilities`   | `CrmLeadPlsUpdate`                |
| `Lead2OpportunityPartner`  | `CrmLead2opportunityPartner`      |
| `Lead2OpportunityMassConvert` | `CrmLead2opportunityPartnerMass` |
| `MergeOpportunity`         | `CrmMergeOpportunity`             |
| `ProjectStageDelete`       | `ProjectProjectStageDeleteWizard` |
| `ProjectSharingCollaboratorWizard` | `ProjectShareCollaboratorWizard` |
| `ProjectTaskTypeDelete`    | `ProjectTaskTypeDeleteWizard`     |

---

## Accounting

### `account` module (version 1.3 → 1.4)

#### ⚠️ BREAKING: Field changes on `account.account`

| Field              | Change                                                  |
|--------------------|---------------------------------------------------------|
| `deprecated`       | **REMOVED** — replaced by `active` (Boolean, default=True) |
| `description`      | ✨ NEW — translatable Text field                        |
| `allowed_journal_ids` | Removed `_constrains_allowed_journal_ids` constraint |

```python
# 18.0 — account was "deprecated"
account.deprecated = True

# 19.0 — account is now archived
account.active = False
```

#### ⚠️ BREAKING: `account.account` now inherits `mail.activity.mixin`

```python
# 19.0
_inherit = ['mail.thread', 'mail.activity.mixin']
```

#### ⚠️ BREAKING: `account.account._get_most_frequent_accounts_for_partner()` signature

```python
# 18.0
def _get_most_frequent_accounts_for_partner(self, company_id, partner_id, move_type,
    filter_never_user_accounts=False, limit=None, journal_id=None):

# 19.0 — journal_id removed
def _get_most_frequent_accounts_for_partner(self, company_id, partner_id, move_type,
    filter_never_user_accounts=False, limit=None):
```

#### ⚠️ BREAKING: `account.tax.tag` field renames

| Old Field     | New Field                |
|---------------|--------------------------|
| `tax_negate`  | `balance_negate` (computed) |
| —             | `report_expression_id` (NEW computed) |

```python
# 18.0
tag.tax_negate

# 19.0
tag.balance_negate
```

#### ✨ NEW: `account.move` fields

| Field                            | Description                              |
|----------------------------------|------------------------------------------|
| `journal_line_ids`               | One2many to journal lines                |
| `exchange_diff_partial_ids`      | Partial exchange differences             |
| `adjusting_entry_origin_move_ids`| Adjusting entry origins                  |
| `adjusting_entries_move_ids`     | Adjusting entries                        |
| `no_followup`                    | Suppress follow-up                       |
| `taxable_supply_date`            | Taxable supply date                      |
| `show_taxable_supply_date`       | Computed visibility                      |
| `alerts`                         | Json alerts field                        |
| `display_send_button`            | Computed button visibility               |
| `is_storno`                      | Now computed (was static Boolean)        |

#### ⚠️ BREAKING: `account.move` field change

```python
# 18.0
is_storno = fields.Boolean(...)

# 19.0 — now computed
is_storno = fields.Boolean(compute='_compute_is_storno')
```

#### ⚠️ BREAKING: `account.bank.statement.line.formatted_read_group()` replaces `read_group()`

```python
# 18.0
def read_group(self, domain, fields, groupby, offset=0, limit=None, orderby=False, lazy=True):

# 19.0
def formatted_read_group(self, domain, groupby=(), aggregates=(), having=(), offset=0, limit=None, order=None):
```

#### ⚠️ BREAKING: `account.edi.common._get_tax_unece_codes()` renamed

```python
# 18.0
def _get_tax_unece_codes(self, customer, supplier, tax):

# 19.0
def _get_tax_category_code(self, customer, supplier, tax):
```

#### ⚠️ BREAKING: `account_edi` method rename

```python
# 18.0
def _retry_edi_documents_error_hook(self):

# 19.0
def _retry_edi_documents_error(self):
```

#### ⚠️ BREAKING: `account_edi_ubl_cii._import_partner()` signature

```python
# 18.0
def _import_partner(self, company_id, name, phone, email, vat, country_code=False,
    peppol_eas=False, peppol_endpoint=False, street=False, street2=False,
    city=False, zip_code=False):

# 19.0 — uses keyword-only args and postal_address dict
def _import_partner(self, company_id, name, phone, email, vat, *, peppol_eas=False,
    peppol_endpoint=False, postal_address={}, **kwargs):
```

#### ⚠️ BREAKING: UBL invoice import methods refactored

All `_import_ubl_invoice_*` and `_import_ubl_invoice_line_*` methods were removed and replaced with a cleaner pipeline approach using `_import_lines()`.

### `payment` module

#### ⚠️ BREAKING: Payment transaction processing renamed

```python
# 18.0
def _handle_notification_data(self, provider_code, notification_data):
def _get_tx_from_notification_data(self, provider_code, notification_data):

# 19.0
def _process(self, provider_code, payment_data):
```

#### ⚠️ BREAKING: `_compute_reference_prefix()` signature

```python
# 18.0
def _compute_reference_prefix(self, provider_code, separator, **values):

# 19.0 — provider_code removed
def _compute_reference_prefix(self, separator, **values):
```

#### ⚠️ BREAKING: `_setup_provider()` signature

```python
# 18.0
def _setup_provider(self, provider_code):

# 19.0
def _setup_provider(self, provider_code, **kwargs):
```

#### ⚠️ BREAKING: `_get_removal_domain()` renamed

```python
# 18.0
def _get_removal_domain(self, provider_code, **kwargs):

# 19.0
def _get_provider_domain(self, provider_code, **kwargs):
```

#### ⚠️ BREAKING: Capture/void/refund request methods

```python
# 18.0
def _send_refund_request(self, amount_to_refund=None):
def _send_capture_request(self, amount_to_capture=None):
def _send_void_request(self, amount_to_void=None):

# 19.0 — amount params removed from send methods; new high-level methods added
def _send_refund_request(self):
def _send_capture_request(self):
def _send_void_request(self):
def _capture(self, amount_to_capture=None):  # NEW high-level
def _void(self, amount_to_void=None):         # NEW high-level
def _refund(self, amount_to_refund=None):     # NEW high-level
def _charge_with_token(self):                 # NEW
```

#### ✨ NEW: `payment.provider` fields/methods

| Field / Method                  | Description                              |
|---------------------------------|------------------------------------------|
| `support_manual_capture`        | Selection field for manual capture       |
| `is_live`                       | Computed Boolean                         |
| `_send_api_request()`           | Generic API request helper               |
| `_build_request_url()`          | Build provider URL                       |
| `_build_request_headers()`      | Build request headers                    |
| `_build_request_auth()`         | Build authentication                     |
| `action_start_onboarding()`     | Replaced onboarding wizard               |
| `action_reset_credentials()`    | NEW                                      |

#### ⚠️ BREAKING: Onboarding wizard removed

The `payment.provider.onboarding.wizard` and related `onboarding.onboarding.step` models were removed. Use `action_start_onboarding()` instead.

---

## Sales & CRM

### `sale` module

#### ✨ NEW: `sale.order` fields

| Field                             | Description                                  |
|-----------------------------------|----------------------------------------------|
| `pending_email_template_id`       | Template for pending confirmation email      |
| `preferred_payment_method_line_id`| Preferred payment method                     |
| `sale_warning_text`               | Computed warning text from partner/product   |
| `has_authorized_transaction_ids`  | Computed Boolean                             |

#### ⚠️ BREAKING: `sale.order._send_order_notification_mail()` signature

```python
# 18.0
def _send_order_notification_mail(self, mail_template):

# 19.0
def _send_order_notification_mail(self, mail_template, allow_deferred_sending=True):
```

#### ⚠️ BREAKING: `sale.order._notify_get_recipients_groups()` signature

```python
# 18.0
def _notify_get_recipients_groups(self, message, model_description, msg_vals=None):

# 19.0
def _notify_get_recipients_groups(self, message, model_description, msg_vals=False):
```

#### ⚠️ BREAKING: `sale.order` portal record filter domain changed

```python
# 18.0 security rule
[('message_partner_ids', 'child_of', [user.commercial_partner_id.id])]

# 19.0
[('partner_id', 'child_of', [user.commercial_partner_id.id])]
```

#### ⚠️ BREAKING: `sale.order.option` model REMOVED

The `sale.order.option` model is removed. Optional products are now handled via `is_optional` field on `sale.order.line`.

```python
# 18.0 — separate model
class SaleOrderOption(models.Model):
    _name = 'sale.order.option'
    order_id = fields.Many2one('sale.order', ...)
    ...

# 19.0 — field on sale.order.line
class SaleOrderLine(models.Model):
    is_optional = fields.Boolean(...)
```

#### ⚠️ BREAKING: `sale.order.template.option` model REMOVED

Replaced by `is_optional` on `sale.order.template.line`.

#### ⚠️ BREAKING: `sale.order.cancel` wizard REMOVED

```python
# 18.0
class SaleOrderCancel(models.TransientModel):
    _name = 'sale.order.cancel'
```

#### ✨ NEW: `sale.order._get_edi_builders()` (EDI integration)

#### ⚠️ BREAKING: `sale.order._get_product_catalog_record_lines()` signature

```python
# 18.0
def _get_product_catalog_record_lines(self, product_ids, **kwargs):

# 19.0
def _get_product_catalog_record_lines(self, product_ids, *, section_id=None, **kwargs):
```

#### ⚠️ BREAKING: `product` field digit precision

```python
# 18.0
sales_count = fields.Float(digits='Product Unit of Measure')
purchased_product_qty = fields.Float(digits='Product Unit of Measure')

# 19.0
sales_count = fields.Float(digits='Product Unit')
purchased_product_qty = fields.Float(digits='Product Unit')
```

#### ⚠️ BREAKING: `sale.order.line.sale_order_line_id` index added

### `sale_management` module

#### ⚠️ BREAKING: `sale.order.template.line` restructured

```python
# 19.0 — new fields
is_optional = fields.Boolean(...)
allowed_uom_ids = fields.Many2many('uom.uom', compute='_compute_allowed_uom_ids')
parent_id = fields.Many2one(...)
```

#### ⚠️ BREAKING: `ProductCategory.property_account_downpayment_categ_id` REMOVED

Replaced by `downpayment_account_id` on product template.

### `crm` module (version 1.8 → 1.9)

#### ✨ NEW: `crm.lead` fields

| Field                  | Description                                |
|------------------------|--------------------------------------------|
| `stage_id_color`       | Related to stage color                     |
| `commercial_partner_id`| Computed commercial partner                |
| `won_status`           | Selection: won/lost/None (replaces flags)  |

#### ⚠️ BREAKING: `crm.lead.active` tracking value changed

```python
# 18.0
active = fields.Boolean('Active', default=True, tracking=True)

# 19.0 — numeric tracking level
active = fields.Boolean('Active', default=True, tracking=72)
```

#### ⚠️ BREAKING: `crm.lead._handle_won_lost()` signature

```python
# 18.0
def _handle_won_lost(self, vals):

# 19.0
def _handle_won_lost(self, old_status_by_lead, new_status_by_lead):
```

#### ⚠️ BREAKING: `crm.lead._find_matching_partner()` signature

```python
# 18.0
def _find_matching_partner(self, email_only=False):

# 19.0 — email_only removed
def _find_matching_partner(self):
```

#### ⚠️ BREAKING: `crm.lead.toggle_active()` renamed

```python
# 18.0
def toggle_active(self):

# 19.0 — split into two methods
def action_unarchive(self):
def action_restore(self):
```

#### ⚠️ BREAKING: `crm.lead.search_fetch()` parameter made optional

```python
# 18.0
def search_fetch(self, domain, field_names, offset=0, limit=None, order=None):

# 19.0 — field_names is now optional
def search_fetch(self, domain, field_names=None, offset=0, limit=None, order=None):
```

#### ⚠️ BREAKING: `crm.team` graph methods removed

The following `crm.team` methods were removed (dashboard graph refactored):
- `_compute_dashboard_graph()`
- `dashboard_graph_data` field
- `_graph_get_model()`, `_graph_get_dates()`, `_graph_date_column()`, `_graph_get_table()`
- `_graph_x_query()`, `_graph_y_query()`, `_extra_sql_conditions()`
- `_graph_title_and_key()`, `_graph_data()`, `_get_dashboard_graph_data()`

#### ⚠️ BREAKING: `crm.team.mobile` field removed

```python
# 18.0
mobile = fields.Char(string='Mobile', related='user_id.mobile')

# 19.0 — removed
```

### `product` module

#### ⚠️ BREAKING: `product.packaging` model REMOVED

The `product.packaging` model (previously defined in `product`) has been removed entirely. Packaging is now handled through UoM (`uom.uom`) with the new `uom_ids` field on product template.

```python
# 18.0
packaging = self.env['product.packaging'].create({
    'name': 'Box of 12',
    'product_id': product.id,
    'qty': 12,
})

# 19.0 — use uom.uom with packaging flag
```

#### ⚠️ BREAKING: `product.template` and `product.product` field changes

| Old Field                  | New Field / Change                          |
|----------------------------|---------------------------------------------|
| `packaging_ids`            | REMOVED (use `uom_ids`)                     |
| `pricelist_item_count`     | REMOVED                                     |
| `uom_category_id`          | REMOVED                                     |
| `uom_po_id`                | REMOVED (purchase UoM merged into UoM)      |
| `uom_name`                 | Still present, label changed to `'Unit Name'` |
| —                          | `uom_ids` (Many2many to uom.uom) — NEW      |
| —                          | `pricelist_rule_ids` — NEW                  |
| —                          | `import_attribute_values` — NEW             |
| —                          | `is_dynamically_created` — NEW              |
| —                          | `product_uom_ids` (unit barcodes) — NEW     |

#### ⚠️ BREAKING: `product.pricelist` renamed class, field changes

```python
# Old class name: Pricelist
# New class name: ProductPricelist

# product.pricelist.item old class: PricelistItem
# new class: ProductPricelistItem
```

#### ⚠️ BREAKING: `product.pricelist.item` field changes

| Old Field           | New Field / Change                                   |
|---------------------|------------------------------------------------------|
| `product_uom`       | Renamed to `product_uom_name`                        |
| `company_id`        | Now computed (`_compute_company_id`), not related    |
| `currency_id`       | Now computed (`_compute_currency_id`), not related   |

#### ⚠️ BREAKING: `product.pricelist.item._compute_price()` signature

```python
# 18.0
def _compute_price(self, product, quantity, uom, date, currency=None):

# 19.0
def _compute_price(self, product, quantity, uom, date, currency=None, **kwargs):
```

#### ⚠️ BREAKING: `product.category._unlink_except_default_category()` removed

Replaced by `copy_data()`.

#### ⚠️ BREAKING: `decimal.precision` mixin removed from `product`

`product` no longer inherits `decimal.precision` to check `_check_main_currency_rounding`.

---

## Inventory & Purchasing

### `stock` module

#### ⚠️ BREAKING: `stock.move.line` packaging fields replaced

```python
# 18.0
product_packaging_id = fields.Many2one('product.packaging', ...)
product_packaging_qty = fields.Float(...)
product_packaging_quantity = fields.Float(...)

# 19.0 — UoM-based packaging
packaging_uom_id = fields.Many2one('uom.uom', ...)
packaging_uom_qty = fields.Float(...)
```

#### ⚠️ BREAKING: `stock.move.line` forecast digits

```python
# 18.0
forecast_availability = fields.Float(digits='Product Unit of Measure', ...)

# 19.0
forecast_availability = fields.Float(digits='Product Unit', ...)
```

#### ⚠️ BREAKING: `stock.quant.package` replaced by `stock.package`

The model `stock.quant.package` was renamed to `stock.package`. All access rights and references must be updated.

#### ⚠️ BREAKING: `stock.package_level` replaced by `stock.package_history`

New view/model for package history.

#### ⚠️ BREAKING: `stock.change.product.qty` wizard REMOVED

```python
# 18.0
class ProductChangeQuantity(models.TransientModel):
    _name = 'stock.change.product.qty'

# 19.0 — use direct quant manipulation
```

#### ⚠️ BREAKING: `stock.track.confirmation` wizard REMOVED

#### ✨ NEW: `stock.put.in.pack` wizard

```python
class StockPutInPack(models.TransientModel):
    _name = 'stock.put.in.pack'
```

#### ✨ NEW: `stock.picking` fields

| Field                            | Description                         |
|----------------------------------|-------------------------------------|
| `package_ids`                    | One2many to `stock.package`         |
| `has_lines_without_result_package`| Computed Boolean                   |
| `is_date_editable`               | Computed Boolean                    |
| `reference_ids`                  | Many2many references                |
| `inventory_name`                 | Readonly Char                       |

#### ✨ NEW: `stock.move` field

| Field                  | Description                           |
|------------------------|---------------------------------------|
| `procurement_values`   | Json store=False for procurement data |
| `reference_ids`        | Many2many references                  |

#### ⚠️ BREAKING: `stock.move._action_confirm()` signature

```python
# 18.0
def _action_confirm(self, merge=True, merge_into=False):

# 19.0
def _action_confirm(self, merge=True, merge_into=False, create_proc=True):
```

#### ✨ NEW: `res.company` (stock) fields

| Field                       | Description                              |
|-----------------------------|------------------------------------------|
| `horizon_days`              | Replenishment horizon in days (default 365) |
| `stock_text_confirmation`   | Enable SMS/text confirmation             |
| `stock_confirmation_type`   | Type: 'sms'                              |
| `module_delivery_fedex_rest`| FedEx REST connector (replaces `module_delivery_fedex`) |
| `module_delivery_ups_rest`  | UPS REST connector (replaces `module_delivery_ups`) |
| `module_delivery_usps_rest` | USPS REST connector                      |
| `module_delivery_envia`     | Envia.com connector (NEW)               |

#### ⚠️ BREAKING: Delivery connector field renames on `res.config.settings`

```python
# 18.0
module_delivery_fedex = fields.Boolean(...)
module_delivery_ups = fields.Boolean(...)
module_delivery_usps = fields.Boolean(...)

# 19.0
module_delivery_fedex_rest = fields.Boolean(...)
module_delivery_ups_rest = fields.Boolean(...)
module_delivery_usps_rest = fields.Boolean(...)
```

#### ⚠️ BREAKING: Stock category changed

```python
# 18.0
'category': 'Inventory/Inventory'

# 19.0
'category': 'Supply Chain/Inventory'
```

### `stock_account` module

#### ⚠️ BREAKING: `account.move.stock_move_id` → `stock_move_ids`

```python
# 18.0
stock_move_id = fields.Many2one('stock.move', ...)

# 19.0 — one2many
stock_move_ids = fields.One2many('stock.move', 'account_move_id', ...)
```

#### ⚠️ BREAKING: `account.move.line.stock_valuation_layer_ids` REMOVED

#### ⚠️ BREAKING: Anglo-Saxon method rename

```python
# 18.0
def _stock_account_prepare_anglo_saxon_out_lines_vals(self):
def _stock_account_anglo_saxon_reconcile_valuation(self, product=False):

# 19.0
def _stock_account_prepare_realtime_out_lines_vals(self):
# reconcile method removed
```

#### ✨ NEW: `account.account` stock variation fields

```python
account_stock_variation_id = fields.Many2one(...)
account_stock_expense_id = fields.Many2one(...)
```

#### ⚠️ BREAKING: `product.template` stock valuation fields are now computed

```python
# 18.0 — related to categ_id
cost_method = fields.Selection(related="categ_id.property_cost_method", ...)
valuation = fields.Selection(related="categ_id.property_valuation", ...)

# 19.0 — computed with proper search
cost_method = fields.Selection(compute='_compute_cost_method', ...)
valuation = fields.Selection(compute='_compute_valuation', ...)
```

#### ⚠️ BREAKING: SVL compute fields consolidated

```python
# 18.0
value_svl = fields.Float(compute='_compute_value_svl', ...)
quantity_svl = fields.Float(compute='_compute_value_svl', ...)
avg_cost = fields.Monetary(compute='_compute_value_svl', ...)
total_value = fields.Monetary(compute='_compute_value_svl', ...)

# 19.0 — simplified
avg_cost = fields.Monetary(compute='_compute_value', ...)
total_value = fields.Monetary(compute='_compute_value', ...)
# value_svl and quantity_svl removed
```

### `purchase` module

#### ⚠️ BREAKING: `purchase.order.notes` renamed to `note`

```python
# 18.0
notes = fields.Html('Terms and Conditions')

# 19.0
note = fields.Html('Terms and Conditions')
```

#### ⚠️ BREAKING: `purchase.order.origin` label changed

```python
# 18.0
origin = fields.Char('Source Document', ...)

# 19.0
origin = fields.Char('Source', ...)
```

#### ✨ NEW: `purchase.order` fields

| Field                    | Description                              |
|--------------------------|------------------------------------------|
| `acknowledged`           | Vendor acknowledgement flag              |
| `duplicated_order_ids`   | Computed duplicate orders                |
| `receipt_reminder_email` | Now stored (was computed)                |
| `reminder_date_before_receipt` | Now stored (was computed)          |
| `is_late`                | Computed late status                     |
| `show_comparison`        | Show price comparison                    |
| `purchase_warning_text`  | Computed warning text                    |
| `lock_confirmed_po`      | Related to company setting               |
| `locked`                 | Order lock status                        |

#### ✨ NEW: `purchase.order.action_acknowledge()` and `get_acknowledge_url()`

#### ⚠️ BREAKING: `purchase.order.action_create_invoice()` signature

```python
# 18.0
def action_create_invoice(self):

# 19.0
def action_create_invoice(self, attachment_ids=False):
```

#### ⚠️ BREAKING: `purchase.order` now inherits `account.document.import.mixin`

```python
# 19.0
_inherit = ['portal.mixin', 'product.catalog.mixin', 'mail.thread',
            'mail.activity.mixin', 'account.document.import.mixin']
```

#### ⚠️ BREAKING: `purchase.bill.line.match` class renamed

```python
# 18.0
class PurchaseBillMatch(models.Model):
    _name = "purchase.bill.line.match"

# 19.0
class PurchaseBillLineMatch(models.Model):
    _name = 'purchase.bill.line.match'
```

#### ⚠️ BREAKING: `product.packaging` for purchase removed

```python
# 18.0 (in purchase module)
class ProductPackaging(models.Model):
    _inherit = 'product.packaging'
    purchase = fields.Boolean("Purchase", ...)

# 19.0 — removed
```

#### ⚠️ BREAKING: Purchase category changed

```python
# 18.0
'category': 'Inventory/Purchase'

# 19.0
'category': 'Supply Chain/Purchase'
```

---

## Human Resources

### `hr` module

#### ⚠️ BREAKING: `hr_contract` module REMOVED — merged into `hr`

All contract functionality that was in `hr_contract` is now in `hr`. Modules that depended on `hr_contract` must update their manifest:

```python
# 18.0 manifest
'depends': [..., 'hr_contract'],

# 19.0
'depends': [..., 'hr'],
```

#### ⚠️ BREAKING: `hr.employee` major restructuring

The employee model now uses a **versioning system** via `hr.version`:

```python
# 19.0 NEW fields
version_id = fields.Many2one(...)
current_version_id = fields.Many2one(...)
current_date_version = fields.Date(...)
version_ids = fields.One2many(...)
versions_count = fields.Integer(...)
```

#### ⚠️ BREAKING: `hr.employee` private address fields REMOVED

```python
# 18.0 — private address stored on employee
private_street = fields.Char(groups="hr.group_hr_user")
private_city = fields.Char(groups="hr.group_hr_user")
private_state_id = fields.Many2one(...)
private_zip = fields.Char(...)
private_country_id = fields.Many2one(...)
gender = fields.Selection(...)
marital = fields.Selection(...)
spouse_complete_name = fields.Char(...)

# 19.0 — private data moved to version model
```

#### ✨ NEW: `hr.employee` work contact

```python
work_contact_id = fields.Many2one('res.partner', 'Work Contact', copy=False)
work_phone = fields.Char(store=True, compute='_compute_work_contact_details', ...)
work_email = fields.Char(compute='_compute_work_contact_details', store=True, ...)
legal_name = fields.Char(compute='_compute_legal_name', store=True, ...)
```

#### ✨ NEW: `hr.employee` presence fields

```python
hr_presence_state = fields.Selection([...])
hr_icon_display = fields.Selection([...])
show_hr_icon_display = fields.Boolean(...)
newly_hired = fields.Boolean(...)
```

#### ⚠️ BREAKING: `hr.employee.resource_calendar_id` tracking removed

```python
# 18.0
resource_calendar_id = fields.Many2one(tracking=True)

# 19.0 — tracking moved; company_id now has tracking
company_id = fields.Many2one('res.company', required=True, tracking=True)
```

#### ⚠️ BREAKING: `hr.employee_base` model REMOVED

The abstract `hr.employee.base` model was removed. Modules that inherited it must adapt.

#### ✨ NEW: HR wizards

- `hr.bank.account.allocation.wizard` — bank account allocation
- `hr.bank.account.allocation.wizard.line` — allocation lines
- `hr.version.wizard` — contract template wizard

#### ✨ NEW: `hr.contract.template` model (from merged `hr_contract`)

#### ✨ NEW: Views added to `hr`

- `views/hr_version_views.xml`
- `views/hr_contract_template_views.xml`
- `views/res_partner_bank_views.xml`

### `hr_holidays` module

#### ⚠️ BREAKING: `hr.employee.base` model removed

Any module inheriting `hr.employee.base` must update to inherit `hr.employee` and/or `hr.employee.public`.

#### ✨ NEW: Wizard views reorganized

Views moved from wizard files to proper `data/` organization.

---

## Project & Timesheet

### `project` module

#### ✨ NEW: `project.role` model

```python
_name = 'project.role'
```

#### ✨ NEW: Project templates

New template system: `project.template.create.wizard`, `project.template.role.to.users.map`.

#### ✨ NEW: Task sharing wizard

```python
class TaskShareWizard(models.TransientModel):
    _name = 'task.share.wizard'
```

#### ✨ NEW: Milestone views

- Kanban, list, and form views for milestones
- Embedded actions for milestones

#### ⚠️ BREAKING: `project.stage.delete` wizard class renamed

```python
# 18.0
class ProjectStageDelete(models.TransientModel):

# 19.0
class ProjectProjectStageDeleteWizard(models.TransientModel):
```

#### ⚠️ BREAKING: `group_project_rating` removed

The group `project.group_project_rating` was removed.

#### ⚠️ BREAKING: SCSS path change

```python
# 18.0
'web/static/src/core/colorpicker/colorpicker.scss'

# 19.0
'web/static/src/core/color_picker/color_picker.scss'
```

---

## Messaging & Communication

### `mail` module (version 1.18 → 1.19)

#### ✨ NEW: `discuss.call.history` model

```python
class DiscussChannel(models.Model):
    _name = "discuss.call.history"
    channel_id = fields.Many2one("discuss.channel", ...)
    start_dt = fields.Datetime(required=True)
    end_dt = fields.Datetime()
    duration_hour = fields.Float(compute=...)
```

#### ✨ NEW: `discuss.channel` fields

| Field                    | Description                              |
|--------------------------|------------------------------------------|
| `call_history_ids`       | One2many to call history                 |
| `message_count`          | Computed message count                   |
| `self_member_id`         | Computed current member                  |
| `invited_member_ids`     | Computed invited members                 |
| `channel_name_member_ids`| One2many for display name computation    |

#### ⚠️ BREAKING: `discuss.channel.is_member` now `compute_sudo=True`

```python
# 19.0
is_member = fields.Boolean(compute='_compute_is_member', search='_search_is_member', compute_sudo=True)
```

#### ⚠️ BREAKING: `discuss.channel.group_public_id` now `recursive=True`

#### ⚠️ BREAKING: `discuss.channel.add_members()` signature

```python
# 18.0
def add_members(self, partner_ids=None, guest_ids=None, invite_to_rtc_call=False,
                open_chat_window=False, post_joined_message=True):

# 19.0 — new keyword-only arguments
def add_members(self, ...):  # extended signature
```

#### ⚠️ BREAKING: `discuss.channel.message_post()` signature

```python
# 18.0
def message_post(self, *, message_type='notification', **kwargs):

# 19.0
def message_post(self, *, message_type="notification", partner_ids=None, **kwargs):
```

#### ⚠️ BREAKING: `discuss.channel._action_unfollow()` signature

```python
# 18.0
def _action_unfollow(self, partner=None, guest=None):

# 19.0
def _action_unfollow(self, partner=None, guest=None, post_leave_message=True):
```

#### ⚠️ BREAKING: `mail.thread` API changes

```python
# 18.0
def _get_allowed_message_post_params(self):
def _get_allowed_message_update_params(self):

# 19.0
def _get_allowed_message_params(self):     # replaces both
def _get_allowed_access_params(self):      # NEW
```

#### ⚠️ BREAKING: `mail.thread._message_update_content()` signature

```python
# 18.0
def _message_update_content(self, message, body, attachment_ids=None, partner_ids=None, ...):

# 19.0 — positional-only separator
def _message_update_content(self, message, /, *, body, attachment_ids=None, partner_ids=None, ...):
```

#### ⚠️ BREAKING: `mail.thread._notify_get_recipients()` default changed

```python
# 18.0
def _notify_get_recipients(self, message, msg_vals, **kwargs):

# 19.0
def _notify_get_recipients(self, message, msg_vals=False, **kwargs):
```

#### ⚠️ BREAKING: `mail.thread._notify_get_recipients_groups()` default changed

```python
# 18.0
def _notify_get_recipients_groups(self, message, model_description, msg_vals=None):

# 19.0
def _notify_get_recipients_groups(self, message, model_description, msg_vals=False):
```

#### ⚠️ BREAKING: `mail.thread._message_compute_author()` signature

```python
# 18.0
def _message_compute_author(self, author_id=None, email_from=None, raise_on_email=True):

# 19.0 — raise_on_email removed
def _message_compute_author(self, author_id=None, email_from=None):
```

#### ⚠️ BREAKING: `mail.thread._notify_thread_by_email()` signature

```python
# 18.0
def _notify_thread_by_email(self, message, recipients_data, msg_vals=False, ...):

# 19.0 — keyword-only
def _notify_thread_by_email(self, message, recipients_data, *, msg_vals=False, ...):
```

#### ⚠️ BREAKING: `mail.thread._notify_by_email_get_base_mail_values()` signature

```python
# 18.0
def _notify_by_email_get_base_mail_values(self, message, additional_values=None):

# 19.0 — recipients_data added
def _notify_by_email_get_base_mail_values(self, message, recipients_data, additional_values=None):
```

#### ⚠️ BREAKING: `mail.thread._notify_by_web_push_prepare_payload()` signature

```python
# 18.0
def _notify_by_web_push_prepare_payload(self, message, msg_vals=False):

# 19.0
def _notify_by_web_push_prepare_payload(self, message, msg_vals=False, force_record_name=False):
```

#### ⚠️ BREAKING: `mail.thread._thread_to_store()` signature

```python
# 18.0
def _thread_to_store(self, store: Store, /, *, fields=None, request_list=None):

# 19.0 — fields is now positional required
def _thread_to_store(self, store: Store, fields, *, request_list=None):
```

#### ⚠️ BREAKING: `mail.thread._get_thread_with_access()` signature

```python
# 18.0
def _get_thread_with_access(self, thread_id, mode="read", **kwargs):

# 19.0 — mode is keyword-only
def _get_thread_with_access(self, thread_id, *, mode="read", **kwargs):
```

#### ⚠️ BREAKING: `mail.thread._partner_find_from_emails*` methods changed

```python
# 18.0
def _mail_search_on_user(self, normalized_emails, extra_domain=False):
def _mail_search_on_partner(self, normalized_emails, extra_domain=False):
def _message_partner_info_from_emails(self, emails, link_mail=False):

# 19.0 — replaced by
def _partner_find_from_emails_single(self, emails, avoid_alias=True, ban_emails=None, ...):
def _partner_find_from_emails(self, records_emails, avoid_alias=True, ban_emails=None, ...):
```

#### ✨ NEW: `res.role` model

```python
_name = 'res.role'
# Accessible via mail.group_mail_canned_response_admin
```

#### ✨ NEW: Wizard renames in `mail`

| Old Model                   | New Model                  |
|-----------------------------|----------------------------|
| `mail.wizard.invite`        | `mail.followers.edit`      |
| `mail.resend.message`       | REMOVED                    |
| `mail.resend.partner`       | REMOVED                    |

#### ✨ NEW: `ir.cron` and `ir.filters` views added to mail

- `views/ir_cron_views.xml`
- `views/ir_filters_views.xml`

#### ✨ NEW: `mail.presence` model

```python
_name = 'mail.presence'
# Replaces bus.presence for mail-specific data
```

### `bus` module

#### ⚠️ BREAKING: `bus.presence` model REMOVED

```python
# 18.0 — bus had its own presence model
class BusPresence(models.Model):
    _name = 'bus.presence'
    user_id, last_poll, last_presence, status ...

# 19.0 — moved to mail.presence (see mail module)
```

#### ⚠️ BREAKING: `im_status` field removed from `res.users` and `res.partner` in bus

```python
# 18.0 (bus module)
im_status = fields.Char('IM Status', compute='_compute_im_status')

# 19.0 — removed from bus, managed by mail module
```

#### ✨ NEW: `bus.bus` class renamed from `ImBus`

```python
# 18.0
class ImBus(models.Model):
    _name = 'bus.bus'

# 19.0
class BusBus(models.Model):
    _name = 'bus.bus'
```

---

## Website & Portal

### `website` module

#### ⚠️ BREAKING: `web_editor` dependency replaced

```python
# 18.0
'depends': [..., 'web_editor', ...]

# 19.0
'depends': [..., 'html_builder', ...]
```

#### ✨ NEW: `website.assets` model (replaces `web_editor.assets`)

```python
class WebsiteAssets(models.AbstractModel):
    _name = 'website.assets'
    # replaces _inherit = 'web_editor.assets'
```

#### ✨ NEW: `website.html.text.processor` model for AI-assisted content

#### ⚠️ BREAKING: `website._get_web_editor_context()` renamed

```python
# 18.0
def _get_web_editor_context(cls):

# 19.0
def _get_editor_context(cls):
```

#### ✨ NEW: New snippets in 19.0

- `s_announcement_scroll`, `s_bento_block`, `s_bento_grid`
- `s_carousel_cards`, `s_comparisons_horizontal`
- `s_company_team_spotlight`, `s_company_team_grid`, `s_company_team_card`
- `s_tabs_images`, `s_quotes_carousel_compact`
- `s_numbers_boxed`, `s_numbers_framed`, `s_floating_blocks`
- `s_timeline_images`, `s_website_form_info`, `s_bento_banner`
- `s_split_intro`, `s_attributes_horizontal`, `s_attributes_vertical`
- `s_banner_product`, `s_website_form_overlay`, `s_inline_text`

#### ⚠️ BREAKING: `apt` dependency added

```python
# 19.0 manifest
'external_dependencies': {
    'apt': {'geoip2': 'python3-geoip2'},
}
```

### `portal` module

#### ⚠️ BREAKING: `web_editor` dependency replaced by `html_editor`

```python
# 18.0
'depends': ['web', 'web_editor', 'http_routing', 'mail', 'auth_signup']

# 19.0
'depends': ['web', 'html_editor', 'http_routing', 'mail', 'auth_signup']
```

#### ⚠️ BREAKING: `portal.mixin._notify_get_recipients_groups()` signature

```python
# 18.0
def _notify_get_recipients_groups(self, message, model_description, msg_vals=None):

# 19.0
def _notify_get_recipients_groups(self, message, model_description, msg_vals=False):
```

#### ⚠️ BREAKING: `portal.mixin._get_thread_with_access()` signature

```python
# 18.0
def _get_thread_with_access(self, thread_id, mode="read", **kwargs):

# 19.0
def _get_allowed_access_params(self):  # new helper
def _get_thread_with_access(self, thread_id, *, hash=None, pid=None, token=None, **kwargs):
```

#### ✨ NEW: `portal.mixin` methods

```python
def _get_frontend_writable_fields(self):
def _can_edit_country(self):
def _can_be_edited_by_current_customer(self, **kwargs):
def _get_current_partner(self, **kwargs):
def _get_delivery_address_domain(self):
```

---

## Authentication

### `auth_totp` module

#### ✨ NEW: `auth.totp.rate.limit.log` model

```python
class AuthTotpRateLimitLog(models.TransientModel):
    _name = 'auth.totp.rate.limit.log'
    user_id, ip, limit_type
```

#### ✨ NEW: `res.users.totp_last_counter` field

```python
totp_last_counter = fields.Integer(copy=False, groups=fields.NO_ACCESS)
```

#### ✨ NEW: Rate limiting methods

```python
def _totp_rate_limit(self, limit_type):
def _totp_rate_limit_purge(self, limit_type):
```

---

## UoM (Unit of Measure)

### ⚠️ BREAKING: Major restructuring of `uom.uom`

The entire UoM model was redesigned from a flat structure with `uom_type` to a hierarchical tree structure.

#### ⚠️ BREAKING: `uom.uom` field changes

| Old Field        | New Field / Change                                      |
|------------------|---------------------------------------------------------|
| `name`           | Label changed to `'Unit Name'`                          |
| `factor`         | Now computed recursively from `relative_factor`         |
| `factor_inv`     | REMOVED                                                 |
| `uom_type`       | REMOVED (replaced by tree structure)                    |
| `ratio`          | REMOVED                                                 |
| `color`          | REMOVED                                                 |
| —                | `relative_factor` NEW — ratio relative to parent        |
| —                | `relative_uom_id` NEW — parent UoM                      |
| —                | `related_uom_ids` NEW — child UoMs                      |
| —                | `parent_path` NEW — for hierarchical queries            |
| —                | `sequence` NEW — computed display order                 |

#### ⚠️ BREAKING: `uom.uom._compute_quantity()` signature

```python
# 18.0
def _compute_quantity(self, qty, to_unit, round=True, rounding_method='UP',
                      raise_if_failure=True):

# 19.0 — type-annotated, rounding_method default changed
def _compute_quantity(self, qty, to_unit, round=True,
                      rounding_method: RoundingMethod = 'HALF-UP', ...):
```

#### ✨ NEW: `uom.uom` convenience methods

```python
def round(self, value: float, rounding_method: RoundingMethod = 'HALF-UP') -> float:
def compare(self, value1: float, value2: float) -> Literal[-1, 0, 1]:
def is_zero(self, value: float) -> bool:
def _has_common_reference(self, other_uom: Self) -> bool:
```

#### ⚠️ BREAKING: `uom.category` model — `_onchange_uom_ids` removed

#### ✨ NEW: `uom` module assets

```python
# 19.0 manifest
'assets': {
    'web.assets_backend': ['uom/static/src/components/**/*'],
}
```

---

## Utilities

### `base_automation` module

#### ✨ NEW: `base.automation` now inherits `mail.thread` and `mail.activity.mixin`

```python
_inherit = ['mail.thread', 'mail.activity.mixin']
```

#### ✨ NEW: `base.automation` fields

| Field                | Description                              |
|----------------------|------------------------------------------|
| `trg_date_range_mode`| Selection for date range mode            |
| `previous_domain`    | Store=False for domain change detection  |

#### ✨ NEW: `base.automation` cron-linked method

```python
def _search_time_based_automation_records(self, *, until):
def _cron_process_time_based_actions(self):
```

#### ⚠️ BREAKING: `base.automation` action server class renamed

```python
# 18.0
class ServerAction(models.Model):
    _inherit = "ir.actions.server"

# 19.0
class IrActionsServer(models.Model):
    _inherit = "ir.actions.server"
```

### `analytic` module

#### ⚠️ BREAKING: `analytic.plan.fields` renamed

```python
# 18.0
class AnalyticPlanFields(models.AbstractModel):
    _name = 'analytic.plan.fields.mixin'

# 19.0
class AnalyticPlanFieldsMixin(models.AbstractModel):
    _name = 'analytic.plan.fields.mixin'
```

#### ✨ NEW: `analytic.account.line` fiscal year search

```python
fiscal_year_search = fields.Boolean(...)
def _search_fiscal_date(self, operator, value):
```

#### ⚠️ BREAKING: `_read_group_groupby()` signature

```python
# 18.0
def _read_group_groupby(self, groupby_spec: str, query: Query) -> SQL:

# 19.0
def _read_group_groupby(self, alias: str, groupby_spec: str, query: Query) -> SQL:
```

### `digest` module

#### ⚠️ BREAKING: Class renamed

```python
# 18.0
class Digest(models.Model):

# 19.0
class DigestDigest(models.Model):
```

### `spreadsheet` module

#### ⚠️ BREAKING: Asset bundle dependency changed

```python
# 18.0
('include', 'spreadsheet.dependencies')

# 19.0
('include', 'web.chartjs_lib')
```

#### ✨ NEW: Chart.js extensions

- `chartjs-chart-geo` (geo charts)
- `chart_js_treemap` (treemap charts)

#### ⚠️ BREAKING: Category changed

```python
# 18.0
'category': 'Hidden'

# 19.0
'category': 'Productivity/Dashboard'
```

---

## Security Model Changes

### ⚠️ BREAKING: `res.groups` field rename

```python
# 18.0
<field name="users" eval="[(4, ref('base.user_root'))]"/>

# 19.0
<field name="user_ids" eval="[(4, ref('base.user_root'))]"/>
```

### ✨ NEW: `res.groups.privilege` model

```python
# 19.0 — new model for group privilege descriptions
<record model="res.groups.privilege" id="res_groups_privilege_accounting">
    <field name="name">Accounting</field>
    <field name="category_id" ref="base.module_category_accounting"/>
    <field name="sequence">20</field>
    <field name="comment">Invoices, payments and basic invoice reporting.</field>
</record>
```

### ⚠️ BREAKING: Module category changes

Many `ir.module.category` records replaced by `res.groups.privilege` for:
- `base.module_category_accounting_accounting`
- `base.module_category_human_resources_employees`
- `base.module_category_services_project`

### ⚠️ BREAKING: `crm` security rule — `groups_id` → `group_ids`

```xml
<!-- 18.0 -->
<field name="groups_id" eval="[(4, ref('sales_team.group_sale_manager'))]"/>

<!-- 19.0 -->
<field name="group_ids" eval="[(4, ref('sales_team.group_sale_manager'))]"/>
```

### ✨ NEW: Access rights added

| Model                        | Notes                           |
|------------------------------|---------------------------------|
| `discuss.call.history`       | user/public/portal read access  |
| `res.role`                   | user read, admin write access   |
| `mail.presence`              | system-level access             |
| `mail.activity.schedule.line`| user access                     |
| `stock.package`              | replaces `stock.quant.package`  |
| `stock.package_history`      | user access                     |
| `stock.put.in.pack`          | user access                     |
| `stock.reference`            | all users read                  |

### ⚠️ BREAKING: `mail.wizard.invite` replaced

```python
# 18.0
access_mail_wizard_invite, model_mail_wizard_invite

# 19.0
access_mail_followers_edit, model_mail_followers_edit
```

### ⚠️ BREAKING: `mail.resend.message` and `mail.resend.partner` REMOVED

---

## JavaScript / OWL Front-End Changes

### ⚠️ BREAKING: Legacy test framework fully removed

All `web.qunit_suite_tests` assets removed across all modules. Replaced by `web.assets_unit_tests`.

```python
# 18.0
'web.qunit_suite_tests': ['module/static/tests/**/*.js']

# 19.0
'web.assets_unit_tests': ['module/static/tests/**/*.test.js']
```

### ⚠️ BREAKING: Portal JS migrated to interactions framework

```python
# 18.0
'web.assets_frontend': [
    'portal/static/src/js/portal.js',
    'portal/static/src/js/portal_sidebar.js',
    'portal/static/src/js/portal_composer.js',
    'portal/static/src/js/portal_security.js',
]

# 19.0
'web.assets_frontend': [
    'portal/static/src/interactions/**/*',
]
```

Similar changes in `sale`, `account`, `purchase` modules.

### ⚠️ BREAKING: `web_editor` assets moved

Code that referenced `web_editor` assets must now reference either `html_editor` or `html_builder`.

### ✨ NEW: `web` module additions

| Asset / Feature         | Description                               |
|-------------------------|-------------------------------------------|
| `web/security/web_security.xml` | NEW security declarations         |
| `views/memory_template.xml`     | Memory inspection template         |
| `views/speedscope_config_wizard.xml` | Performance profiling tool    |
| `views/ir_ui_view_views.xml`    | UI view management                 |
| `res.users.settings.embedded.action` | Embedded action config model |

### ✨ NEW: Dark mode support in project/hr_holidays

```python
'web.assets_web_dark': [
    'project/static/src/**/*.dark.scss',
]
```

---

## Summary of Breaking Changes

### Critical (will cause startup failures)

1. **Module removals**: `hr_contract`, `web_editor`, `account_edi_ubl_cii_tax_extension`, `auth_totp_mail_enforce`
2. **`bus.presence` model removed** — code referencing `self.env['bus.presence']` will fail
3. **`product.packaging` model removed** — code using `self.env['product.packaging']` will fail
4. **`uom.uom.uom_type` field removed** — code filtering by `uom_type` will fail
5. **`uom.uom.factor_inv` removed** — code using `factor_inv` will fail
6. **`sale.order.option` model removed** — code using `self.env['sale.order.option']` will fail
7. **`stock.change.product.qty` wizard removed**
8. **`stock.quant.package` → `stock.package`** — access rights and references must be updated

### High Impact (will cause runtime errors)

9. **`osv.expression` → `odoo.fields.Domain`** — any custom code using `from odoo.osv import expression`
10. **`name_search(args=...)` → `name_search(domain=...)`** — positional calls may break
11. **`res.groups.users` → `res.groups.user_ids`** — XML and Python references
12. **`purchase.order.notes` → `purchase.order.note`**
13. **`account.account.deprecated` → `account.account.active`**
14. **`account.tax.tag.tax_negate` → `account.tax.tag.balance_negate`**
15. **`payment._handle_notification_data` → `payment._process`**
16. **`_compute_reference_prefix(provider_code, ...)` → `_compute_reference_prefix(...)` (no provider_code)**

### Medium Impact (method signature changes)

17. **`mail.thread` API changes** — `msg_vals=None` → `msg_vals=False`, `_get_allowed_message_post_params` renamed
18. **`hr.employee.base` removed** — modules inheriting it must adapt
19. **`stock_account` SVL fields changed** — `value_svl`, `quantity_svl` removed
20. **`stock.move.line` packaging fields** — `product_packaging_id/qty` → `packaging_uom_id/qty`
21. **`crm.lead._handle_won_lost()` signature** completely changed
22. **`uom._compute_quantity()` rounding default** changed from 'UP' to 'HALF-UP'
