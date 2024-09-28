# Some tips of using this repo

### 1. 如何批量导出`Publications`?

- Zotero 中选中需要导出的文献。
- 右击，点击`Export Items`，选择`BibTeX`格式.
- 保存到`_Publications`.
- 修改`pubsFromBib.py`中的`publist`变量。
- 在`markdown_generator`下运行`pubsFromBib.py`。
- 将`category: manuscripts`添加到一作文章中去。
