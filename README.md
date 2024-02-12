# Kivy notes
These notes have two goals. The first is to provide documentation about odds and ends of Kivy but the official documentation does not describe in a satisfactory manner. The second is to describe tips and tricks that I have found useful in my own experience with Kivy. I hope this guide will prove to be useful to others.

Note that is a resource for **intermediate** Kivy developers. If you are new to Kivy, then I recommend consulting the following resources before reading ahead:
 - The [Kivy crash course](https://www.youtube.com/watch?v=F7UKmK9eQLY&list=PLdNh1e1kmiPP4YApJm8ENK2yMlwF1_edq), a brief series of videos which are a fantastic way to get your feet wet with the framework.
 - The official [Kivy programming guide](https://kivy.org/doc/stable/guide/basic.html) written by the Kivy developers themselves.
 - The following sections of the official Kivy API documentation:
   - The [Kivy language documention](https://kivy.org/doc/stable/api-kivy.lang.html).
   - The [Kivy properties documentation](https://kivy.org/doc/stable/api-kivy.properties.html).
   - The [coordinate system section](https://kivy.org/doc/stable/api-kivy.uix.relativelayout.html#coordinate-systems) from the RelativeLayout documentation. This is the only official description of Kivy's coordinate systems.
   - The [Kivy clock documentation](https://kivy.org/doc/stable/api-kivy.clock.html).

To be written:
 - `size_hint_min` and `size_hint_max`
 - The canvas tree
 - Things I wish I knew about canvases when I first started using Kivy
 - When to use RelativeLayouts (and why it's usually bad practice)
 - Sharing data between widgets
 - style.kv
 - RecycleView
 - How the event loop works
