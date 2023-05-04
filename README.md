The `pos` attribute of a Kivy widget refers to the (`x`, `y`) position of the bottom-left corner of that widget. Typically, the origin of this coordinate system is the bottom-left corner of the containing window. 

Coordinates in this coordinate system are said to be in window coordinates. 

However, there are four “special” widgets which change the origin of the coordinate system for their children. These widgets are 

 - RelativeLayout 
 - ScrollView 
 - Scatter 
 - ScatterLayout 

To discuss this in detail, let us introduce the term “parent stack.” For example, if the root kv file of a Kivy application looked like the following: 

```kvlang
BoxLayout: 
    BoxLayout: 
        AnchorLayout: 
            BoxLayout: 
                Widget: 
```
 

Then `Widget`’s parent stack would be `BoxLayout`, `AnchorLayout`, `BoxLayout`, `BoxLayout`. `AnchorLayout`’s parent stack would be `BoxLayout`, `BoxLayout`. Note that the parent stack is ordered. 

Now, if the parent stack of a widget (let’s call it `widget_a`) includes one of the four special widgets, then `widget_a` adopts a new coordinate system. This new coordinate system has an origin located at the bottom-left corner of the first “special” widget in `widget_a`’s parent stack. 

Now we introduce the term parent coordinates. The parent coordinates of `widget_a` depend on whether `widget_a` has a special widget in its parent stack.  

If widget_a does not have a special widget in its parent stack: 

Then widget_a’s parent coordinates are equivalent to window coordinates. 

If widget_a has a special widget in its parent stack: 

Then the origin of the parent coordinates for widget_a is located at the bottom-left corner of the first special widget in widget_a’s parent stack. 

Kivy uses one other coordinate system which is referred by two names: widget coordinates / local coordinates. The widget/local coordinates of widget_a are calculated in the following way: 

If widget_a is a special Widget: 

Then the widget/local coordinates for widget_a are relative to the bottom-left corner of widget a. 

If widget_a is not a special Widget: 

Then the widget/local coordinates for widget_a are equivalent to its parent coordinates. 

It is crucial to understand that the pos attribute of a widget is in the parent coordinates of that widget. 

 

The Kivy docs do not mention this, but there is another coordinate system that they use. We will call it relative coordinates. The origin of the relative coordinates of widget_a is located at the bottom left of widget_a. 

(If only the other coordinate systems were so easy to understand.) 

Window coordinates are an absolute coordinate system. The position (x, y) in window coordinates is the same point on the screen for every widget in a running application. However, parent coordinates, widget/local coordinates, and relative coordinates are always in reference to a particular widget. In other words, each widget has its own parent, widget/local, and relative coordinate system that may have a different origin from that of another widget. 

 

--- 

 

It is interesting to notice that, if there are no special widgets in the application, then window coordinates, widget/local coordinates, and parent coordinates for all widgets have the same origin. 

 

It is also interesting to notice that the widget/local coordinates of the direct parent of widget_a are the same as the parent coordinates of widget_a. That is, given 

BoxLayout: 

	Widget: 

		id: widget_parent 

		Widget: 

			id: widget_child 

the widget/local coordinate system of the widget with id “widget_parent” has the same origin as the parent coordinate system of “widget_child”. 

 

It is also interesting to notice that, if widget_a is a special widget, then its relative coordinates and widget/local coordinates are the same. 

 

--- 

 

Every widget has an API for converting positions to different coordinate systems. 

 

to_local(x, y, relative=False) 

If relative is set to False (the default value): 

x, y are assumed to be in the parent coordinates of the widget which calls the method. It returns a tuple that converts this position into widget/local coordinates of the widget which calls the method. 
 
For example, widgetA.to_local(*widgetA.pos) converts WidgetA’s position into the local coordinates of widgetA. (Remember that the pos attribute is always in parent coordinates). 

If relative is set to True: 

x, y are assumed to be in the parent coordinates of the widget which calls the method. Then the returned tuple will be in relative coordinates of the widget which calls the method. 

 

to_parent(x, y, relative=False) 

If relative is set to False (the default value): 

x, y are assumed to be in the local coordinates of the widget which calls the method. It returns a tuple that is this position in the parent coordinates of the widget which calls the method. 

If relative is set to True: 

x, y are assumed to be relative coordinates of the widget which calls the method. It returns a tuple that is this position in the parent coordinates of the widget which calls the method. 

 

to_widget(x, y, relative=False) 

If relative is set to False (the default value): 

x, y are assumed to be in window coordinates. It returns a tuple that is this position in the widget/local coordinates of the widget which calls the method. 

If relative is set to True: 

x, y are assumed to be in window coordinates. It returns a tuple that is this position in the relative coordinates of the widget which calls the method. 

 

to_window(x, y, initial=True, relative=False) 

If initial is set to True (the default value): 

x, y are assumed to be in parent coordinates of the widget which calls the method. It returns a tuple that is this position in window coordinates. 

If initial is set to False: 

If relative is set to False (the default value): 

x, y are assumed to be in widget/local coordinates of the widget which calls the method. It returns a tuple that is this position in window coordinates. 

If relative is set to True: 

x, y are assumed to be in the relative coordinates of the widget which calls the method. It returns a tuple that is this position in window coordinates. 

What are we to make of the case where initial is True and relative is True? 

If we inspect Kivy’s source code, widget_a.to_window(x, y, initial=True, relative=True) is equivalent to calling widget_a.parent.to_window(x, y, relative=True) 

Then x, y are assumed to be in the relative coordinates of the direct parent of the widget which calls the method. It returns a tuple that is this position in window coordinates. 

Never do this. If you ever need this very specific conversion, just use widget_a.parent.to_window(x, y, relative=True), which is more declarative. 

 

--- 

 

It is a code smell if one widget directly accesses the position of another widget. For example, the following code attempts to place widget_b directly next to widget_a. 

 

BoxLayout: 

	RelativeLayout: 

Widget: 

		id: widget_a 

	Widget: 

		id: widget_b 

		pos: widget_a.x + widget_a.width, widget_a.y 

 

However, the pos attribute (and the x and y attributes) of widget_a and widget_b are in different coordinate systems because widget_a is in a RelativeLayout (one of the “special” widgets) while widget_b is not. This will not place widget_b in the expected location. 

It is safest to always use the following code snippet when one widget accesses the position of another widget. 

def convert_pos(*, input_widget, output_widget):  

window_coords = input_widget.to_window(*input_widget.pos) 

 

# the widget/local coordinates of the parent of output_widget are the same as the parent 		# coordinates of output_widget. 

return output_widget.parent.to_widget(*window_coords) 

 

For example, convert_pos(input_widget=widget_a, output_widget=widget_b) returns a tuple describing the position of widget_a in the parent coordinates of widget_b. 
