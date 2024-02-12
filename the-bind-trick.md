# implicit bindings and the bind trick

Let us create a button in kivy that informs us whether it has been pressed an even or odd number of times.

```python
from kivy.app import App
from kivy.lang import Builder

root_widget = Builder.load_string(f"""
AnchorLayout:
    num_presses: 0
    Button:
        size_hint: None, None
        size: 400, 50
        text: "pressed an even number of times" if root.num_presses % 2 == 0 else "pressed an odd number of times"
        on_release: root.num_presses += 1
""")

class TestApp(App):
    def build(self):
        return root_widget

if __name__ == "__main__":
    TestApp().run()
```

For this to work, realize that Kivy creates an *implicit binding* to the `num_presses` Kivy property so that whenever `num_presses` changes, the `text` property is recalculated. The value of implicit bindings in that it makes code concise and simpler to write. But the downside is that it is easy to forget that the implicit binding exists which can lead to mistakes when you refactor widgets written in kvlang.

Suppose that you want the button to display "Press me!" when it hasn't been pressed yet. In kvlang, this would require a nested ternary operator:

```python
from kivy.app import App
from kivy.lang import Builder

root_widget = Builder.load_string(f"""
AnchorLayout:
    num_presses: 0
    Button:
        size_hint: None, None
        size: 400, 50
        text: "Press me!" if root.num_presses == 0 else "pressed an even number of times" if root.num_presses % 2 == 0 else "pressed an odd number of times"
        on_release: root.num_presses += 1
""")


class TestApp(App):
    def build(self):
        return root_widget


if __name__ == "__main__":
    TestApp().run()
```

This is difficult ro read and will be difficult to refactor further. Therefore, it will make sense to extract the logic for determining the button text into Python. We might try to refactor the widget in the following incorrect way:

```python
from kivy.app import App
from kivy.lang import Builder
from kivy.properties import NumericProperty
from kivy.uix.anchorlayout import AnchorLayout

Builder.load_string(f"""
<RootWidget>:
    num_presses: 0
    Button:
        size_hint: None, None
        size: 400, 50
        text: root.get_button_text()
        on_release: root.num_presses += 1
""")

class RootWidget(AnchorLayout):
    num_presses = NumericProperty(0)

    def get_button_text(self):
        if self.num_presses == 0:
            return "Press me!"
        elif self.num_presses % 2 == 0:
            return "pressed an even number of times"
        else:
            return "pressed an odd number of times"

class TestApp(App):
    def build(self):
        return RootWidget()

if __name__ == "__main__":
    TestApp().run()
```

But this no longer works. The button always says "pressed an even number of times". The reason this no longer works is that there is no bindings to the Button widget's `text` property. `root.get_button_text()` is performed once to intialize the text value and then is never called again.  

There are many ways you can fix this mistake. The first is to provide a reference in the RootWidget class to the button widget and then explicitly call `get_button_text` whenever `num_presses` changes:

```python
from kivy.app import App
from kivy.lang import Builder
from kivy.properties import NumericProperty
from kivy.properties import ObjectProperty
from kivy.uix.anchorlayout import AnchorLayout

Builder.load_string(f"""
<RootWidget>:
    btn: btn
    num_presses: 0
    Button:
        id: btn
        size_hint: None, None
        size: 400, 50
        text: root.get_button_text()
        on_release: root.num_presses += 1
""")

class RootWidget(AnchorLayout):

    btn = ObjectProperty(None)
    num_presses = NumericProperty(0)

    def on_num_presses(self, *args):
        self.btn.text = self.get_button_text()

    def get_button_text(self):
        if self.num_presses == 0:
            return "Press me!"
        elif self.num_presses % 2 == 0:
            return "pressed an even number of times"
        else:
            return "pressed an odd number of times"

class TestApp(App):
    def build(self):
        return RootWidget()
```

A second is to *reintroduce* the implicit binding by making num_presses an argument in get_button_text:

```python
from kivy.app import App
from kivy.lang import Builder
from kivy.properties import NumericProperty
from kivy.uix.anchorlayout import AnchorLayout

Builder.load_string(f"""
<RootWidget>:
    num_presses: 0
    Button:
        size_hint: None, None
        size: 400, 50
        text: root.get_button_text(root.num_presses)
        on_release: root.num_presses += 1
""")

class RootWidget(AnchorLayout):
    num_presses = NumericProperty(0)

    def get_button_text(self, num_presses):
        if num_presses == 0:
            return "Press me!"
        elif num_presses % 2 == 0:
            return "pressed an even number of times"
        else:
            return "pressed an odd number of times"

class TestApp(App):
    def build(self):
        return RootWidget()

if __name__ == "__main__":
    TestApp().run()
```

Personally, I prefer the former solution. While the second approach is simpler, additional code is worth it if the bindings are explicit.

### An evil hack

There is a third approach which I call the "bind trick". The bind trick exploits the way implicit bindings are created in kvlang to make them explicit.

```python
from kivy.app import App
from kivy.lang import Builder
from kivy.properties import NumericProperty
from kivy.uix.anchorlayout import AnchorLayout

Builder.load_string(f"""
<RootWidget>:
    num_presses: 0
    Button:
        size_hint: None, None
        size: 400, 50
        text: root.get_button_text(bind=[root.num_presses])
        on_release: root.num_presses += 1
""")

class RootWidget(AnchorLayout):

    num_presses = NumericProperty(0)

    def get_button_text(self, *, bind=None):
        if self.num_presses == 0:
            return "Press me!"
        elif self.num_presses % 2 == 0:
            return "pressed an even number of times"
        else:
            return "pressed an odd number of times"

class TestApp(App):
    def build(self):
        return RootWidget()

if __name__ == "__main__":
    TestApp().run()
```

The bind keyword in `RootWidget::get_button_text` is not actually used by the method, but we pass a list of Kivy properties to `bind` in kvlang anyway. Because of how implicit bindings are implemented in Kivy, this automatically binds the expression assigned to the Button `text` property so that it recalculates whenever any Kivy property assigned to `bind` changes. While this solution is likely an unintended exploit of kvlang, the approach is concise without sacrificing explicitness.

One issue with the "bind trick" is that the method signature has an argument that has no usage in the method. Not only will this cause most modern IDE's to raise a warning to the user, it will also make it awkward to properly document the method. So, if you're going to use the bind trick, an evil hack is to create a decorator like the following:

```python
from functools import wraps

from kivy.app import App
from kivy.lang import Builder
from kivy.properties import NumericProperty
from kivy.uix.anchorlayout import AnchorLayout

def kvbind(function):
    """A function decorator which effectively stitches the keyword "bind" onto the function.
    This can be used to create explicit bindings to Kivy properties in kvlang."""
    @wraps(function)
    def wrapper(*args, **kwargs):
        if "bind" in kwargs:
            kwargs.pop("bind")
        return function(*args, **kwargs)
    return wrapper

Builder.load_string(f"""
<RootWidget>:
    num_presses: 0
    Button:
        size_hint: None, None
        size: 400, 50
        text: root.get_button_text(bind=[root.num_presses])
        on_release: root.num_presses += 1
""")

class RootWidget(AnchorLayout):
    num_presses = NumericProperty(0)

    @kvbind
    def get_button_text(self):
        if self.num_presses == 0:
            return "Press me!"
        elif self.num_presses % 2 == 0:
            return "pressed an even number of times"
        else:
            return "pressed an odd number of times"

class TestApp(App):
    def build(self):
        return RootWidget()

if __name__ == "__main__":
    TestApp().run()
```

If the bind trick seems too "hacky" to you, there is no need to use it. The other two techniques described here are fully adequate.
