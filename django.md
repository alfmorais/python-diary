# save_formset

~~~python
    def save_formset(self, request, form, formset, change):
        for inline_form in formset.forms:
            if formset.model == PosOrderReturn:
                if inline_form.has_changed():
                    inline_form.instance.requester = request.user
        super().save_formset(request, form, formset, change)
~~~

# formfield_for_foreignkey

~~~python
    def formfield_for_foreignkey(self, db_field, request, **kwargs):
        if db_field.name == "order_item":
            id = int(resolve(request.path).kwargs["object_id"])
            kwargs["queryset"] = PosOrder.objects.get(id=id).get_items()
        return super().formfield_for_foreignkey(db_field, request, **kwargs)
~~~

# CreateOnlyAdminMixin

~~~python
class CreateOnlyAdminMixin(object):
    extra = 0
    max_num = 0
    can_delete = False

    def has_delete_permission(self, request, obj=None):
        return False

    def has_change_permission(self, request, obj=None):
        return False
~~~

# admin.TabularInline

~~~python
class PosOrderItemReturnInline(CreateOnlyAdminMixin, admin.TabularInline):
    model = PosOrderReturn
    extra = 0
    max_num = None
    fields = (
        "order_item",
        "quantity",
        "reason",
        "msg",
        "requester",
    )
    readonly_fields = ("requester",)
~~~

# def clean - models

~~~python
    def clean(self):
        if self.order_item.qty < self.quantity:
            raise ValidationError(
                "Valor solicitado não pode ser maior que o saldo residual do pedido"
            )
~~~

# def save - models

~~~python
    @transaction.atomic
    def save(self, *args, **kwargs):
        if self._state.adding:
            decrement = self.quantity
            self.order_item.decrement_qty(decrement)
        super().save(*args, **kwargs)
~~~

# id

~~~python
id = int(resolve(request.path).kwargs["object_id"])
~~~
