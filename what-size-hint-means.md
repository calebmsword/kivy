# What size hint means

Not every Layout listens to the `size_hint` attribute of its children. But for those that do, there are two general approaches to how they interpret the `size_hint` property.

The first is that the `size_hint_x` represents a percent width of the containing Layout, and `size_hint_y` represents a percent height of the containing Layout. We will refer to any Layout which interprets `size_hint` in this way as "**percent-like**".

The second approach is very similar to how CSS treats the "flex-grow" property. I can't explain this better than how the [following image](https://css-tricks.com/snippets/css/a-guide-to-flexbox/#aa-flex-grow) describes it:

![image](https://github.com/calebmsword/kivy/assets/85499281/a5e43781-8caa-4d18-867f-50b29db511af)

We will refer to any Layout which interprets `size_hint` in this way as "**flex-like**".

Another useful concept is **allotted width** and **allotted height**. Some Layouts will subtract a fixed number of pixels from the width and height to create an allotted width and allotted height, and then divies up this remaining space amongst its children. For example:

 - If an AnchorLayout has left padding and right padding, it will subtract that from its total width when determining its allotted width
 - If a BoxLayout has three children and its vertical spacing is 10 pixels, it will subtract 20 pixels from either its allotted height
 - If a BoxLayout has a child with a `size_hint_x` of `None` and a `width` of 100, then the Layout will subract 100 pixels from its allotted width  

We will now go into more specific detail about the Layouts which listen to `size_hint`, with one exception: I will not describe how the `size_hint_mix_x` and `size_hint_max_y` and other similar attributes affect things. If I get around to it, I will describe the effects of those in another chapter.

### FloatLayout and similar widgets

There are three "widgets" which behave like FloatLayout:

 - FloatLayout
 - RelativeLayout
 - Window

The global `Window` object is technically not a widget. However, the root widget of the application's `parent` attribute points to the `Window` object, and the `Window` object actually manages the size of the root widget. `Window` can, unintuitively, also have multiple widgets as children and manage all of their sizes. So it is fair in this context to treat it like a widget.

These Layouts listen are all percent-like in the most straighforward way. `size_hint_x` represents a percentage of the width of the containing Layout and `size_hint_y` represents a percentage of the height of the containing Layout. That's it.

### AnchorLayout and size hint

AnchorLayout is also percent-like, but it also calculates an allotted width and allotted height, and the `size_hint_x` is a percentage of the AnchorLayout's allotted width, not its total width.

An AnchorLayout's allotted width is its total width minus the sum of its left and right padding, while the allotted height is its total height minus the top and bottom padding. The `padding` attribute is a list with four elements which determines the left, top, right, and bottom padding values in that order.

### BoxLayout and size hint

BoxLayout is an interesting case, as it is percent-like and flex-like at the same time. The behavior also depends on its `orientation` attribute (which is either `"horizontal"` or `"vertical"`).

- If `orientation` is `"horizontal"`:
  - `size_hint_x` is flex-like.
    - The BoxLayout also calculates an allotted width by subtracting the following values from the total width:
      - The left and right padding
      - The value of the `spacing` attribute multiplied by (1 - (number of children))
      - The sum of `width`s of all children whose `size_hint_x` are `None` 
  - `size_hint_y` is percent-like.
    - The BoxLayout calculates an allotted height by subtracting the top and bottom padding from the its total height.
- If `orientation` is `"vertical"`:
  - `size_hint_x` is percent-like.
    - The BoxLayout calculates an allotted width by subtracting the left and right padding from its total width.
  - `size_hint_y` is flex-like
    - The BoxLayout also calculates the allotted height by subtracting the following values from the total height:
      - The top and bottom padding
      - The value of the `spacing` attribute multiplied by (1 - (number of children))
      - The sum of `height`s of all children whose `size_hint_y` are `None` 

### StackLayout and size hint

For StackLayouts, `size_hint` is percent-like. StackLayouts also calculate allotted widths and allotted heights. However, how the allotted width/height is calculated depends on the `orientation` of the StackLayout, which can be one of the strings `tb-lr`, `tb-rl`, `bt-lr`, `bt-rl`, `lr-tb`, `lr-bt`, `rl-tb`, `rl-bt`.

- If `orientation` starts with `"tb"` or `"bt"`:
  - Allotted width:
    - The allotted width is the same for each column.
    - Unintuitively, the StackLayout does not use the horizontal spacing value (`spacing[0]`) when calculating the allotted width.
    - The allotted width is simply total width of the Stacklayout minus the left and right padding.
    - The allotted width is not changed if one child has a fixed width (i.e., its `size_hint_x` is `None`)
  - Allotted height:
    - The allotted height can be different for each column.
    - The allotted height for the nth column in the total height minus:
      - The number of children in that column minus 1, multiplied by the vertical spacing value (`spacing[1]`)
      - The top and bottom padding
    - The allotted height is not changed if one child has a fixed height (i.e., its `size_hint_y` is `None`)
- If `orientation` starts with `"rl"` or `"lr"`:
  - Allotted width:
    - The allotted width can be different for each column.
    - The allotted width for the nth row is the total width minus:
      - The number of children in that row minus 1, multiplied by the horizontal spacing value (`spacing[0]`)
      - The top and bottom padding
    - The allotted width is not changed if one child has a fixed width (i.e., its `size_hint_x` is `None`)
  - Allotted height:
    - The allotted height is the same for each column.
    - Unintuitively, the StackLayout does not use the vertical spacing value (`spacing[1]`) when calculating the allotted height.
    - The allotted height is simply total height of the Stacklayout minus the top and bottom padding.
    - The allotted height is not changed if one child has a fixed height (i.e., its `size_hint_y` is `None`)

### GridLayouts and size hint

GridLayouts are by far the most complex Layout in regards to how they listen to `size_hint`. Most of the time, GridLayouts are flex-like but there are a lot of edge cases and nuances. Since there is so much to cover, so we will first go over some useful facts regarding GridLayouts.

Useful facts:
 - Rows are 0-indexed, so row 0 is the first row, row 1 is the second row, etc. No matter the orientation of the GridLayout (`tb-lr`, `bt-lr`, etc), the **topmost** row is row 0.
 - Columns are also 0-indexed. No matter the orientation of the GridLayout, the **leftmost** column is column 0.
 - GridLayouts create slots for each of their child widgets. The slots have the same height for every row and have the same width for every column. Widgets are placed in the bottom left of each slot. 
 - If the widget has a value for `size_hint_y` that is not `None`, then it will take the full height of the slot. If the `size_hint_y` is `None`, then it is possible for the widget to take less than or more than the height of the slot. A similar rule is followed for `size_hint_x`.
   - (There are expections to this if the child has values for `size_hint_y_min` or other similar attributes. But we'll discuss those another time). 
 - GridLayouts calculate a minimum height for each of its rows. The GridLayout ensures that the height of the slots in each row is at least the minimum height, even if the sum of minimum heights exceeds the height of the GridLayout. The GridLayout will even draw its children outside of the Window if it has to.
 -  When keeping track of these behaviors, it is extremely useful to remember the following default values for properties of GridLayouts and widgets:
    - GridLayout:
      - `force_row_default`: `False`
      - `rows_minimum`: `{}`
      - `row_height_default`: 0
    - Widget:
      - `size_hint`: `(1, 1)`
        - And consequently, `size_hint_y`: `1`
      - `size`: `(100, 100)`
        - And consequently, `height`: `100`

We will now describe the process for how `size_hint_y` is calculated. The behavior of `size_hint_x` follows in the natural way. Just copy-paste what is written below, but replace every "row" with "column", every `row` with `col`, every "height" with "width", every "vertical" with "horizontal", and so on.

The process:
 - If `cols` is `None` and `rows` is `None`:
   - The GridLayout does not manage the positions and sizes of its children. The `size_hint` attribute of any child of the GridLayout has no effect. 
 - If `force_row_default` set to `True`:
   - The the height of the slots for row `n` is determined by the keys in `rows_minimum`.
     - if `rows_minimum` contains the key `n`:
       - Then the height of the slots for row `n` is set to the value mapped by `n` in `rows_minimum`. 
     - If `rows_minimum` does not contain a key for `n`:
       - Then the height of the slots for row `n` is the value of `row_height_default`.
   - The slot height is now determined for every row. The children are then placed in the bottom left of each slot.
     - If the child has any non-`None` value for `size_hint_y`:
       - Then the child will fill the entire height of the slot.
     - If a child in row `n` has a `size_hint_y` set to `None`:
       - Then the GridLayout allows the user to determine the widget height. And value you assign to the `height` attribute becomes the height of the widget.
       - If `height` is less than the height of the slot, then the widget will not fill the entire vertical space of the slot.
       - If the `height` exceeds the height of the slot, then the widget will exceed the space of the slot.
   
 - If `force_row_default` is set to `False`:
   - The slot height for each row is the sum of the "minimum height" calculated for that row and a fraction of the "allotted height" of the GridLayout.
     - Minimum Height:
       - The GridLayout finds the largest of the following three values for the `n`th row and sets that value as the minimum height of that row.
         - If the key `n` exists in `rows_minimum`, then it uses the value mapped by `n`. If the key `n` is not in `rows_minimum`, then negative infinity.
         - If at least one child in the `n`th row has a `size_hint_y` set to `None`, then it uses the maximum height of every one of these children. If there are no children with a `size_hint_y` set to `None` in the `n`th row, then negative infinity.
         - The value of `row_height_default`.
     - Allotted Height:
       - The GridLayout determines the allotted height for each row by subtracting the following values from the height of the GridLayout:
         - The padding_top of the GridLayout (`padding[1]`)
         - The padding_bottom of the GridLayout (`padding[3]`)
         - The spacing between each row (`spacing[1]` * (number of rows - 1))
         - The sum of minimum heights for each row
       - If the allotted height is 0 or less than zero:
         - The slot height for every child in that row is just the minimum height and nothing more.
       - If the allotted height is greater than 0:
         - If every widget in the `n`th row has a `size_hint_y` with a value of `None`:
           - The slot height for every child in that row is just the minimum height and nothing more.
         - If at least one widget in row `n` has a non-`None` `size_hint_y`:
           - The GridLayout determines the maximum `size_hint_y` for every row which contains at least one child with a non-`None` `size_hint_y`.
           - The GridLayout sums of all maximum `size_hint_y`'s.
           - The GridLayout calculates the ratio of the maximum `size_hint_y` in row `n` to the sum of all maximum `size_hint_y`'s.
           - The GridLayout multiplies this ratio by the allotted height and adds that to the minimum height of row `n`.
    - The slot height is now determined for every row. The children are then placed in the bottom left of each slot.
      - If the child has a value of `size_hint_y` that is not `None`:
        - The child will take the full vertical space of the slot.
      - If a child has a `size_hint_y` whose value is `None`:
        - Then the GridLayout allows the user to determine the widget height. Any value you assign to the `height` attribute becomes the height of the widget.
        - Note that the `height` parameter will never exceed the height of the slot. If you review the process for how the slot height is calculated when `force_row_default` is `False`, you will see that the slot height for that row will increase to contain the widget if you try to increase the widget height beyond the slot height.
