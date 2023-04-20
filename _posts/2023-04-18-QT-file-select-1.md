---
title: QT 파일 선택 위젯 만들기(1)
categories: [QT]
tags: [QT, Python]
comments: true
---

탐색기에 있는 파일을 끌어다 놓거나 버튼을 눌러 파일을 추가하는 위젯을 만들 것이다.

## Drop 이벤트를 처리할 수 있는 Line Edit Widget 만들기

```python
'''
filedroplineedit.py
'''
from PyQt5.QtWidgets import QLineEdit, QToolButton
from PyQt5.QtGui import QDragEnterEvent, QDropEvent
from PyQt5.QtCore import pyqtSignal

class FileDropLineEdit(QLineEdit):
    # 파일 첨부 시그널
    fileDropped = pyqtSignal(str)

    def __init__(self, parent=None):
        super(FileDropLineEdit, self).__init__(parent)

        # 드롭 이벤트를 처리할 수 있도록 위젯 설정
        self.setAcceptDrops(True)

        self.setPlaceholderText("파일을 여기에 끌어다 놓으세요.")
        self.setReadOnly(True)
        self.setStyleSheet(
            "padding: 3px;"
            "border-style: solid;"
            "border-width: 1px;"
            "border-color: gray;"
            "border-radius: 3px")
        self.setClearButtonEnabled(True)
        self.findChild(QToolButton).setEnabled(True)
        self.textChanged.connect(lambda: self.clear)

    def dragEnterEvent(self, event: QDragEnterEvent):
        # 드래그 앤 드롭을 허용할 데이터 타입이 있는지 확인합니다.
        if event.mimeData().hasUrls():
            event.accept()
        else:
            event.ignore()

    def dropEvent(self, event: QDropEvent):
        # 파일 경로를 가져와서 시그널을 발생시킵니다.
        files = [u.toLocalFile() for u in event.mimeData().urls()]
        all_files = ""
        for file_path in files:
            self.fileDropped.emit(file_path)
            all_files += file_path + ';'

        if all_files:
            self.clear()
            self.setText(all_files[:-1])
```

**`setReadOnly(True)`**를 호출하면 QLineEdit 위젯을 사용자가 수정할 수 없도록 설정한다. 이 설정 때문에 사용자가 Clear 버튼을 누르는 것과 같은 수정 작업을 수행하면 `textChanged()` 시그널은 발생하지 않는다.

따라서, **`findChild(QToolButton).setEnabled(True)`**를 호출하여 Clear 버튼을 활성화해 준다. 이렇게 하면 Clear 버튼을 누를 때 `textChanged()` 시그널이 발생한다.

## 파일 선택 위젯 만들기

1. 빈 Widget을 하나 생성한 뒤, **QLabel, QLineEdit, QPushButton**을 추가한다.
2. QLineEdit을 FileDropLineEdit으로 승격시킨다.

![qt_sec_1](/assets/img/post/qt_sec_1.png)

![qt_sec_2](/assets/img/post/qt_sec_2.png)

1. 끌어다 놓기 이외에도 파일 추가 버튼을 눌러 파일을 추가할 수 있도록 한다.

```python
'''
fileselectorwidget.py
'''
import sys
from PyQt5.QtWidgets import *
from PyQt5.QtCore import pyqtSignal
from PyQt5 import uic

# UI 파일 로드
form_class = uic.loadUiType("fileselectorwidget.ui")[0]

class FileSelectorWidget(QWidget, form_class):
    # 파일 첨부 시그널
    fileSelected = pyqtSignal(str)

    def __init__(self, parent=None):
        super(FileSelectorWidget, self).__init__(parent)
        self.setupUi(self)

        self.fileDropLineEdit.fileDropped.connect(self.handle_drop_file)
        self.fileAddButton.clicked.connect(self.handle_upload_file)
        self.fileAddButton.setStyleSheet(
            "padding: 3px;"
            "border-style: solid;"
            "border-width: 1px;"
            "border-color: gray;"
            "border-radius: 3px")

    def handle_upload_file(self):
        # 파일 선택 대화상자 열기
        file_path, _ = QFileDialog.getOpenFileName(self, "파일 업로드", "", "All Files (*)")

        # 파일 선택 후 처리
        if file_path:
            self.fileWidget.clear()
            self.fileWidget.setText(file_path)
            self.fileSelected.emit(file_path)

    def handle_drop_file(self, file_path):
        self.fileSelected.emit(file_path)

if __name__ == "__main__":
    # QApplication 인스턴스 생성
    app = QApplication(sys.argv)
    myWindow = FileSelectorWidget()
    myWindow.show()
    app.exec_()
```

![qt_sec_3](/assets/img/post/qt_sec_3.png)

![qt_sec_4](/assets/img/post/qt_sec_4.png)

## 파일 선택 위젯 사용하기

빈 Form에 Widget을 추가해서 해당 위젯을 FileSelectorWidget으로 승격하여 사용하면 된다.

아래 예시는 Dialog에 FileSelectorWidget을 추가한 것이다.
![qt_sec_5](/assets/img/post/qt_sec_5.png)
