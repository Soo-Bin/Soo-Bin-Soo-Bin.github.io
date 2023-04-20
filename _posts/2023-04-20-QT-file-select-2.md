---
title: QT 파일 선택 위젯 만들기(2)
categories: [QT]
tags: [QT, Python]
comments: true
---

[파일 선택 위젯(1)](https://soo-bin.github.io/posts/QT-file_select_1/) 보다 기능이 추가된 위젯을 만들 것이다.

탐색기에 있는 파일/폴더를 끌어다 놓거나 버튼을 눌러 파일/폴더를 추가/삭제할 수 있다.

## Drop 이벤트를 처리할 수 있는 Tree Widget 만들기

```python
'''
filedroptreewidget.py
'''
import os
from PyQt5.QtWidgets import QTreeWidget, QTreeWidgetItem
from PyQt5.QtCore import Qt
from PyQt5.QtGui import QIcon

class FileDropTreeWidget(QTreeWidget):
    def __init__(self, parent=None):
        super().__init__(parent)

        self.setHeaderHidden(True)
        self.setSelectionMode(QTreeWidget.ExtendedSelection)
        self.setAlternatingRowColors(True)

        # 드롭 이벤트를 받도록 설정
        self.setAcceptDrops(True)

    # 드래그 앤 드롭 액션이 들어갈 때 위젯으로 보내는 이벤트
    def dragEnterEvent(self, event):
        if event.mimeData().hasUrls():
            event.accept()
        else:
            event.ignore()

    # 드래그 앤 드롭 동작이 진행되는 동안 전송되는 이벤트
    def dragMoveEvent(self, event):
        if event.mimeData().hasUrls():
            event.setDropAction(Qt.CopyAction)
            event.accept()
        else:
            event.ignore()

    # 드래그 앤 드롭 동작이 완료되면 전송되는 이벤트
    def dropEvent(self, event):
        # 드랍된 파일 정보 가져오기
        if event.mimeData().hasUrls():
            event.setDropAction(Qt.CopyAction)
            event.accept()
            for url in event.mimeData().urls():
                path = str(url.toLocalFile())
                if os.path.isfile(path):
                    # 파일인 경우에는 QTreeWidgetItem을 생성하여 TreeWidget에 추가
                    self.add_file_icon_item(path)
                else:
                    self.add_folder_icon_item(path)
        else:
            event.ignore()

    def add_file_icon_item(self, root):
        item = QTreeWidgetItem(self, [root])
        item.setIcon(0, QIcon("icon/mIconFile.svg"))  # 파일 아이콘 설정
        self.addTopLevelItem(item)

    def add_folder_icon_item(self, root):
        item = QTreeWidgetItem(self, [root])
        item.setIcon(0, QIcon("icon/mIconFolder.svg"))  # 폴더 아이콘 설정
        self.addTopLevelItem(item)
        self.add_subdirs(item, root)

    def add_subdirs(self, parent, root):
        for dirpath, dirs, files in os.walk(root):
            for dir_name in dirs:
                sub_dir_path = os.path.join(dirpath, dir_name)
                sub_dir_item = QTreeWidgetItem(parent, [sub_dir_path])
                sub_dir_item.setIcon(0, QIcon("icon/mIconFolder.svg"))
                parent.addChild(sub_dir_item)
                self.add_subdirs(sub_dir_item, sub_dir_path)

            for filename in files:
                file_path = os.path.join(dirpath, filename)
                file_item = QTreeWidgetItem(parent, [file_path])
                file_item.setIcon(0, QIcon("icon/mIconFile.svg"))  # 파일 아이콘 설정
                parent.addChild(file_item)
```

## Drop 이벤트 검증 및 파일 선택 위젯 만들기

1. 아래 그림과 같은 Form을 만든다. Tree Widget은 FileDropTreeWidget으로 승격시킨다.
   ![qt_trd_1](/assets/img/post/qt_trd_1.png)
2. 추가된 버튼들에도 기능을 추가해준다.

```python
'''
fileselectorlistwidget.py
'''
import sys
from PyQt5.QtWidgets import *
from PyQt5 import uic

# UI 파일 로드
form_class = uic.loadUiType("fileselectorlistwidget.ui")[0]

class FileSelectorListWidget(QWidget, form_class):
    def __init__(self):
        super().__init__()
        self.setupUi(self)

        # 버튼에 대한 이벤트 처리
        self.btnFileAdd.clicked.connect(self.add_file)
        self.btnFileRemove.clicked.connect(self.remove_item)
        self.btnFolderAdd.clicked.connect(self.add_folder)

    def add_file(self):
        # 파일 선택 다이얼로그 생성
        file_dialog = QFileDialog(self)
        file_dialog.setFileMode(QFileDialog.ExistingFiles)
        if file_dialog.exec_():
            file_names = file_dialog.selectedFiles()

            # 선택된 파일들 Tree Widget에 추가
            for file_name in file_names:
                self.fileTreeWidget.add_file_icon_item(file_name)

    def remove_item(self):
        # 선택된 아이템 삭제
        root = self.fileTreeWidget.invisibleRootItem()
        for item in self.fileTreeWidget.selectedItems():
            (item.parent() or root).removeChild(item)

    def add_folder(self):
        # 폴더 선택 다이얼로그 생성
        dir_dialog = QFileDialog(self)
        dir_dialog.setFileMode(QFileDialog.Directory)
        if dir_dialog.exec_():
            dir_name = dir_dialog.selectedFiles()[0]

            # 선택된 폴더 Tree Widget에 추가
            self.fileTreeWidget.add_folder_icon_item(dir_name)

if __name__ == "__main__":
    # QApplication 인스턴스 생성
    app = QApplication(sys.argv)
    myWindow = FileSelectorListWidget()
    myWindow.show()
    app.exec_()
```

- **`invisibleRootItem()`** : QTreeWidget 클래스에서 제공하는 트리 위젯의 루트 아이템을 반환하는 메소드

- **`(item.parent() or root).removeChild(item)`** : `item.parent()`는 item의 부모 item을 반환합니다. 만약 item이 top-level item이라면 부모가 없으므로 `None`을 반환. `item.parent()`가 `None`인 경우에는 `root.removeChild(item)`을 실행하고, 그렇지 않은 경우에는 `item.parent().removeChild(item)`
  을 실행

|                                            |                                            |
| ------------------------------------------ | ------------------------------------------ |
| ![qt_trd_2](/assets/img/post/qt_trd_2.png) | ![qt_trd_3](/assets/img/post/qt_trd_3.png) |

### Icon Source

- [https://github.com/qgis/QGIS/tree/master/images/themes/default](https://github.com/qgis/QGIS/tree/master/images/themes/default)
