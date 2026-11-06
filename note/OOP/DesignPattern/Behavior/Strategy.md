## Strategy Pattern

- what
  - Use Interface to avoid IF- Else
- How
  - Interface 
  - Different implementation 
  - Client use based on need 
- When 
  - Export to different format 

### Export 
```python
from abc import ABC, abstractmethod

# 策略接口：定义导出格式的通用行为
class ExportStrategy(ABC):
    @abstractmethod
    def export(self, title: str, content: str) -> str:
        pass
```
```python
# Markdown 格式策略
class MarkdownExporter(ExportStrategy):
    def export(self, title: str, content: str) -> str:
        return f"# {title}\n\n{content}"

# HTML 格式策略
class HtmlExporter(ExportStrategy):
    def export(self, title: str, content: str) -> str:
        return f"<h1>{title}</h1>\n<p>{content}</p>"

# 纯文本格式策略
class PlainTextExporter(ExportStrategy):
    def export(self, title: str, content: str) -> str:
        return title.upper() + "\n" + "=" * len(title) + "\n" + content
```
```python
class ReportExporter:
    def __init__(self, strategy: ExportStrategy):
        self._strategy = strategy

    # 运行时可以切换策略
    def set_strategy(self, strategy: ExportStrategy):
        self._strategy = strategy

    def export(self, title: str, content: str) -> str:
        return self._strategy.export(title, content)
```
```python
if __name__ == "__main__":
    exporter = ReportExporter(MarkdownExporter())
    print(exporter.export("Monthly Report", "Sales grew by 20% this month"))
    # # Monthly Report
    #
    # Sales grew by 20% this month

    # 需要 HTML 格式？换个策略就行
    exporter.set_strategy(HtmlExporter())
    print(exporter.export("Monthly Report", "Sales grew by 20% this month"))
    # <h1>Monthly Report</h1>
    # <p>Sales grew by 20% this month</p>
```
