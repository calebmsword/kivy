
# The Root Widget is not the actual root

Widgets are stored in the widget tree. The root of the widget tree is the widget returned by the `build` method of the Kivy app. For example, given:

```python
from kivy.app import App
from kivy.uix.button import Button


class TestApp(App):

    def build(self):
        return Button(text="Press me!")


if __name__ == '__main__':
    TestApp().run()
```

The `Button` instance will be the root widget. (It is also worth mentioning that the root widget can be determined from a kv file. See the first bullet point under [here](https://kivy.org/doc/stable-2.0.0/guide/lang.html#how-to-load-kv).)

Widgets have a `parent` attribute which points to that widget's parent. From what we know about trees, the root of the tree has no parent. So we would expect the root widget's `parent` attribute to be `None`. Let's test this. Try running the following code:

```python
from kivy.app import App
from kivy.uix.button import Button


class TestApp(App):

    def build(self):
        return Button(text="Press me!")

    def on_start(self):
        print(self.root.parent)  # the App instance has a "root" attribute which points to the root widget


if __name__ == '__main__':
    TestApp().run()
```

The result is surprising:

![image](https://github.com/calebmsword/kivy/assets/85499281/f82d8537-203b-4efd-886f-0d4ec278fd5b)

Evidently, the root widget has a parent, and it so happens to be the [Window object](https://kivy.org/doc/stable/api-kivy.core.window.html). If you poke around the documentation for the `Window` object, you'll find that it has a method called `add_widget`. Wait a minute... can we add a second "root widget" to a running app?

Let's try it:

```python
from kivy.app import App
from kivy.core.window import Window
from kivy.uix.button import Button


class TestApp(App):

    def build(self):
        return Button(text="Press me!")

    def on_start(self):
        button2 = Button(text="button 2", size_hint=(0.1, 0.1))
        Window.add_widget(button2)


if __name__ == '__main__':
    TestApp().run()
```

The result:

<img src="https://github.com/calebmsword/kivy/assets/85499281/3ed90734-6fd3-4bb3-ac33-b5cd8e3bcb7b" width="600px" />

It ends up that the Window object _can_ have multiple children. (Seems like the name "root widget" is a misnomer!)

So, all of this begs the question: _why_ is Kivy architectured this way? The reason is that the most recently added child of `Window` **is drawn on top of every other widget in the application**. The standard use case for this is when you want to display a popup window. In fact, the [ModalView](https://kivy.org/doc/stable-2.0.0/api-kivy.uix.popup.html) and [PopUp](https://kivy.org/doc/stable-2.0.0/api-kivy.uix.popup.html) widgets are implemented in this way.

The `Window` also has a `remove_widget` method that you can use to remove the widget from view.

Unlike the `add_widget` method available to the `Widget` class, `Window::add_widget` does not have an optional index argument, meaning that `Window` will always draw the most recently added Widget on top of everything else.

My tips for those who start to use `Window.add_widget` in their apps: 
 - Try your best to have no more than two widgets as children of `Window` (i.e., the root widget and some popup widget). You don't want to run into a situation where multiple widgets are children of `Window` and then you struggle to make one widget display over another.
 - Always use `Window.remove_widget` whenever you want the widget to no longer displayed. Don't, for example, set a popup window's opacity to zero when the user exits the popup. (This will help you follow the previous rule).
