Widgets
=======
.. module:: wtforms.widgets

Widgets are classes whose purpose are to render a field to its usable
representation, usually XHTML.  When a field is called, the default behaviour
is to delegate the rendering to its widget. This abstraction is provided so
that widgets can easily be created to customize the rendering of existing
fields.

**Note** All built-in widgets will return upon rendering a "HTML-safe" unicode
string subclass that many templating frameworks (Jinja, Mako, Genshi) will
recognize as not needing to be auto-escaped.

Built-in widgets
----------------

.. autoclass:: ListWidget
.. autoclass:: TableWidget
.. autoclass:: Input
.. autoclass:: TextInput()
.. autoclass:: PasswordInput
.. autoclass:: HiddenInput()
.. autoclass:: CheckboxInput()
.. autoclass:: FileInput()
.. autoclass:: SubmitInput()
.. autoclass:: TextArea
.. autoclass:: Select

Widget-Building Utilities
-------------------------

These utilities are used in WTForms widgets to help render HTML and also in
order to work along with HTML templating frameworks. They can be imported for
use in building custom widgets as well.

.. autofunction:: html_params

WTForms uses `MarkupSafe`_ to escape unsafe HTML characters before
rendering. You can mark a string using :class:`markupsafe.Markup` to
indicate that it should not be escaped.

.. _MarkupSafe: https://markupsafe.palletsprojects.com/


Custom widgets
--------------

Widgets, much like validators, provide a simple callable contract. Widgets can
take customization arguments through a constructor if needed as well. When
the field is called or printed, it will call the widget with itself as the
first argument and then any additional arguments passed to its caller as
keywords.  Passing the field is done so that one instance of a widget might be
used across many field instances.

Let's look at a widget which renders a text field with an additional class if
there are errors::

    class MyTextInput(TextInput):
        def __init__(self, error_class='has_errors'):
            super(MyTextInput, self).__init__()
            self.error_class = error_class

        def __call__(self, field, **kwargs):
            if field.errors:
                c = kwargs.pop('class', '') or kwargs.pop('class_', '')
                kwargs['class'] = '%s %s' % (self.error_class, c)
            return super(MyTextInput, self).__call__(field, **kwargs)

In the above example, we extended the behavior of the existing
:class:`TextInput` widget to append a CSS class as needed. However, widgets
need not extend from an existing widget, and indeed don't even have to be a
class.  For example, here is a widget that renders a
:class:`~wtforms.fields.SelectMultipleField` as a collection of checkboxes::

    def select_multi_checkbox(field, ul_class='', **kwargs):
        kwargs.setdefault('type', 'checkbox')
        field_id = kwargs.pop('id', field.id)
        html = ['<ul %s>' % html_params(id=field_id, class_=ul_class)]
        for value, label, checked in field.iter_choices():
            choice_id = '%s-%s' % (field_id, value)
            options = dict(kwargs, name=field.name, value=value, id=choice_id)
            if checked:
                options['checked'] = 'checked'
            html.append('<li><input %s /> ' % html_params(**options))
            html.append('<label for="%s">%s</label></li>' % (field_id, label))
        html.append('</ul>')
        return ''.join(html)
