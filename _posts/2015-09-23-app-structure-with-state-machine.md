---
layout: site
title: Structure your app with a State Machine
contributors: [marco_piccolino]
---

*Contributors:* {% for contributor in page.contributors %} -- {{ site.authors[contributor].name }} {% endfor %}

**Problem:** to keep your app's structure manageable and efficient.

**Baseline:** for simpler cases, a [StackView](http://doc.qt.io/qt-5/qml-qtquick-controls-stackview.html#using-stackview-in-an-application) works well.

**Improvement:** for more complex cases, a [Declarative State Machine](http://doc.qt.io/qt-5/qmlstatemachine.html) helps keeping app structure manageable.

<pre>
import QtQuick.Window 1.2  
    Window {
}
</pre>
