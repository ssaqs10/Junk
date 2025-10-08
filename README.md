# Junk
Store Snippets of Code to used for changes repo




```
ui/qt/main_window.ui
```

```xml
<widget class="QMenuBar" name="menubar">
 <property name="geometry">
  <rect>
   <x>0</x>
   <y>0</y>
   <width>800</width>
   <height>22</height>
  </rect>
 </property>
 <widget class="QMenu" name="menuFile">
  <property name="title">
   <string>File</string>
  </property>
  ...
 </widget>
 ...
</widget>
```

---

## Remove or comment out the menubar widget

Option A — Delete this `<widget class="QMenuBar" ...>` section entirely.

Option B — Comment it out, like this:

```xml
<!--
<widget class="QMenuBar" name="menubar">
   ... all menu definitions ...
</widget>
-->
```

That will prevent `uic` (Qt’s UI compiler) from generating the menu bar at all.

---


After you remove the menu definition, the generated code will no longer create `main_ui_->menubar`.

So in `ui/qt/main_window.cpp`, find lines like this:

```cpp
main_ui_->menubar = menuBar();
setMenuBar(main_ui_->menubar);
```

and **comment them out** if present.

Then, optionally add:

```cpp
menuBar()->hide();
```

to be safe (though if the UI element is gone, this won’t do anything).

---

Excellent 👍 — here’s the **follow-up edit** you should make in
`ui/qt/main_window.cpp` after removing (or commenting out) the `<QMenuBar>` block from `main_window.ui`.

This step prevents any null-pointer or UI layout warnings when Wireshark starts and the `menubar` object no longer exists.

---


1. **Open the file** and locate the `MainWindow::MainWindow(QWidget *parent)` constructor.
   It typically begins like this:

   ```cpp
   MainWindow::MainWindow(QWidget *parent) :
       QMainWindow(parent),
       main_ui_(new Ui::MainWindow),
       ...
   {
       main_ui_->setupUi(this);
       ...
   }
   ```

2. **After** the call to `main_ui_->setupUi(this);`,
   insert the following lines:

   ```cpp
   // --- BEGIN: Disable/Hide Menu Bar if removed from UI ---
   if (menuBar()) {
       menuBar()->hide();            // Hides the QMenuBar if Qt auto-creates one
       menuBar()->setFixedHeight(0); // Removes any blank space at the top
   }
   setMenuBar(nullptr);              // Ensure no dangling pointer to menubar
   // --- END: Disable/Hide Menu Bar ---
   ```

   This ensures:

   * The layout doesn’t leave an empty strip.
   * `QMainWindow::setMenuBar()` has no invalid reference.
   * No runtime warning like “QMainWindow::menuBar: menu bar already deleted”.

3. If you previously saw lines like:

   ```cpp
   setMenuBar(main_ui_->menubar);
   ```

   or

   ```cpp
   main_ui_->menubar = menuBar();
   ```

   → **Comment those out**:

   ```cpp
   // setMenuBar(main_ui_->menubar);
   // main_ui_->menubar = menuBar();
   ```

---


Perfect — here’s the **exact C++ snippet** you can safely paste into your
`ui/qt/main_window.cpp` file inside the **`MainWindow::MainWindow(QWidget *parent)`** constructor to disable *all* keyboard shortcuts (Ctrl+O, Ctrl+E, etc.) globally.

This method works cleanly across all Wireshark 3.2 / `master-3.2` builds — even if your exact file layout differs slightly.

---

## 🧩 **Exact Code to Add**

Locate this section in `ui/qt/main_window.cpp`:

```cpp
MainWindow::MainWindow(QWidget *parent) :
    QMainWindow(parent),
    main_ui_(new Ui::MainWindow)
{
    main_ui_->setupUi(this);
```

Right **after** `main_ui_->setupUi(this);`, insert the following block 👇

```cpp
    //
    // --- REMOVE MENU BAR (if UI still tries to create one) ---
    //
    if (menuBar()) {
        menuBar()->hide();            // Hide the menu bar if Qt auto-created one
        menuBar()->setFixedHeight(0); // Prevent any blank strip at the top
    }
    setMenuBar(nullptr);              // Ensure no invalid references

    //
    // --- DISABLE ALL SHORTCUTS ---
    //
    // This removes all keyboard accelerators (Ctrl+O, Ctrl+E, etc.)
    // so that nothing from removed menus can be opened.
    //
    foreach (QAction *a, findChildren<QAction *>()) {
        a->setShortcut(QKeySequence());
    }
```

Would you like me to show a **ready-to-apply `.patch` file** that combines this change with the menu bar removal (so you can `git apply` it directly)?



