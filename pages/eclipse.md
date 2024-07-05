# Nit Developer Tools for Eclipse

## Installing

For Eclipse Juno

1. In the main menu, go to `Help > Install New Software...`
2. In the "Work with" fields, enter `http://nitlanguage.org/eclipse`
3. Select the feature "Nit Developer Tools"
4. The next window is a list of the tools to be downloaded. Click Next.
5. Read and accept the license agreements, then click Finish.
6. Note: If you get a security warning saying that the authenticity or validity of the software can't be established, click OK.
7. When the installation completes, restart Eclipse.

## Features

* Syntax highlighting on *.nit source files
* Background compiler to spot errors and warnings (require a working Nit installation)
* Nit perspective
* Nit project wizard
* Basic completion (ctrl+space)
* Parametrizable editor
* Toggle comment
* Indentation correction

## Using

### Configuring the Nit Environment

1. In the main menu, go to `Windows > Preferences`
2. Select `Nit`
3. Set the `Nit Folder Location` to the root of your Nit installation
4. The other directories should then be automatically set
5. Click OK.

### Creating a new Nit Project

1. In the main menu, go to `File > New > Nit...`
2. Select wizard `Nit Project`
3. Click Next.

### Using the Nit Development Perspective

1. In the main menu, go to `Windows > Open Perspective > Other...`
2. Choose `Nit`

### Creating a new Nit module

1. In the main menu, go to File > New > File
2. Type your file name followed by ".nit"
3. Finish

### Compiling and Running

0. Ensure that your perspective is "Nit"
1. In the main menu, go to `Run > Run Configurations...`
2. Create a new Nit Program configuration (double click)
3. Choose where to save the output for the compiler and the file to compile, you can also set the arguments for compiling and executing the program.
4. To see the output (and enter input), open a console by `Window > Show View > Console`
5. If you want to reuse this config, use the popup menu of the green play icon in the toolbar.

Note : it is also possible to skip this step by asking Eclipse to use the current selection (editor or explorer) as the default target for future executions.
