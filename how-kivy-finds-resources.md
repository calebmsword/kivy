# How Kivy finds resources

When you write `my_image = Image(source="my_image.png")`, how does Kivy resolve the filepath represented by `"my_image.png"`?

Before we continue, we should establish what the "Kivy data directory" is. If you don't know, the Kivy data directory is is a generic "data dump" directory through which one can put images, logos, and even kv files which "style" the widgets in your application. By default, the Kivy data directory is located within the Kivy package itself (on Windows, for example, this will look like `C:\Users\<your_username>\AppData\Local\Programs\Python\<your_python_version>\lib\site-packages\kivy\data`). But you can make the Kivy data directory any generic directory by assigning a filepath to the `KIVY_DATA_DIR` environment variable on your machine.

Now, let's answer the question. Kivy has a method found at `kivy.resources.resource_find`. It takes one argument, a string, and returns a string representing the absolute filepath of the file found (or it returns `None` if no file is found). Every widget which can be given a file uses this method to find the file. So, how does `resource_find(str)` resolve the filepath represented by the string `str`? 

<ol>
	<li><code>resource_find(str)</code> first attempts to resolve <code>str</code> as an absolute filepath. If a file is found at the absolute filepath represented by <code>str</code>, then <code>resource_find(str)</code> simply returns <code>str</code>.</li>
	<li>If the string, as an absolute filepath, does not represent any file on the machine, <code>resource_find(str)</code> will see if <code>str</code> is a filepath relative to one of many paths contained in a specific list of paths. More specifically, it will see if <code>os.path.join(path, str)</code>, where <code>path</code> is a path from the list of paths, represents some file on the system. If a file is found, the absolute filepath of that file is returned and no further paths are tried. The following paths will be tried in the listed order:</li>
	<ol>
		<li><code>&ltkivy_data_directory&gt/fonts</code></li>
		<li>Where fonts are stored on your local machine. (On Windows, this look like <code>C:\WINDOWS\Fonts</code>.)</li>
		<li>The parent directory of the Kivy data directory.</li>
		<li>The parent directory of the default Kivy data directory.</li>
		<li>If Kivy is being run on iOS, it will look for a directory called <code>YourApp</code> in the parent directory of the script which starts of the Kivy app. On any other platform, this path is skipped.</li>
		<li>The directory containing the Python script which starts the Kivy app.</li>
		<li>The current working directory of the process running the Python script which started the app.</li>
	</ol>
</ol>

The Kivy internals which use `resource_find` have different behaviors when `resource_find` returns `None`. Most widgets which manage a file, such as the `Image` widget, raise an Error when the resource cannot be found. Other times, Kivy will simply perform some sort of default behavior (such as `app::get_application_icon`, which returns an empty string if no icon is found). 

If this list of directories is not suitable for your circumstances, you can modify the list of paths with `kivy.resources.resource_add_path` or `kivy.resources.resource_remove_path`. Note that the most recently added path is always searched _first_. The list of paths is stored in the Python list `kivy.resources.resource_paths` and the list is searched in reverse, starting from the last element and working towards the first. However, you're not supposed to access this list directly. In 99% of use cases, `resource_add_path` and `resource_remove_path` will give you adequate control of `resource_paths`.

You might wonder why Kivy searches the _parent_ of the data directory instead of the data directory itself. The reason is that, to access something the data directory, you can write something like `my_image = Image(source="data/my_image.png")`, something that will work on any platform given that the environment is configured correctly.

Obscure edge cases:
 - Paths i and ii are only included in the list of paths if your application includes a widget which renders text (or more specifically, if the [core provider](https://kivy.org/doc/stable/guide/architecture.html#core-providers-and-input-providers) which renders text is used).
 - If the string starts with the characters `"atlas://"`, then `resource_find` will simply return the given string. Kivy's internals which render images handle this edge case appropriately, so it's unlikely you will ever have to worry about this. If you are curious what this is for, see [this section of the Kivy docs](https://kivy.org/doc/stable-2.0.0/api-kivy.atlas.html).
 - If `resource_find(str)` would return `None` but `str` starts with the characters `"data:"`, then `resource_find` will return `str` instead of `None`. I do not know why this behavior exists.
