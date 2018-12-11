I recently worked on a Chatbot that would recommend a wine based on document similarity. One of the harder aspects I encountered was developing a nice UI that would serve as the front end for my chatbot. To this end, I set out to learn how to use [PyQt5](https://pypi.org/project/PyQt5/).

Here is a visual of the final UI in my project. In the code I’ll walk you through how to establish this kind of setup.

![winebot](../images/winebot1.png =200x200) ![winebot](../images/winebot2.png =200x200)

Here are the imports for my project. `QtWidgets` is the main import as this is what creates our app via `QApplication`, and when we create an `Window` class object we use `QtWidgets.QWidget`. The only functionality I use `QtCore.Qt` for is to switch the alignment of my `QTextEdit` object from left to right. This gives me the flexibility to give the impression of a back and forth chat dialog by putting the bot text on the right and the user text on the left. `QtGui.QFont` is used to resize the font of the button object and the user input `QLineEdit` object.


```python
# Imports for PyQt5 Lib and Functions to be used
from PyQt5 import QtWidgets
from PyQt5.QtCore import Qt
from PyQt5.QtGui import QFont
from PyQt5.QtWidgets import QWidget,QApplication
```

The following stylings are assigned to variables which I'll use later on to set the style on my `QTextEdit` and `QLineEdit` objects.


```python
# alignment to PyQt Widgets
setStyleQte = """QTextEdit {
    font-family: "Courier"; 
    font-size: 12pt; 
    font-weight: 600; 
    text-align: right;
    background-color: Gainsboro;
}"""

setStyletui = """QLineEdit {
    font-family: "Courier";
    font-weight: 600; 
    text-align: left;
    background-color: Gainsboro;
}"""
```

As the Doc String for `__init__` implies here we define all the objects that will be used to construct our PyQt App. There are various layouts available in PyQt. I chose the QVBoxLayout because it worked best for my project. The important thing to note is that I have a `QTextEdit` and a `QLineEdit` object. Why do I have two different types of text containers? `QTextEdit` will expand depending on the size of my app. `QLineEdit` as the name implies will stay as a single line and will not grow when I adjust the app height or change the window geometries. You can see how I would want the size of my user input to remain static regardless of how I change app dimesions whereas the Chat dialog box (`QTextEdit`) can expand as this is OK for the project and what I'm trying to do.


```python
class Window(QtWidgets.QWidget):
    def __init__(self):
        '''
        Initilize all the widgets then call the GuiSetup to customize them
        '''
        QtWidgets.QWidget.__init__(self)
        self.v = None
        self.layout = QtWidgets.QVBoxLayout(self)
        self.button2 = QtWidgets.QPushButton('Start New Session')
        self.font = QFont()
        self.font.setPointSize(12)
        self.chatlog = QtWidgets.QTextEdit()
        self.userinput = QtWidgets.QLineEdit()
        self.userinput.returnPressed.connect(self.AddToChatLogUser)
        self.button2.clicked.connect(self.getBot)
        self.GuiSetup()
```

Yes, `GuiSetup()` gets called in the `__init__`. I could have just put these lines in there, but I wanted to be explicit that these lines are tweaking the widgets. Recall that we created 'styles' to be applied to the widgets prior. Here we apply those parameters with `.setStyleSheet()`. The `self.layout.addWidget` calls sequentially add the widgets from top to bottom (vertically) thus now you know why it is called `QVBoxLayout` because it is a box laid out vertically. What is important to note is that the button and user input widgets do not expand when the window size changes while chatlog (a `QTextEdit` object) will expand. This is exactly the flavoring we want for our chat box.


```python
    def GuiSetup(self):
        '''
        Styling and Layout.
        '''
        self.chatlog.setStyleSheet(setStyleQte)
        self.userinput.setStyleSheet(setStyletui)
        self.userinput.setFont(self.font)
        self.button2.setFont(self.font)
        self.layout.addWidget(self.button2)
        self.layout.addWidget(self.chatlog)
        self.layout.addWidget(self.userinput)
```

In `UpdateCycle` the app will set the alignment to right justified via `setAlignment(Qt.AlignRight)` because we want our bot messages to appear on the right side. In the `AddToChatLogUser` call, we set the alignment left with `Qt.AlignLeft` append the user's input with `.append` and then explicitly reset the alignment back to right as the next output will be the bot responding to the user. While it's true that the bot's response is aligned right before posting, it's always a good idea to be explicit about my default state which I choose as being right aligned.


```python
    def UpdateCycle(self):
        '''
        Retrieves a new bot message and appends to the chat log.
        '''
        bmsg = self.v.getBotMessage()
        self.chatlog.setAlignment(Qt.AlignRight)
        [self.chatlog.append(m) for m in bmsg]
        self.userinput.setFocus()
```


```python
    def AddToChatLogUser(self):
        '''
        Takes guest's entry and appends to the chatlog
        '''
        umsg = self.userinput.text()
        self.chatlog.setAlignment(Qt.AlignLeft)
        self.chatlog.append(umsg)
        self.chatlog.setAlignment(Qt.AlignRight)
        self.userinput.setText("")
```

Here's the call to self when I'm opening this code stand-alone. The important thing to note is `setGeometry`. Setting the third and fourth parameter to 480 I essentially create a box app 480x480 pixels.


```python
if __name__ == '__main__':
    app = QtWidgets.QApplication(sys.argv)
    win = Window()
    win.setGeometry(10,10,480,480)
    win.show()
    sys.exit(app.exec_())
```
I hope you enjoyed reading and remember:

Stay Chaotic – Stay Neutral

[ARI](mailto:ari.virrey@gmail.com)
