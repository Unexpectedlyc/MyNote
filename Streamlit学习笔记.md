# Streamlit学习笔记

## Streamlit在ide中调试的方法

```python
# run_app.py
import sys
import streamlit as st
import streamlit.web.cli as stcli


def main():
    # 从命令行参数中获取文件名
    extra_cmdline_args = ["test.py"]

    # 启动Streamlit应用程序
    sys.argv = ["streamlit", "run"] + extra_cmdline_args
    sys.exit(stcli.main())


if __name__ == "__main__":
    main()
```