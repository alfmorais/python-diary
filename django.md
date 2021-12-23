# save_formset

~~~python
    def save_formset(self, request, form, formset, change):
        for inline_form in formset.forms:
            if formset.model == PosOrderReturn:
                if inline_form.has_changed():
                    inline_form.instance.requester = request.user
        super().save_formset(request, form, formset, change)
~~~

Essa função tem como objetivo verificar quais os inline possui o model PosOrderReturn e verufuca se houve uma mudança. Caso ambas for verdadeiro, pega o campo do model que possui o nome `requester` e altera para o usuário logado. 

[Documentação do Django](https://docs.djangoproject.com/en/4.0/ref/contrib/admin/)

# formfield_for_foreignkey

~~~python
    def formfield_for_foreignkey(self, db_field, request, **kwargs):
        if db_field.name == "order_item":
            id = int(resolve(request.path).kwargs["object_id"])
            kwargs["queryset"] = PosOrder.objects.get(id=id).get_items()
        return super().formfield_for_foreignkey(db_field, request, **kwargs)
~~~

A função `formfield_for_foreignkey` tem como objetivo fazer um filtro nos campos inseridos no admin.py que são `foreignkey`. No exemplo acima a função está verificando se o campo ordem_item está inserido no admin e após a confirmação, pega o ID da página através da função `resolve` e faz um filtro para mostrar um dropdown somente os pedidos referente aquele ID.

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
CreateOnlyAdminMixin é uma classe modificada do Django pelo o time da MaisTODOS, que tem como objetivo ter uma permissão de adicionar um item novo, não alterar e somente visualizações. 

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

No admin.py a classe `Inline` recebe como argumento o mixin CreateOnlyAdminMixin para ter as funçãos de adicionar, não alterar e somente visualização.

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
