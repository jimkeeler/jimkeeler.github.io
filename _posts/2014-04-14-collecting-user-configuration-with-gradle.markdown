---
layout: post
title:  "Collecting User Configuration with Gradle"
date:   2014-04-14 16:30:00
---

At work we have a very manual process when packaging patches for our product. I wanted to alleviate some of the pain by automating the mindless/error-prone tasks, but there was a lot of metadata that needed to be gathered and stored in various places. I needed the file and source control power of a build tool, but the flexibility of a programming language for collecting metadata. After a few experiments I finally settled on Gradle.

Using Groovy’s [Swing Builder][1], I constructed a dialog with inputs for all of the patch information I needed:
{% highlight groovy %}
import javax.swing.*
import java.awt.Toolkit
import groovy.swing.SwingBuilder

task showDialog << {
  def patchFiles
  new SwingBuilder().edt {
      dialog(
        title:                 'Create patch...',
        defaultCloseOperation: JFrame.DISPOSE_ON_CLOSE,
        size:                  [685, 419],
        show:                  true,
        modal:                 true,
        locationRelativeTo:    null,
        iconImage:             Toolkit.getDefaultToolkit().getImage("resources/package_add.png"),
        resizable:             false){
          lookAndFeel("system")
          panel(layout: null){
            label(text: "JIRA Ticket", bounds: [10, 11, 73, 14])
            textField(bounds: [72, 8, 105, 20], columns: 10)

            label(text: "Patch Description:", bounds: [10, 36, 105, 14])
            scrollPane(bounds: [10, 52, 600, 107]) {
              textArea()
            }

            label(text: "Patch Files:", bounds: [10, 170, 73, 14])
            scrollPane(bounds: [10, 186, 600, 144]) {
              patchFiles = table(fillsViewportHeight: true, selectionMode: ListSelectionModel.SINGLE_SELECTION)
              def customModel = new javax.swing.table.DefaultTableModel(
                [] as Object [][],
                ['File', 'Description'].toArray())
              patchFiles.setModel(customModel)
            }

            button(bounds: [620, 186, 39, 30], icon: new ImageIcon("resources/add.png"), toolTipText: "Add files", actionPerformed: {
              def fc = fileChooser(dialogTitle: "Choose files to include in the patch...", fileSelectionMode: JFileChooser.FILES_ONLY, multiSelectionEnabled: true)
              if(fc.showOpenDialog() != JFileChooser.APPROVE_OPTION) return
              for(f in fc.selectedFiles) {
                patchFiles.getModel().addRow(f)
              }
              patchFiles.model.fireTableDataChanged()
            })

            button(bounds: [620, 216, 39, 30], icon: new ImageIcon("resources/delete.png"), toolTipText: "Remove selected file", actionPerformed: {
              patchFiles.model.removeRow(patchFiles.getSelectedRow())
              patchFiles.model.fireTableDataChanged()
            })

            button(bounds: [620, 270, 39, 30], icon: new ImageIcon("resources/arrow_up.png"), toolTipText: "Move selected files earlier in execution order", actionPerformed: {
              def selectedRow = patchFiles.getSelectedRow()
              if(selectedRow == 0) return
              patchFiles.model.moveRow(selectedRow, selectedRow, selectedRow - 1)
              patchFiles.setRowSelectionInterval(selectedRow - 1, selectedRow - 1)
            })

            button(bounds: [620, 300, 39, 30], icon: new ImageIcon("resources/arrow_down.png"), toolTipText: "Move selected files later in execution order", actionPerformed: {
              def selectedRow = patchFiles.getSelectedRow()
              if(selectedRow + 1 == patchFiles.getRowCount()) return
              patchFiles.model.moveRow(selectedRow, selectedRow, selectedRow + 1)
              patchFiles.setRowSelectionInterval(selectedRow + 1, selectedRow + 1)
            })

            button(text: "Cancel", bounds: [455, 347, 89, 23], actionPerformed: {
              dispose()
            })

            button(text: "Create Patch", bounds: [554, 347, 105, 23], actionPerformed: {
              // TODO: Copy data to project properties
              dispose()
            })
          }
      }
  }
}
{% endhighlight %}

(Note: I cheated a little and used Eclipse’s [WindowBuilder][2] to design the UI. I then had to translate the generated code into a Groovy equivalent.)

Now, when I execute the “showDialog” task, a nice UI is displayed to collect the patch metadata:
![A simple patch UI built using Groovy’s Swing Builder.](/assets/screenshot.png)

While this solution provides a nice front-end for my patch builder, I want to make sure it aligns with Gradle’s best practices. In the Gradle documentation, I discovered a chapter on [initialization scripts][3] which mentions the following:

> 60.1. Basic usage
>   Initialization scripts (a.k.a. init scripts) are similar to other scripts in Gradle. These scripts, however, are run before the build starts. Here are several possible uses:
>
> * Set up enterprise-wide configuration, such as where to find custom plugins.
> * Set up properties based on the current environment, such as a developer’s machine vs. a continuous integration server.
> * Supply personal information about the user that is required by the build, such as repository or database authentication credentials.
> * Define machine specific details, such as where JDKs are installed.
> * Register build listeners. External tools that wish to listen to Gradle events might find this useful.
> * Register build loggers. You might wish to customize how Gradle logs the events that it generates.
>
>   One main limitation of init scripts is that they cannot access classes in the buildSrc project (see Section 59.3, “Build sources in the buildSrc project” for details of this feature).

My interpretation is that this is a good spot in the build lifecycle to collect customized configuration or metadata. I also like that the init script (`init.gradle`) is separate from my build script (`build.gradle`) and must be called explicitly on the command line with the `--init-script` or `-I` options. This means if I later want to build a patch using a properties file or some other configuration method, I can just drop the init script.

One of the caveats of using an init script is that it executes very early in the build lifecycle. To set data on a Project object, you need to wait until it has been evaluated. I wrapped my Swing Builder in an `allProjects.afterEvaluate` closure to make sure I could set extra properties on the Project object:

{% highlight groovy %}
allprojects {
  afterEvaluate { project ->
    new SwingBuilder().edt { ... }
  }
}
{% endhighlight %}

[1]: http://groovy.codehaus.org/Swing+Builder
[2]: http://www.eclipse.org/windowbuilder/
[3]: http://www.gradle.org/docs/current/userguide/init_scripts.html
